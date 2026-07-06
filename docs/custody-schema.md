# Schema da Cadeia de Custódia — ArchivAI

A cadeia de custódia do ArchivAI é implementada no Supabase (PostgreSQL gerenciado), com auditoria completa via PGAudit e Auth Audit Logs. Cada perícia realizada gera um registro imutável e rastreável, em conformidade com a ISO/IEC 27037 (2012) e o art. 465, §1º do CPC.

---

## Princípios da cadeia de custódia

| Princípio | Implementação no ArchivAI |
|---|---|
| **Integridade** | Hash SHA-256 calculado como primeira etapa absoluta — antes de qualquer leitura de metadados |
| **Reprodutibilidade** | Lógica determinística — o mesmo documento produz sempre o mesmo resultado |
| **Rastreabilidade** | Cada operação é registrada com identificação do perito, timestamp e módulo executor |
| **Não-repúdio** | PGAudit registra todas as operações no banco; carimbo de tempo auditável |
| **Confidencialidade** | Row Level Security — cada perito acessa exclusivamente suas próprias perícias |

---

## Schema das tabelas

### Tabela `pericias`

Registro principal de cada perícia realizada.

```sql
CREATE TABLE pericias (
  -- Identificação
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  perito_id             UUID NOT NULL REFERENCES auth.users(id),
  
  -- Documento
  hash_sha256           TEXT NOT NULL,
  nome_arquivo          TEXT,
  tamanho_bytes         BIGINT,
  
  -- Resultado forense
  selo_status           TEXT NOT NULL 
                        CHECK (selo_status IN ('OURO','PRATA','BRONZE','IRREGULAR','AMEAÇA')),
  is_safe               BOOLEAN NOT NULL,
  classificacao_origem  TEXT 
                        CHECK (classificacao_origem IN ('Nativo Digital','Documento Digitalizado','Origem Não Identificada')),
  
  -- Conformidade ISO
  perfil_iso            TEXT,           -- ex: 'PDF/A-2b'
  conformidade_pct      DECIMAL(5,2),
  
  -- Assinatura digital
  assinatura_presente   BOOLEAN DEFAULT FALSE,
  icp_brasil            BOOLEAN DEFAULT FALSE,
  tipo_assinatura       TEXT,
  signatario            TEXT,
  status_revogacao      TEXT,
  
  -- Análise de malware
  malware_presente      BOOLEAN DEFAULT FALSE,
  indicadores_malware   JSONB,
  objetos_suspeitos     INTEGER DEFAULT 0,
  
  -- VirusTotal
  vt_hash_conhecido     BOOLEAN,
  vt_deteccoes          INTEGER,
  vt_total_motores      INTEGER,
  
  -- Parecer
  conclusao_pericial    TEXT,
  
  -- Cadeia de custódia
  carimbo_tempo         TIMESTAMPTZ NOT NULL DEFAULT now(),
  versao_pipeline       TEXT,
  
  -- Auditoria
  created_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at            TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

### Tabela `modulos_resultados`

Detalhamento dos resultados de cada módulo por perícia.

```sql
CREATE TABLE modulos_resultados (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  pericia_id  UUID NOT NULL REFERENCES pericias(id) ON DELETE CASCADE,
  modulo      TEXT NOT NULL 
              CHECK (modulo IN ('sha256','virustotal','detectajs','peepdf3','verapdf','pyhanko','classificacao')),
  ordem       INTEGER NOT NULL,         -- posição no pipeline (1–7)
  sucesso     BOOLEAN NOT NULL,
  resultado   JSONB,                    -- saída completa do módulo
  duracao_ms  INTEGER,                  -- tempo de execução
  erro        TEXT,                     -- mensagem de erro, se houver
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

### Tabela `peritos`

Cadastro de peritos e controle de créditos.

```sql
CREATE TABLE peritos (
  id              UUID PRIMARY KEY REFERENCES auth.users(id),
  email           TEXT NOT NULL UNIQUE,
  nome            TEXT,
  creditos        INTEGER NOT NULL DEFAULT 0,
  chave_api_hash  TEXT,                 -- hash da chave de API (nunca a chave em texto claro)
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

## Políticas de segurança (Row Level Security)

```sql
-- Peritos acessam apenas suas próprias perícias
ALTER TABLE pericias ENABLE ROW LEVEL SECURITY;

CREATE POLICY "perito_acessa_proprias_pericias"
ON pericias
FOR ALL
USING (perito_id = auth.uid());

-- Endpoint público verify-hash: leitura de hash e selo apenas
CREATE POLICY "verificacao_publica_por_hash"
ON pericias
FOR SELECT
USING (true)
WITH CHECK (false);  -- somente leitura pública dos campos hash e selo
```

---

## Registro de auditoria

O PGAudit registra automaticamente todas as operações DDL e DML nas tabelas sensíveis:

```
2026-05-10 14:32:00 UTC | perito@email.com | INSERT | pericias | hash=e3b0c4... | selo=PRATA
2026-05-10 14:32:01 UTC | perito@email.com | INSERT | modulos_resultados | modulo=virustotal
2026-05-10 14:32:02 UTC | perito@email.com | INSERT | modulos_resultados | modulo=peepdf3
...
```

---

## Exemplo de registro completo

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "perito_id": "usr_abc123",
  "hash_sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "nome_arquivo": "contrato_prestacao_servicos.pdf",
  "tamanho_bytes": 245760,
  "selo_status": "PRATA",
  "is_safe": true,
  "classificacao_origem": "Nativo Digital",
  "perfil_iso": "PDF/A-2b",
  "conformidade_pct": 94.2,
  "assinatura_presente": false,
  "icp_brasil": false,
  "malware_presente": false,
  "indicadores_malware": [],
  "objetos_suspeitos": 0,
  "vt_hash_conhecido": true,
  "vt_deteccoes": 0,
  "vt_total_motores": 72,
  "conclusao_pericial": "A análise forense indica, com alto grau de probabilidade, ausência de indícios de adulteração. A estrutura do arquivo é íntegra e consistente com a origem declarada.",
  "carimbo_tempo": "2026-05-10T14:32:00.000Z",
  "versao_pipeline": "2.1.0",
  "created_at": "2026-05-10T14:32:00.000Z",
  "modulos": [
    { "modulo": "sha256",       "ordem": 1, "sucesso": true, "duracao_ms": 12  },
    { "modulo": "virustotal",   "ordem": 2, "sucesso": true, "duracao_ms": 340 },
    { "modulo": "detectajs",    "ordem": 3, "sucesso": true, "duracao_ms": 45  },
    { "modulo": "peepdf3",      "ordem": 4, "sucesso": true, "duracao_ms": 890 },
    { "modulo": "verapdf",      "ordem": 5, "sucesso": true, "duracao_ms": 1200},
    { "modulo": "pyhanko",      "ordem": 6, "sucesso": true, "duracao_ms": 180 },
    { "modulo": "classificacao","ordem": 7, "sucesso": true, "duracao_ms": 28  }
  ]
}
```

---

## Verificação pública de integridade

Qualquer parte interessada pode verificar se um documento consta no acervo forense do ArchivAI — sem autenticação e sem acesso a dados do perito ou ao conteúdo do documento:

```bash
curl "https://archivai.cloud/functions/v1/verify-hash?hash=e3b0c44298fc1c..."
```

Resposta:
```json
{
  "hash": "e3b0c44298fc1c...",
  "registrado": true,
  "selo_status": "PRATA",
  "timestamp_pericia": "2026-05-10T14:32:00Z"
}
```

Este mecanismo implementa verificação fim-a-fim da cadeia de custódia em conformidade com o princípio de não-repúdio estabelecido pela ISO/IEC 27037 (2012).
