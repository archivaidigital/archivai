# ArchivAI — Plataforma de Perícia Forense Digital para PDFs

[![Plataforma](https://img.shields.io/badge/plataforma-archivai.cloud-blue)](https://archivai.cloud)
[![Norma](https://img.shields.io/badge/norma-ISO%2FIEC%2027037-green)](https://www.iso.org)
[![Escala](https://img.shields.io/badge/escala-ENFSI-orange)](https://enfsi.eu)
[![Licença](https://img.shields.io/badge/licença-proprietária-red)](#licença)
[![GitHub](https://img.shields.io/badge/GitHub-arquivaidigital%2Farchivai-black?logo=github)](https://github.com/archivaidigital/archivai)

> Repositório público de documentação do ArchivAI. O código-fonte da plataforma é proprietário e não está disponível neste repositório.

---

## O que é o ArchivAI?

O **ArchivAI** é uma plataforma brasileira de perícia forense digital, disponível em [archivai.cloud](https://archivai.cloud), voltada a profissionais que atuam com documentos no formato PDF em contexto probatório ou de conformidade — peritos digitais, assistentes técnicos, advogados e cartórios.

A plataforma automatiza a análise estática forense de documentos PDF, produzindo **pareceres técnicos com cadeia de custódia auditável**, em conformidade com:

- **ISO/IEC 27037** — Identificação, coleta, aquisição e preservação de evidências digitais
- **ISO/IEC 27042** — Análise e interpretação de evidências digitais
- **NIST SP 800-86** — Integração de técnicas forenses em resposta a incidentes
- **Escala verbal ENFSI** — Comunicação padronizada de conclusões periciais
- **Art. 465, §1º do CPC** — Fundamentação técnica de laudos e pareceres periciais

---

## O que o ArchivAI faz?

Para cada documento PDF submetido, o pipeline forense executa em sequência:

| Etapa | Módulo | O que verifica |
|---|---|---|
| 1 | **Hash SHA-256** | Identidade imutável do documento — âncora da cadeia de custódia |
| 2 | **VirusTotal** | Reputação do hash em +70 motores antivírus (por hash, sem envio do arquivo) |
| 3 | **DetectaJS** | Presença de elementos ativos: `/JavaScript`, `/OpenAction`, `/Launch`, `/EmbeddedFile` |
| 4 | **peepdf-3** | Análise estrutural binária completa — inspeciona objetos e streams descomprimidos |
| 5 | **veraPDF** | Conformidade com ISO 19005 (PDF/A) e ISO 14289 (PDF/UA) |
| 6 | **pyHanko** | Verificação de assinaturas digitais ICP-Brasil (PAdES/CAdES/PKCS#7) |
| 7 | **Classificação forense** | Atribuição de selo e geração do parecer técnico em linguagem ENFSI |

O resultado é um **parecer técnico em PDF** com hash SHA-256, carimbo de tempo, selo forense e cadeia de custódia registrada no Supabase com auditoria completa.

---

## Sistema de Selos Forenses

| Selo | Critérios | Equivalência ENFSI |
|---|---|---|
| 🥇 **OURO** | PDF/A válido, assinatura digital ICP-Brasil verificada, sem elementos ativos, sem edição incremental | +4 / +3 |
| 🥈 **PRATA** | Estrutura íntegra, falhas menores de conformidade ISO, ausência de ameaças ativas | +2 / +1 |
| 🥉 **BRONZE** | Múltiplas falhas de conformidade, elementos suspeitos não confirmados | 0 / +1 |
| ⚠️ **IRREGULAR** | Edição incremental confirmada, metadados inconsistentes, indícios de manipulação | -1 / -2 |
| 🚨 **AMEAÇA** | JavaScript ativo, ações automáticas maliciosas, *exploits* PDF confirmados | -3 / -4 |

*Fonte: elaboração do autor, adaptado de ENFSI (2015).*

---

## Integração via API REST

O ArchivAI é exposto como *SaaS* via API REST documentada. Consulte a documentação completa em [`docs/api.md`](docs/api.md).

**Endpoint de verificação pública** (sem autenticação):

```bash
curl "https://archivai.cloud/functions/v1/verify-hash?hash={sha256}"
```

Permite que qualquer sistema verifique a integridade de um documento pelo seu hash SHA-256.

---

## Arquitetura

A plataforma é composta por sete componentes organizados em três camadas:

```
┌─────────────────────────────────────────────┐
│ CAMADA DE INTERFACE                         │
│  Frontend (Lovable) — archivai.cloud        │
└─────────────────────────┬───────────────────┘
                          │
┌─────────────────────────▼───────────────────┐
│ CAMADA DE PROCESSAMENTO                     │
│  Orquestrador: n8n                          │
│  ├── Módulo VirusTotal (API v3)             │
│  ├── Módulo DetectaJS (Node.js)             │
│  ├── Módulo peepdf-3 (Python)               │
│  ├── Módulo veraPDF (Java)                  │
│  └── Módulo pyHanko (Python/Flask)          │
└─────────────────────────┬───────────────────┘
                          │
┌─────────────────────────▼───────────────────┐
│ CAMADA DE PERSISTÊNCIA                      │
│  Supabase (PostgreSQL)                      │
│  ├── Row Level Security (RLS)               │
│  ├── Auth Audit Logs                        │
│  └── PGAudit                                │
└─────────────────────────────────────────────┘
```

Consulte a documentação detalhada dos módulos em [`docs/modules.md`](docs/modules.md).

---

## Documentação

| Arquivo | Conteúdo |
|---|---|
| [`docs/api.md`](docs/api.md) | Referência completa da API REST |
| [`docs/modules.md`](docs/modules.md) | Arquitetura e descrição dos módulos |
| [`docs/custody-schema.md`](docs/custody-schema.md) | Schema da cadeia de custódia |
| [`examples/`](examples/) | Exemplos de pareceres técnicos gerados |

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
