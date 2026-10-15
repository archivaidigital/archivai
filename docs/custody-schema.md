# Registro de Custódia — ArchivAI

Cada perícia realizada pelo ArchivAI gera um registro no banco de dados da plataforma (Supabase/PostgreSQL) e preserva o arquivo PDF original no armazenamento da plataforma. Este documento descreve o que o registro guarda, sem detalhes de implementação ou de segurança da infraestrutura.

---

## Princípios

| Princípio | Como o ArchivAI aplica |
|---|---|
| **Identificação** | Hash SHA-256 calculado antes de qualquer outra operação e gravado no registro |
| **Preservação** | Arquivo original guardado no armazenamento da plataforma, vinculado ao registro |
| **Reprodutibilidade** | Selo e Parecer produzidos por regras determinísticas |
| **Rastreabilidade** | Cada registro vincula o documento ao usuário que solicitou a perícia |
| **Verificação** | Qualquer pessoa pode confirmar, pelo hash, se o documento consta no acervo |

---

## O que o registro guarda

### Documento

| Informação | Descrição |
|---|---|
| Nome do arquivo | Nome do arquivo enviado |
| Hash SHA-256 | Identidade do arquivo recebido |
| Tamanho | Tamanho do arquivo em bytes |
| Número de páginas | Total de páginas |
| Referência ao arquivo preservado | Localização do original no armazenamento |
| Metadados extraídos | Dicionário de informações do PDF (Producer, Creator, Author, datas, versão, presença de assinatura e formulário) |
| Título e resumo | Título identificado nos metadados e resumo do texto extraído |

### Resultado da perícia

| Informação | Descrição |
|---|---|
| Selo | OURO, PRATA, BRONZE, IRREGULAR ou AMEAÇA |
| Conformidade ISO | Resultado para ISO 32000, ISO 19005 (PDF/A) e ISO 14289 (PDF/UA) |
| Segurança | Indicação de documento seguro ou de elementos ativos e indicadores de código malicioso |
| Não conformidades | Lista das regras ISO violadas |
| Classificação | Origem do documento, detalhe técnico da classificação e base legal aplicável |
| Alertas | Segurança, reputação por hash, anomalia de datas, assinatura digital e conclusão pericial |
| Estado do documento | Síntese do estado de integridade apresentada no Parecer |
| Justificativa | Texto técnico que fundamenta a conclusão |

### Contexto

| Informação | Descrição |
|---|---|
| Usuário | Identificação do usuário que solicitou a perícia |
| Situação do processamento | Indica a conclusão da análise |

---

## Verificação pública

Qualquer pessoa pode verificar se um documento consta no acervo, sem login, informando o hash SHA-256 na página [archivai.cloud/verify](https://archivai.cloud/verify) ou lendo o QR Code do Parecer. A verificação devolve a existência do registro e o selo, sem expor o conteúdo do documento nem dados do usuário.

---

## Limitações

- Não há carimbo de tempo (RFC 3161) vinculado ao registro; a data registrada é a do próprio sistema.
- O registro identifica o estado do arquivo **no momento do envio**. Não permite afirmar que o documento não foi alterado antes de chegar ao ArchivAI, exceto quando há assinatura digital íntegra.
