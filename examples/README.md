# Exemplos de Pareceres Técnicos

Esta pasta contém exemplos de pareceres técnicos gerados pelo ArchivAI para documentos PDF públicos, ilustrando os diferentes selos forenses e cenários de análise.

## Documentos utilizados

Todos os PDFs utilizados nestes exemplos são documentos públicos, obtidos de fontes oficiais brasileiras, sem dados pessoais sensíveis.

| Arquivo | Origem | Selo esperado | Descrição do cenário |
|---|---|---|---|
| `exemplo_ouro/` | A publicar | 🥇 OURO | PDF/A com assinatura ICP-Brasil válida |
| `exemplo_prata/` | A publicar | 🥈 PRATA | PDF nativo íntegro, sem assinatura |
| `exemplo_bronze/` | A publicar | 🥉 BRONZE | PDF com falhas menores de conformidade ISO |
| `exemplo_irregular/` | A publicar | ⚠️ IRREGULAR | PDF com edição incremental detectada |
| `exemplo_ameaca/` | A publicar | 🚨 AMEAÇA | PDF com JavaScript embutido (amostra educacional) |

> **Nota:** Os exemplos serão publicados após a conclusão das perícias de validação da plataforma, realizadas com documentos públicos selecionados.

## Fontes sugeridas para PDFs públicos

Para replicar análises com documentos públicos:

- **Diário Oficial da União** — [in.gov.br](https://www.in.gov.br) — PDFs nativos digitais com e sem assinatura ICP-Brasil
- **Portal da Transparência** — [portaldatransparencia.gov.br](https://portaldatransparencia.gov.br) — relatórios e portarias em PDF
- **CNJ** — [cnj.jus.br](https://www.cnj.jus.br) — resoluções e atos normativos assinados digitalmente
- **STF** — [stf.jus.br](https://www.stf.jus.br) — acórdãos e decisões em PDF
- **Receita Federal** — [gov.br/receitafederal](https://www.gov.br/receitafederal) — documentos fiscais públicos

## Estrutura de cada exemplo

```
exemplo_prata/
├── documento.pdf          # PDF analisado (público)
├── parecer.pdf            # Parecer técnico gerado pelo ArchivAI
├── resultado.json         # Resposta JSON completa da API
└── README.md              # Contexto e análise do caso
```
