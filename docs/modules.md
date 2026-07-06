# Módulos do ArchivAI

O pipeline forense do ArchivAI é composto por sete módulos executados em sequência determinística. A ordem é forense: a integridade é verificada primeiro, o curto-circuito ocorre o mais cedo possível, e a análise estrutural aprofundada só é realizada quando não há ameaça confirmada.

```
PDF recebido
     │
     ▼
[1] SHA-256 ──────────────────────────── âncora imutável
     │
     ▼
[2] VirusTotal ── ameaça confirmada? ── SIM ──► AMEAÇA (curto-circuito)
     │ NÃO
     ▼
[3] DetectaJS ─── elementos ativos?
     │
     ▼
[4] peepdf-3 ──── análise binária ────── indica malware? ── SIM ──► AMEAÇA
     │ NÃO
     ▼
[5] veraPDF ───── conformidade ISO
     │
     ▼
[6] pyHanko ───── assinaturas digitais
     │
     ▼
[7] Classificação forense ────────────── gera selo + parecer + cadeia de custódia
```

---

## Módulo 1 — Hash SHA-256

**Função:** Calcular o identificador imutável do documento antes de qualquer outra operação.

**Por que é o primeiro:** Qualquer leitura posterior ao arquivo — mesmo apenas de metadados — poderia em tese alterar timestamps de acesso. O hash calculado como primeira etapa absoluta garante que a impressão digital do documento corresponde ao estado em que foi recebido, em conformidade com a ISO/IEC 27037 (2012).

**Saída:**
```json
{
  "hash_sha256_entrada": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
}
```

**Base normativa:** ISO/IEC 27037:2012 — o cálculo e registro do hash constitui etapa obrigatória na preservação de evidências digitais.

---

## Módulo 2 — Reputação por Hash — VirusTotal

**Função:** Verificar se o hash SHA-256 do documento está indexado como malicioso na base de inteligência coletiva do VirusTotal.

**Característica forense essencial:** A consulta é realizada **exclusivamente por hash** — o arquivo nunca é transmitido ao VirusTotal. Isso garante que o conteúdo do documento (que pode ser sigiloso) não seja exposto a serviços externos.

**Lógica de curto-circuito:** Se qualquer motor antivírus registrar detecção positiva para o hash, o *pipeline* interrompe imediatamente e emite parecer com selo **AMEAÇA**, sem executar os módulos seguintes.

**Limitação documentada:** Documentos novos ou inéditos — que nunca foram submetidos ao VirusTotal por nenhum usuário — retornam ausência de registro mesmo que contenham *malware*. Por isso, o módulo peepdf-3 realiza análise estrutural local independentemente do resultado do VirusTotal.

**Base normativa:** VIRUSTOTAL (2026); ALSHAMRANI (2022).

---

## Módulo 3 — Detecção de Elementos Ativos — DetectaJS

**Função:** Varredura textual do binário PDF em busca de elementos ativos em suas formas literais.

**Elementos detectados:**

| Elemento | Risco |
|---|---|
| `/JavaScript`, `/JS` | Alto — código executável |
| `/OpenAction` | Alto — execução automática ao abrir |
| `/Launch` | Alto — execução de programas externos |
| `/EmbeddedFile` | Médio — arquivo embutido potencialmente malicioso |
| `/URI`, `/SubmitForm` | Médio — comunicação com serviços externos |
| `/ImportData`, `/RichMedia`, `/XFA` | Variável |
| `/AA` | Variável — ações automáticas |

**Limitações conhecidas:**
- Não detecta JavaScript ofuscado por codificação hexadecimal (ex: `/J#61vaScript`)
- Não descomprime *streams* FlateDecode antes da varredura

Ambas as limitações são cobertas pelo módulo peepdf-3, que realiza análise binária completa incluindo descompressão.

---

## Módulo 4 — Análise Estática de Malware — peepdf-3

**Função:** Análise estrutural binária completa do PDF em nível de objetos, incluindo *streams* comprimidos após decodificação FlateDecode.

**O que inspeciona:** O peepdf-3 percorre a árvore de referências cruzadas do PDF e inspeciona cada objeto individualmente — diferentemente do DetectaJS, que opera sobre o texto bruto do binário. Isso permite detectar elementos ativos ofuscados e código embutido em *streams* comprimidos.

**Saída:**
```json
{
  "malware_presente": false,
  "indicadores": [],
  "objetos_suspeitos": 0
}
```

**Relação com VirusTotal:** Os dois módulos são complementares. Enquanto o VirusTotal verifica reputação histórica por hash (coletiva, retrospectiva), o peepdf-3 analisa a estrutura atual do arquivo de forma independente — capturando documentos novos ou modificados que ainda não constam na base do VirusTotal.

**Base científica:** SAIFULLAH (2022); RAY (2023); FISHBEIN (2023); PEEPDF-3 (2024).

---

## Módulo 5 — Validação de Conformidade ISO — veraPDF

**Função:** Validar a conformidade do documento com as normas ISO de preservação e acessibilidade de PDFs.

**Normas verificadas:**

| Norma | Descrição |
|---|---|
| ISO 32000 | Especificação geral do formato PDF |
| ISO 19005 (PDF/A) | Arquivamento de longo prazo |
| ISO 14289 (PDF/UA) | Acessibilidade universal |

**Saída:**
```json
{
  "perfil_detectado": "PDF/A-2b",
  "conformidade_percentual": 94.2,
  "falhas": [
    {
      "clausula": "6.7.3",
      "descricao": "Font not embedded",
      "criticidade": "baixa"
    }
  ],
  "metadados_xmp": {
    "creator": "Microsoft Word",
    "producer": "Adobe PDF Library",
    "creation_date": "2026-01-15T10:00:00Z",
    "modification_date": "2026-01-15T10:00:00Z"
  }
}
```

**Observação:** A conformidade ISO por si só não comprova autenticidade — fornece indicadores técnicos relevantes para a avaliação pericial. Um documento pode ser PDF/A válido e ainda ter sido adulterado.

**Base científica:** OPEN PRESERVATION FOUNDATION (2023); ISO 19005; ISO 14289.

---

## Módulo 6 — Verificação de Assinaturas Digitais — pyHanko

**Função:** Verificação completa de assinaturas digitais conforme os padrões PAdES, CAdES e PKCS#7, com suporte à cadeia de certificação ICP-Brasil.

**Verificações realizadas:**

1. **Integridade criptográfica** — recalcula o hash dos bytes cobertos pelo campo `/ByteRange` e compara com o valor assinado. Detecta qualquer modificação após a assinatura, inclusive alterações mínimas invisíveis ao olho humano.

2. **Modificações pós-assinatura** — identifica alterações incrementais ao PDF realizadas após a assinatura (sobreposição de camadas, substituição de objetos).

3. **Cadeia de certificados** — valida a hierarquia de certificação até a AC raiz ICP-Brasil (ITI), verificando se o certificado foi emitido por Autoridade Certificadora credenciada.

4. **Status de revogação** — consulta listas CRL e serviços OCSP. Quando essas fontes estão inacessíveis, o resultado retorna como indeterminado (*soft-fail*) — limitação documentada do módulo.

5. **Identificação do signatário** — extrai nome, CPF ou CNPJ embutidos no certificado ICP-Brasil.

6. **Tipo de assinatura** — classifica entre Aprovação, Certificação (MDP) e Carimbo de Tempo.

**Saída:**
```json
{
  "assinatura_presente": true,
  "quantidade": 1,
  "assinaturas": [
    {
      "tipo": "PAdES",
      "tipo_assinatura": "Aprovação",
      "signatario": "NOME DO SIGNATÁRIO",
      "cpf_cnpj": "***.***.***-**",
      "icp_brasil": true,
      "validade_certificado": "2027-12-31",
      "status_revogacao": "válido",
      "integridade_pos_assinatura": "íntegra",
      "modificacoes_pos_assinatura": false
    }
  ]
}
```

**Base científica:** VALVEKENS (2021); Lei 14.063/2020; ETSI EN 319 102-1.

---

## Módulo 7 — Classificação Forense e Geração do Parecer

**Função:** Consolidar os resultados de todos os módulos anteriores, atribuir o selo forense e gerar o parecer técnico em linguagem ENFSI.

**Lógica determinística:** O módulo implementa regras baseadas em evidências — não utiliza modelos de linguagem em tempo de execução. O mesmo conjunto de entradas produz sempre o mesmo parecer, garantindo reprodutibilidade e auditabilidade forense.

**Classificação de origem:** Analisa os metadados XMP para classificar o documento em:
- **Nativo Digital** — criado originariamente em meio eletrônico
- **Documento Digitalizado** — obtido por digitalização de documento físico
- **Origem Não Identificada** — metadados insuficientes ou ausentes

**Cadeia de custódia:** Ao final de cada análise, um registro imutável é gravado no Supabase com:
- Hash SHA-256 do documento
- Resultados de cada módulo
- Selo forense atribuído
- Carimbo de tempo
- Identificação do perito (via chave de API)
- Auditoria via PGAudit e Auth Audit Logs

**Base normativa:** ENFSI (2015); ISO/IEC 27037 (2012); art. 465, §1º do CPC.
