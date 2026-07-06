# API REST — ArchivAI

> **Base URL:** `https://archivai.cloud/functions/v1`

## Autenticação

Todos os endpoints (exceto `verify-hash`) exigem autenticação via chave de API individual, transmitida no header:

```
Authorization: Bearer hc_live_...
```

**Boas práticas de segurança:**
- Armazene a chave exclusivamente em variáveis de ambiente
- Nunca inclua a chave em repositórios públicos ou código-fonte *frontend*
- Rotacione a chave periodicamente via endpoint `manage-api-key`

---

## Endpoints

### 1. Analisar documento PDF

**`POST /process-document`**

Submete um PDF ao pipeline forense completo e retorna o parecer técnico estruturado.

**Request:**
```
Content-Type: multipart/form-data
Authorization: Bearer hc_live_...
```

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `file` | `File` | ✅ | Arquivo PDF (máximo 20 MB) |

**Exemplo (curl):**
```bash
curl -X POST \
  "https://archivai.cloud/functions/v1/process-document" \
  -H "Authorization: Bearer hc_live_..." \
  -F "file=@documento.pdf"
```

**Exemplo (Python):**
```python
import requests

with open("documento.pdf", "rb") as f:
    response = requests.post(
        "https://archivai.cloud/functions/v1/process-document",
        headers={"Authorization": "Bearer hc_live_..."},
        files={"file": f}
    )

result = response.json()
print(result["selo_status"])       # OURO | PRATA | BRONZE | IRREGULAR | AMEAÇA
print(result["hash_sha256_entrada"])
print(result["conclusao_pericial"])
```

**Response (200 OK):**
```json
{
  "hash_sha256_entrada": "e3b0c44298fc1c149afb...",
  "classificacao_arq": {
    "normas": {
      "iso_32000": true,
      "iso_19005": "PDF/A-2b",
      "iso_14289": false
    },
    "origem": "Nativo Digital",
    "conformidade_percentual": 94.2
  },
  "selo_status": "PRATA",
  "is_safe": true,
  "assinatura_digital": {
    "presente": true,
    "tipo": "PAdES",
    "signatario": "NOME DO SIGNATÁRIO",
    "cpf_cnpj": "***.***.***-**",
    "icp_brasil": true,
    "status_revogacao": "válido",
    "tipo_assinatura": "Aprovação"
  },
  "elementos_ativos": {
    "detectados": false,
    "lista": []
  },
  "malware_indicators": {
    "presente": false,
    "indicadores": [],
    "objetos_suspeitos": 0
  },
  "virustotal": {
    "hash_conhecido": true,
    "deteccoes": 0,
    "total_motores": 72
  },
  "conclusao_pericial": "A análise forense do documento indica, com alto grau de probabilidade, ausência de indícios de adulteração ou comprometimento. A estrutura do arquivo é íntegra e consistente com a origem declarada.",
  "cadeia_custodia_id": "uuid-da-pericia",
  "carimbo_tempo": "2026-05-10T14:32:00Z",
  "creditos_restantes": 47
}
```

**Response (curto-circuito AMEAÇA):**
```json
{
  "hash_sha256_entrada": "a1b2c3d4...",
  "selo_status": "AMEAÇA",
  "is_safe": false,
  "motivo_curto_circuito": "VirusTotal: 3 detecções confirmadas",
  "conclusao_pericial": "O documento apresenta indicadores positivos de comprometimento por agente malicioso. Análise adicional em ambiente controlado é recomendada.",
  "cadeia_custodia_id": "uuid-da-pericia",
  "carimbo_tempo": "2026-05-10T14:32:01Z",
  "creditos_restantes": 46
}
```

**Códigos de erro:**

| Código | Descrição |
|---|---|
| `400` | Arquivo ausente ou formato inválido (não é PDF) |
| `401` | Chave de API inválida ou ausente |
| `402` | Créditos insuficientes |
| `413` | Arquivo excede 20 MB |
| `500` | Erro interno — campo `error_node` indica o módulo com falha |

---

### 2. Listar perícias realizadas

**`POST /list-documents`**

Retorna o histórico completo de perícias do usuário autenticado.

**Request:**
```
Authorization: Bearer hc_live_...
Content-Type: application/json
```

**Exemplo:**
```bash
curl -X POST \
  "https://archivai.cloud/functions/v1/list-documents" \
  -H "Authorization: Bearer hc_live_..."
```

**Response (200 OK):**
```json
{
  "pericias": [
    {
      "id": "uuid-da-pericia",
      "hash_sha256": "e3b0c44298fc1c149afb...",
      "nome_arquivo": "contrato.pdf",
      "selo_status": "PRATA",
      "timestamp": "2026-05-10T14:32:00Z"
    }
  ],
  "total": 1
}
```

---

### 3. Obter parecer técnico em PDF

**`GET /get-document-parecer?id={uuid}`**

Gera e entrega o parecer técnico completo em formato PDF binário.

**Exemplo:**
```bash
curl -X GET \
  "https://archivai.cloud/functions/v1/get-document-parecer?id=uuid-da-pericia" \
  -H "Authorization: Bearer hc_live_..." \
  --output parecer.pdf
```

**Response:** arquivo PDF binário com o parecer técnico completo, incluindo hash SHA-256, resultados de cada módulo, selo forense, cadeia de custódia e carimbo de tempo.

---

### 4. Gerenciar chave de API

**`GET /manage-api-key`**

Verifica e rotaciona a chave de API do usuário autenticado.

**Exemplo:**
```bash
curl -X GET \
  "https://archivai.cloud/functions/v1/manage-api-key" \
  -H "Authorization: Bearer hc_live_..."
```

**Response (200 OK):**
```json
{
  "chave_atual": "hc_live_...",
  "criada_em": "2026-01-15T10:00:00Z",
  "ultimo_uso": "2026-05-10T14:32:00Z"
}
```

---

### 5. Verificar integridade por hash (público)

**`GET /verify-hash?hash={sha256}`**

Endpoint público, sem autenticação. Verifica se um hash SHA-256 está registrado no acervo do ArchivAI e retorna o resultado da perícia associada.

**Exemplo:**
```bash
curl "https://archivai.cloud/functions/v1/verify-hash?hash=e3b0c44298fc1c149afb..."
```

**Response — hash encontrado (200 OK):**
```json
{
  "hash": "e3b0c44298fc1c149afb...",
  "registrado": true,
  "selo_status": "PRATA",
  "timestamp_pericia": "2026-05-10T14:32:00Z",
  "mensagem": "Documento encontrado no acervo forense do ArchivAI. Integridade verificada."
}
```

**Response — hash não encontrado (200 OK):**
```json
{
  "hash": "a1b2c3...",
  "registrado": false,
  "mensagem": "Hash não encontrado no acervo. O documento pode ser inédito ou não ter sido submetido ao ArchivAI."
}
```

---

## Fluxo de integração recomendado

```
Sistema emissor                    ArchivAI
      │                                │
      │── POST /process-document ──────►│
      │                                │ 1. Calcula SHA-256
      │                                │ 2. Consulta VirusTotal
      │                                │ 3. DetectaJS
      │                                │ 4. peepdf-3
      │                                │ 5. veraPDF
      │                                │ 6. pyHanko
      │                                │ 7. Classifica + gera parecer
      │◄── JSON com selo e hash ───────│
      │                                │
      │  [armazena hash junto          │
      │   ao documento]                │
      │                                │
Sistema receptor                   ArchivAI
      │                                │
      │── GET /verify-hash?hash={h} ──►│
      │◄── {registrado: true, selo} ───│
      │                                │
      │  [confirma integridade]        │
```

---

## Limitações documentadas

| Limitação | Descrição |
|---|---|
| Documentos inéditos no VirusTotal | Retornam ausência de registro mesmo que contenham malware — complementado pelo peepdf-3 |
| *Soft-fail* CRL/OCSP | Quando fontes de revogação estão inacessíveis, status retorna como indeterminado |
| Metadados falsificados | A classificação de origem depende de metadados declarados — documentos com metadados adulterados podem contornar a classificação |
| Tamanho máximo | 20 MB por requisição |
| Análise exclusivamente estática | O arquivo nunca é executado — compatível com o princípio de integridade forense |
