# ArchivAI — Plataforma de Perícia Forense Digital para PDFs

[![Plataforma](https://img.shields.io/badge/plataforma-archivai.cloud-blue)](https://archivai.cloud)
[![Norma](https://img.shields.io/badge/norma-ISO%2FIEC%2027037-green)](https://www.iso.org)
[![Linguagem](https://img.shields.io/badge/linguagem-ENFSI-orange)](https://enfsi.eu)
[![Licença](https://img.shields.io/badge/licença-proprietária-red)](#licença)
[![GitHub](https://img.shields.io/badge/GitHub-archivaidigital%2Farchivai-black?logo=github)](https://github.com/archivaidigital/archivai)

> Repositório público de documentação do ArchivAI. O código-fonte da plataforma é proprietário e não está disponível neste repositório.

---

## O que é o ArchivAI?

O **ArchivAI** é uma plataforma brasileira de perícia forense digital, disponível em [archivai.cloud](https://archivai.cloud), voltada a profissionais que atuam com documentos no formato PDF em contexto probatório ou de conformidade — peritos digitais, assistentes técnicos, advogados e cartórios.

A plataforma automatiza a análise estática forense de documentos PDF e emite um **Parecer Técnico** com registro de integridade por hash SHA-256 e preservação do arquivo original. O desenho do processo toma como referência:

- **ISO/IEC 27037** — Identificação, coleta, aquisição e preservação de evidências digitais
- **ISO/IEC 27042** — Análise e interpretação de evidências digitais
- **NIST SP 800-86** — Integração de técnicas forenses em resposta a incidentes
- **ENFSI** — Orientação de linguagem para conclusões periciais

---

## O que o ArchivAI faz?

Para cada documento PDF submetido, o pipeline forense executa, nesta ordem:

| Etapa | Módulo | O que verifica |
|---|---|---|
| 1 | **Hash SHA-256** | Identidade do arquivo recebido — âncora do registro de integridade |
| 2 | **Reputação por hash (VirusTotal)** | Se o hash consta como malicioso em base de ameaças conhecidas (só o hash é consultado; o arquivo não é enviado). Hash malicioso encerra a análise com selo AMEAÇA |
| 3 | **Triagem** | Rejeita PDFs criptografados (com senha de abertura ou restrições de uso) e devolve o resultado anterior quando o mesmo arquivo já foi periciado pelo mesmo usuário |
| 4 | **Conformidade ISO (veraPDF)** | ISO 32000 (estrutura do PDF), ISO 19005 (PDF/A) e ISO 14289 (PDF/UA) |
| 5 | **Metadados e origem** | Extração de metadados e texto; classificação de origem (Nativo Digital, Documento Digitalizado ou Origem Não Identificada) |
| 6 | **Análise estática de malware (peepdf-3)** | Inspeção estrutural de objetos e streams em busca de JavaScript, ações automáticas, anexos e outros indicadores de código malicioso |
| 7 | **Assinaturas digitais (pyHanko)** | Integridade criptográfica e modificação posterior à assinatura — só quando o documento é assinado |
| 8 | **Parecer Técnico** | Atribuição do selo, conclusão pericial e gravação do registro de custódia |

O resultado é um **Parecer Técnico em PDF** com o hash SHA-256 do documento periciado, o selo de integridade, as não conformidades encontradas e um QR Code para verificação pública do documento pelo hash. A análise é estática: o PDF nunca é executado.

---

## Sistema de Selos

O selo é calculado por regras determinísticas, avaliadas nesta ordem:

| Selo | Critério |
|---|---|
| 🚨 **AMEAÇA** | Hash identificado como malicioso, indício crítico de código malicioso na análise estática, ou assinatura digital cuja integridade não confere ou que foi modificada após a assinatura |
| ⚠️ **IRREGULAR** | Não conformidade com a ISO 32000 (estrutura do PDF) |
| 🥇 **OURO** | Conformidade com ISO 32000, ISO 19005 (PDF/A) e ISO 14289 (PDF/UA) |
| 🥈 **PRATA** | Conformidade com ISO 32000 e com ISO 19005 ou ISO 14289 |
| 🥉 **BRONZE** | Conformidade apenas com ISO 32000 |

Se a análise de código malicioso não puder ser concluída (por exemplo, tempo limite excedido), o selo fica limitado a BRONZE e o Parecer informa que essa análise não foi concluída.

Elementos ativos sem indício de exploração, como JavaScript de abertura automática sem código malicioso conhecido, ou arquivos anexados e multimídia sem JavaScript, aparecem no Parecer como alerta informativo e não alteram o selo.

OURO, PRATA e BRONZE expressam o grau de conformidade normativa de um documento sem indícios de comprometimento; não medem autenticidade. A ausência de conformidade com PDF/A ou PDF/UA reflete escolhas de produção do arquivo e não indica adulteração.

A conclusão do Parecer é descritiva: quando nada compromete o documento, declara que não foram identificados indícios de conteúdo malicioso nem não conformidades com a estrutura PDF; quando há achados, descreve-os sem atribuir causa ou autoria. Em todos os casos, o resultado descreve o arquivo recebido e não atesta sua autenticidade, autoria ou validade jurídica.

---

## Verificação pública

Qualquer pessoa pode verificar, sem login, se um documento consta no acervo do ArchivAI pelo seu hash SHA-256, na página [archivai.cloud/verify](https://archivai.cloud/verify) ou pelo QR Code impresso no Parecer.

A API REST para integração externa está **em manutenção**. Consulte [`docs/api.md`](docs/api.md).

---

## Arquitetura

```
┌─────────────────────────────────────────────┐
│ CAMADA DE INTERFACE                         │
│  Aplicação web — archivai.cloud             │
│  Funções de backend (sessão autenticada)    │
└─────────────────────────┬───────────────────┘
                          │
┌─────────────────────────▼───────────────────┐
│ CAMADA DE PROCESSAMENTO                     │
│  Orquestrador: n8n                          │
│  ├── Reputação por hash (VirusTotal API v3) │
│  ├── Conformidade ISO (veraPDF)             │
│  ├── Malware estático (peepdf-3)            │
│  └── Assinaturas digitais (pyHanko)         │
└─────────────────────────┬───────────────────┘
                          │
┌─────────────────────────▼───────────────────┐
│ CAMADA DE PERSISTÊNCIA                      │
│  Supabase (PostgreSQL + armazenamento)      │
│  ├── Registro de custódia por perícia       │
│  └── Arquivo original preservado            │
└─────────────────────────────────────────────┘
```

Os módulos peepdf-3 e pyHanko rodam em serviços internos isolados, com usuário sem privilégios, tempo máximo de execução e limite de memória. Detalhes em [`docs/modules.md`](docs/modules.md).

---

## Documentação

| Arquivo | Conteúdo |
|---|---|
| [`docs/modules.md`](docs/modules.md) | Pipeline e descrição de cada módulo |
| [`docs/custody-schema.md`](docs/custody-schema.md) | O que o registro de custódia guarda |
| [`docs/api.md`](docs/api.md) | Situação da API REST e verificação pública |
| [`docs/versoes/`](docs/versoes/) | Notas de versão do motor de perícia |
| [`examples/`](examples/) | Exemplos de perícias em documentos públicos |

---

## Limitações conhecidas

- A classificação de origem depende de metadados declarados e do texto extraído; metadados adulterados podem levar a uma classificação incorreta.
- Hash ausente da base de ameaças não significa arquivo seguro; por isso a análise estrutural local é sempre executada.
- A revogação de certificados e a conformidade com a ICP-Brasil não são verificadas; a validade jurídica de assinaturas deve ser confirmada no validador oficial do ITI ([validar.iti.gov.br](https://validar.iti.gov.br)).
- Não há detecção de edição incremental em documentos sem assinatura digital.
- Arquivos anexados ao PDF são identificados, mas não são periciados individualmente; em PDFs do tipo Portfólio, apenas o arquivo contêiner é analisado.
- Não há carimbo de tempo (RFC 3161) no Parecer; o registro de integridade é feito por hash SHA-256.

---

## Base Científica e Normativa

Este projeto é objeto de estudo no artigo científico:

> ALVES, Marcelo Miguel. **ArchivAI: Plataforma de Perícia Forense Digital para Análise Automatizada de Documentos PDF com Cadeia de Custódia**. Trabalho de Conclusão de Curso (Especialização em Computação Forense e Segurança da Informação) — Instituto de Pós-Graduação e Graduação IPOG. Porto Alegre, 2026.

---

## Licença

O código-fonte do ArchivAI é **proprietário**. Este repositório contém exclusivamente documentação pública. Todos os direitos reservados © 2024–2026 Marcelo Miguel Alves.

[ArchivAI | Perícia e Integridade Digital](https://www.archivai.cloud/#como-funciona)  
Para contato: marmalves@gmail.com  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-marmalves-blue?logo=linkedin)](https://www.linkedin.com/in/marmalves/)  
[![YouTube](https://img.shields.io/badge/YouTube-ArchivAI-red?logo=youtube)](https://www.youtube.com/channel/UCYWJ9jDHDcFLXBspQGh0xAw)
