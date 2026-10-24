# Módulos do ArchivAI

O pipeline forense do ArchivAI é orquestrado no n8n. O hash é calculado antes de qualquer outra operação, a consulta de reputação permite encerrar cedo a análise de arquivos já conhecidos como maliciosos, e todas as demais camadas são executadas para cada documento aceito. O selo é atribuído ao final, com todos os resultados reunidos.

```
PDF recebido
     │
     ▼
[1] SHA-256 ─────────────────────────── identidade do arquivo
     │
     ▼
[2] Reputação por hash ── malicioso? ── SIM ──► AMEAÇA (análise encerrada)
     │ NÃO
     ▼
[3] Triagem ── protegido por senha? ─── SIM ──► rejeitado
     │       └ já periciado pelo usuário? ─ SIM ──► resultado anterior
     ▼
[4] veraPDF ─────────────────────────── conformidade ISO 32000 / 19005 / 14289
     │
     ▼
[5] Metadados e origem ──────────────── classificação de origem
     │
     └──► [6] peepdf-3 ──► [7] pyHanko  malware estático / assinaturas
                │
                ▼
[8] Parecer Técnico ─────────────────── selo + conclusão + registro de custódia
```

---

## Módulo 1 — Hash SHA-256

**Função:** calcular o identificador do arquivo recebido antes de qualquer leitura de conteúdo ou metadados.

**Uso:** o hash identifica o documento no registro de custódia, na verificação pública e na detecção de envios repetidos, e é impresso no Parecer.

**Base normativa:** ISO/IEC 27037:2012 — cálculo e registro do hash como etapa de preservação da evidência digital.

---

## Módulo 2 — Reputação por hash (VirusTotal)

**Função:** verificar se o hash SHA-256 do documento consta como malicioso na base do VirusTotal.

**Característica forense:** a consulta é feita **exclusivamente pelo hash**. O arquivo nunca é enviado ao serviço, o que preserva o sigilo do conteúdo.

**Encerramento antecipado:** se ao menos um motor de análise classificar o hash como malicioso, a análise é encerrada e o resultado é registrado com selo **AMEAÇA**.

**Limitação:** arquivos que nunca foram submetidos ao VirusTotal não têm registro, mesmo que sejam maliciosos. Por isso a análise estrutural local (módulo 6) é sempre executada quando o hash não é identificado como malicioso.

---

## Módulo 3 — Triagem

**Documento criptografado:** PDF com senha de abertura ou com restrições de uso é rejeitado antes da análise, com orientação para remover a proteção e reenviar.

**Envio repetido:** se o mesmo usuário já periciou um arquivo com o mesmo hash, o resultado registrado é devolvido sem nova análise e sem consumo de crédito.

---

## Módulo 4 — Conformidade ISO (veraPDF)

**Função:** validar o documento contra as normas ISO do formato PDF.

| Norma | O que representa |
|---|---|
| ISO 32000 | Especificação do formato PDF — estrutura do arquivo |
| ISO 19005 (PDF/A) | Preservação de longo prazo |
| ISO 14289 (PDF/UA) | Acessibilidade |

**Uso no selo:** a não conformidade com a ISO 32000 leva ao selo IRREGULAR. As conformidades com ISO 19005 e ISO 14289 definem OURO, PRATA ou BRONZE.

**Não conformidades:** as regras violadas são listadas no Parecer, identificadas pela norma correspondente.

**Observação:** conformidade ISO não comprova autenticidade. Um documento pode ser PDF/A válido e ter sido adulterado antes da perícia.

---

## Módulo 5 — Metadados e classificação de origem

**Função:** extrair os metadados do dicionário de informações do PDF (Producer, Creator, Author, datas de criação e modificação) e o texto do documento.

**Classificação de origem:**

| Origem | Critério |
|---|---|
| **Documento Digitalizado** | Software de digitalização identificado nos metadados, ou camada de texto ausente ou mínima em documento sem sistema de autoria identificado |
| **Nativo Digital** | Sistema de autoria identificado nos metadados, ou texto nativo em quantidade suficiente |
| **Origem Não Identificada** | Metadados e texto insuficientes para classificar |

**Datas:** data de criação posterior à de modificação gera alerta de anomalia; data de modificação posterior à de criação gera alerta informativo. Esses alertas não alteram o selo.

**Base legal:** o Parecer compõe a base legal aplicável a partir da origem e da presença de assinatura digital.

**Limitação:** metadados podem ser falsificados; a classificação descreve o que o arquivo declara e o que sua estrutura indica, não a história real do documento.

---

## Módulo 6 — Análise estática de malware (peepdf-3)

**Função:** inspeção estrutural do PDF em nível de objetos, incluindo streams comprimidos e nomes ofuscados, em busca de JavaScript, ações automáticas, arquivos anexados, conteúdo multimídia e outros indicadores de código malicioso.

**Execução isolada:** roda em serviço interno separado, com usuário sem privilégios, tempo máximo de execução e limite de memória. O arquivo é analisado de forma estática e nunca é executado.

**Classificação dos achados:**

| Achado | Efeito |
|---|---|
| JavaScript com padrões de exploração conhecidos ou com endereços de internet no código; JavaScript combinado com execução de programa (`/Launch`), arquivo anexado, multimídia, envio ou importação de dados; ou `/Launch` mesmo sem JavaScript | Indício crítico: selo **AMEAÇA**, com o indício citado no Parecer |
| Elementos ativos sem indício de exploração (por exemplo, JavaScript de abertura automática) ou arquivo anexado ou multimídia sem JavaScript | Alerta informativo no Parecer, sem alterar o selo |
| Análise não concluída (tempo limite ou falha) | Selo limitado a **BRONZE**; o Parecer informa que a análise não foi concluída e não afirma a ausência de conteúdo malicioso |

**Relação com o módulo 2:** a reputação por hash é retrospectiva e coletiva; a análise estrutural é local e independe de o arquivo já ter sido visto antes.

**Limitação:** links com o esquema `javascript:` em ações de URL não são sinalizados.

---

## Módulo 7 — Assinaturas digitais (pyHanko)

**Função:** quando o documento possui assinatura digital, verificar cada assinatura.

**Verificações:**

1. **Integridade criptográfica** — confere se os bytes cobertos pela assinatura correspondem ao valor assinado.
2. **Modificação posterior à assinatura** — identifica alterações feitas no arquivo depois de assinado.
3. **Signatário e certificado** — extrai a identificação do signatário, a autoridade certificadora e a validade **declaradas** no certificado.

**Uso no selo:** assinatura cuja integridade não confere ou modificada após a assinatura leva ao selo AMEAÇA.

**O que não é verificado:** revogação do certificado e conformidade com a ICP-Brasil. A validade jurídica da assinatura não é verificada pelo ArchivAI; recomenda-se conferi-la no validador do ITI.

**Processamento local:** a validação é feita no próprio ArchivAI; o documento não é enviado a validadores externos. A verificação não substitui o validador oficial do ITI.

---

## Módulo 8 — Parecer Técnico e registro de custódia

**Função:** consolidar os resultados, atribuir o selo, redigir a conclusão pericial e gravar o registro.

**Lógica determinística:** o selo e o texto do Parecer são produzidos por regras fixas, sem modelos de linguagem em tempo de execução. O mesmo arquivo, nas mesmas condições, produz o mesmo resultado.

**Ordem de avaliação do selo:** AMEAÇA → IRREGULAR → OURO → PRATA → BRONZE (critérios no [README](../README.md#sistema-de-selos)). Análise de código malicioso não concluída limita o selo a BRONZE.

**Conclusão pericial:** descritiva e definida pelo selo. Sem achados que comprometam o documento, declara que não foram identificados indícios de conteúdo malicioso nem não conformidades com a estrutura PDF; com achados, descreve-os sem atribuir causa ou autoria. O resultado descreve o arquivo recebido e não atesta autenticidade, autoria ou validade jurídica.

**Versão:** cada registro guarda a versão do motor de perícia que o produziu, exibida no Parecer. Veja [`versoes/`](versoes/).

**Registro:** o arquivo original é preservado no armazenamento da plataforma e o resultado é gravado no registro de custódia. Veja [`custody-schema.md`](custody-schema.md).

**Base normativa:** ISO/IEC 27037 (2012); ISO/IEC 27042 (2015); ENFSI (2015).
