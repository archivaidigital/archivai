# API REST — ArchivAI

> **Situação: ainda não disponível.** A autenticação por chave de API para integrações externas não foi implementada. A geração de chaves está suspensa na plataforma, e as chaves existentes não são aceitas pelos endpoints. Esta página será atualizada quando a API estiver disponível.

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
| PDFs criptografados (com senha de abertura ou só com restrições de uso) | Rejeitados; remova a proteção antes de enviar |
| Envio repetido do mesmo arquivo pelo mesmo usuário | Devolve o resultado anterior, sem consumo de crédito |
| Análise | Exclusivamente estática; o arquivo nunca é executado |

---

## Trabalho futuro

- Implementação da autenticação por chave de API nos endpoints.
- Publicação do endereço base, dos contratos de requisição e resposta e dos códigos de erro.
