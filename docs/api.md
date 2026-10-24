# API REST — ArchivAI

> **Situação: em manutenção.** A autenticação por chave de API para integrações externas está suspensa. A geração de chaves continua disponível na plataforma, mas as chaves ainda não são aceitas pelos endpoints. Esta página será atualizada quando a API for reaberta.

---

## O que está disponível hoje

### Verificação pública por hash

Qualquer pessoa pode verificar, sem login, se um documento consta no acervo do ArchivAI:

- pela página [archivai.cloud/verify](https://archivai.cloud/verify), informando o hash SHA-256 do documento;
- pelo QR Code impresso em cada Parecer Técnico.

O hash deve ter de 16 a 128 caracteres hexadecimais. A verificação informa se o documento está registrado, sem expor o conteúdo do documento, o nome do arquivo nem dados do usuário.

### Perícia de documentos

A submissão de documentos e a emissão do Parecer Técnico estão disponíveis na plataforma [archivai.cloud](https://archivai.cloud), com sessão autenticada.

---

## Limites

| Item | Valor |
|---|---|
| Tamanho máximo por arquivo | 20 MB |
| PDFs protegidos por senha | Rejeitados; remova a proteção antes de enviar |
| Envio repetido do mesmo arquivo pelo mesmo usuário | Devolve o resultado anterior, sem consumo de crédito |
| Análise | Exclusivamente estática; o arquivo nunca é executado |

---

## Trabalho futuro

- Validação das chaves de API nos endpoints e reabertura da integração externa.
- Publicação do endereço base, dos contratos de requisição e resposta e dos códigos de erro.
