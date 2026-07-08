# Exemplos de Pareceres Técnicos

Perícias forenses realizadas pelo ArchivAI em documentos PDF públicos de domínio público, demonstrando os diferentes selos forenses e cenários de análise. Todos os documentos são de acesso público e foram submetidos ao pipeline completo da plataforma.

---

## Resultados das Perícias

| Documento | Origem | Páginas | Selo | SHA-256 |
|---|---|---|---|---|
| [Matterhorn Protocol 1.1](https://pdfa.org/wp-content/uploads/2021/04/Matterhorn-Protocol-1-1.pdf) | PDF Association | — | 🥈 PRATA | `1ee0d4bc` |
| [Resolução CNJ 469/2022](https://juslaboris.tst.jus.br/bitstream/handle/20.500.12178/205881/2022_res0469_cnj.pdf?sequence=3&isAllowed=y) | CNJ | — | 🥉 BRONZE | `1034e1cd` |
| [CGI.br e o Marco Civil da Internet](https://www.cgi.br/media/docs/publicacoes/4/CGI-e-o-Marco-Civil.pdf) | CGI.br | — | 🥉 BRONZE | `cf652284` |
| [Lei de Acesso à Informação — Lei 12.527/2011](https://www.gov.br/inpi/pt-br/acesso-a-informacao/dados-abertos/arquivos/documentos/diversos/Lei12.5272011DisciplinadoDireitoFundamentaldeAcessoaInformao.pdf) | INPI/Gov.br | — | 🥉 BRONZE | `8b5ac6ae` |
| [ECA 2026 — Estatuto da Criança e do Adolescente](https://cedecarj.org.br/wp-content/uploads/2026/02/ECA2026_Miolo_1_online.pdf) | CEDECA-RJ | — | 🥉 BRONZE | `030e9c77` |
| [e-ARQ Brasil v2 — CONARQ](https://www.gov.br/conarq/pt-br/centrais-de-conteudo/publicacoes/EARQV203MAI2022.pdf) | CONARQ/Gov.br | — | 🥉 BRONZE | `2fc80d4d` |
| [CLT — Consolidação das Leis do Trabalho 4ª ed.](https://sindhosfilvp.com.br/wp-content/uploads/2024/06/consolidacao_leis_trabalho_4ed-atualizada.pdf) | Câmara dos Deputados | — | 🥉 BRONZE | `a092c9ac` |
| [MoReq-Jus 2ª edição — CNJ](https://www.cnj.jus.br/wp-content/uploads/2023/10/moreq-jus-2a-edicao.pdf) | CNJ | — | 🥉 BRONZE | `ba818e67` |
| [Constituição Federal 88 — EC91/2016 (Senado)](https://www2.senado.leg.br/bdsf/bitstream/handle/id/518231/CF88_Livro_EC91_2016.pdf) | Senado Federal | — | 🥉 BRONZE | `90f6a616` |
| [Constituição Federal 88 — 69ª edição (Câmara)](https://www.camara.leg.br) | Câmara dos Deputados | 298 | 🥉 BRONZE | `bb4d1368` |
| [Lei Áurea — Lei 3.353/1888](http://www12.senado.leg.br/institucional/arquivo/documentos-apenas/lei-aurea) | Senado Federal | — | 🥉 BRONZE | `bb2fff39` |

---

## Destaques Analíticos

### 🥈 Matterhorn Protocol 1.1 — A norma de acessibilidade que pratica o que prega
**Protocolo:** `1ee0d4bc` | **Produtor:** Adobe InDesign 16.1 + Adobe PDF Library 15.0

O Matterhorn Protocol é o modelo de conformidade para PDF/UA (ISO 14289) — a norma internacional de acessibilidade para PDFs. O documento é **conforme com PDF/UA e ISO 32000**, o que é coerente com sua especialidade: o documento de acessibilidade é, ele mesmo, acessível. A única não conformidade detectada é com PDF/A (preservação de longo prazo) — falha de identificação de versão (`pdfaid:part`) —, o que é esperado, pois PDF/A e PDF/UA são normas com objetivos distintos e o Matterhorn Protocol não se propõe a ser um documento de arquivo. Existe um equivalente para PDF/A — o **Isartor Test Suite** e o **Bavaria Test Suite**, mantidos pela PDF Association. Selo PRATA — estrutura íntegra com falha menor de preservação.

---

### 🥉 Resolução CNJ 469/2022 — A norma que define digitalização, sem conformidade de preservação
**Protocolo:** `1034e1cd` | **Produtor:** PDF/A Validator and PDF to PDF/A

A própria resolução que estabelece diretrizes de digitalização de documentos do Poder Judiciário **não está em conformidade com PDF/A** — apresenta 4 falhas, incluindo problema de fonte CID e ausência de marcação de acessibilidade. Ferramenta de criação identificada como `UnknownApplication`, com o produtor PDF indicando tentativa de conversão para PDF/A que não foi concluída com sucesso.

---

### 🥉 Lei Áurea (1888) — Documento digitalizado identificado corretamente
**Protocolo:** `bb2fff39` | **Produtor:** LuraTech PDF Compressor Desktop 6.1 + LuraDocument PDF v2.63

Único documento da amostra classificado pelo ArchivAI como **Documento Digitalizado** — origem física convertida para meio digital, com indícios de OCR detectados no conteúdo textual. O resumo gerado reflete o texto bruto do OCR com ruídos característicos de digitalização de manuscritos históricos. Demonstra a capacidade do módulo de classificação de origem de distinguir documentos digitalizados de nativos digitais com base em evidências estruturais, sem depender exclusivamente dos metadados declarados.

---

### 🥉 e-ARQ Brasil v2 — A norma arquivística também não é PDF/A
**Protocolo:** `2fc80d4d` | **Produtor:** Adobe InDesign 17.2 + Adobe PDF Library 16.0.7

O modelo de requisitos para sistemas arquivísticos do CONARQ — referência normativa central para gestão de documentos digitais no Brasil — apresenta 5 não conformidades com PDF/A e PDF/UA. Ilustra que mesmo documentos técnicos de alto nível institucional frequentemente não atendem às normas de preservação digital de longo prazo.

---

### 🥉 CGI.br e o Marco Civil — Documento de 2013 com ferramentas da época
**Protocolo:** `cf652284` | **Produtor:** Adobe InDesign CS6 + Adobe PDF Library 10.0.1 (2013)

Publicado em 2013, antes da vigência das normas ISO de acessibilidade amplamente adotadas. As 5 não conformidades refletem o estado da arte das ferramentas de produção PDF da época, quando requisitos de PDF/A e PDF/UA ainda não eram padrão de mercado.

---

## Observações Técnicas

**Padrão identificado na amostra:** nenhum dos documentos públicos analisados é conforme com PDF/A (ISO 19005), o que é consistente com a literatura técnica — a maioria dos documentos PDF produzidos por órgãos públicos e privados não foi gerada com conformidade de preservação de longo prazo como requisito. Essa lacuna é exatamente o contexto que justifica plataformas de perícia forense como o ArchivAI.

**Sobre os selos:** a ausência de conformidade ISO não implica adulteração ou comprometimento — reflete escolhas técnicas de produção. O selo BRONZE indica documento íntegro com falhas de conformidade normativa, enquanto IRREGULAR e AMEAÇA indicam evidências de comprometimento estrutural.

**Campo `conclusao_pericial` vazio:** identificado como item de backlog técnico — a lógica de geração da conclusão narrativa está em revisão na versão atual da plataforma.

---

> **Verificação de integridade:** qualquer um dos documentos acima pode ser verificado pelo hash SHA-256 via endpoint público:
> ```
> curl "https://archivai.cloud/functions/v1/verify-hash?hash={sha256_completo}"
> ```
