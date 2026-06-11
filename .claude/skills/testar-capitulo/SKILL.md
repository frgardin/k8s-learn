---
name: testar-capitulo
description: Aplica um teste/avaliação sobre um capítulo, seção ou tópico de um livro técnico em PDF. Use quando o usuário pedir para ser testado/avaliado/arguido sobre o conteúdo de um PDF (ex.: "me teste sobre o capítulo 3", "quiz da seção de Pods", "teste rápido sobre Services", "me avalie sobre o capítulo 2"). Localiza as páginas pelo sumário, lê o conteúdo, gera perguntas com rigor acadêmico, corrige as respostas e salva a ficha de progresso.
---

# Testar capítulo de PDF (avaliador acadêmico)

Aplica um teste sobre um capítulo, seção ou tópico de um livro técnico em PDF. O agente
atua como um **examinador acadêmico exigente**: não apenas marca certo/errado, mas
corrige imprecisões, avalia a qualidade da redação/argumentação e induz o estudante a
escrever cada vez melhor. Ao final, registra a ficha de progresso no vault.

## Ponto técnico importante

A ferramenta **Read lê PDFs nativamente** através do parâmetro `pages`
(ex.: `pages: "25-44"`). **Máximo de 20 páginas por chamada** — capítulos maiores
exigem múltiplas chamadas `Read` encadeadas. Não use bibliotecas externas
(pdfplumber, PyPDF, etc.) nem escreva scripts; basta o `Read`.

## Passo a passo

### 1. Identificar o PDF
- Por padrão, use `kubernetes_upandrunning3e.pdf` na raiz do projeto.
- Se houver mais de um `*.pdf` na raiz (use Glob `*.pdf`), liste-os e pergunte qual o
  usuário quer, a menos que ele já tenha especificado.

### 2. Mapear escopo → páginas (ler o sumário)
Aceite três granularidades de escopo:
- **Capítulo** inteiro (ex.: "capítulo 3").
- **Seção/subseção** (ex.: "a seção de Services do cap. 7").
- **Tópico/assunto** solto (ex.: "ReplicaSets", "health checks").

Para mapear:
- Leia as primeiras páginas do PDF para encontrar o sumário / Table of Contents
  (ex.: `Read` com `pages: "1-15"`; se não aparecer, tente `pages: "16-30"`).
- Extraia o intervalo de páginas correspondente ao escopo. Para um tópico solto,
  localize no sumário a(s) seção(ões) que o cobrem.
- **Cuidado com o offset**: a numeração impressa no livro normalmente difere do número
  físico da página no PDF (capa, prefácio, etc. deslocam a contagem). Se o conteúdo
  lido não corresponder ao esperado, ajuste o intervalo e releia.

### 3. Ler o conteúdo
- Leia o intervalo identificado em lotes de **no máximo 20 páginas** por chamada `Read`.
- Encadeie os lotes até cobrir todo o escopo **antes** de gerar o teste. Você precisa
  dominar o material para avaliar com rigor — nunca invente perguntas ou gabaritos a
  partir de conhecimento externo ao texto lido.

### 4. Definir o modo de interação (perguntar ao aluno)
Antes de começar o teste, pergunte como ele quer responder:
- **Uma pergunta por vez**: apresenta 1 pergunta, aguarda a resposta, corrige com
  feedback e só então segue para a próxima.
- **Todas em lote**: lista todas as perguntas de uma vez; o aluno responde tudo junto e
  recebe a correção no final.

### 5. Calibrar o teste pelo escopo
- **Capítulo inteiro** → 3 níveis completos (modelo do `CLAUDE.md`):
  - **Nível 1 — Conceitual** (5 perguntas): definições, "o que é X".
  - **Nível 2 — Aplicação** (3–4 perguntas): cenários, "como você faria Y".
  - **Nível 3 — Design** (2 perguntas): tradeoffs, decisões e riscos em produção.
- **Seção/tópico** → versão enxuta (~5 perguntas) misturando conceitual + aplicação.
- Sempre que couber, vincule as perguntas a **arquitetura de produção, operação e
  observabilidade**, e conecte com **DDIA (Kleppmann)** ou o **Udemy do Mumshad**
  (alinhado ao `CLAUDE.md`).

### 6. Postura de avaliação — RIGOR ACADÊMICO (núcleo desta skill)
Avalie como um **examinador exigente**, não como um facilitador. Padrão de domínio: 80%+.
- **Gabarito comentado**: explique *por que* a resposta está certa ou errada — nunca
  apenas o veredito.
- **Avalie a resposta como texto acadêmico**: cobre precisão terminológica, clareza,
  estrutura do argumento e uso correto dos conceitos do livro. Aponte imprecisões,
  ambiguidades e jargão mal empregado **mesmo quando a ideia central está correta**.
- **Induza melhor redação**: após corrigir, forneça uma versão modelo (reescrita) ou
  apontamentos concretos de como o aluno poderia ter redigido a resposta com mais rigor.
  Cobre esse padrão mais alto nas perguntas seguintes.
- **Nota por questão**: atribua um conceito (ex.: ❌ 0 / ⚠️ parcial / ✅ completo) com
  justificativa breve.
- Se o aluno errar um conceito, indique a seção específica a revisar (estilo `CLAUDE.md`:
  "Este conceito reaparece no capítulo X. Revise [seção]").

### 7. Ficha de progresso + persistência
Ao final, emita a ficha no formato do `CLAUDE.md`:

```
CAPÍTULO/ESCOPO: [N - Título | seção | tópico]
ACERTOS: [X]/[Total] | PERCENTUAL: [Y]%
TÓPICOS DOMINADOS: [lista]
GAPS IDENTIFICADOS: [conceitos frágeis]
QUALIDADE DA REDAÇÃO: [precisão/clareza/estrutura; evolução vs. testes anteriores]
AÇÃO: [revisar seção específica antes de continuar?]
CONEXÃO COM OUTROS LIVROS: [DDIA cap X / Mumshad módulo Y]
STATUS: ✅ Pronto para próximo / ⚠️ Revise antes
```

Persista a ficha no tracker `up-and-running/notes/Progresso.md`:
- Crie o arquivo com o título `# Progresso de Estudo` no topo, se não existir.
- Acrescente **uma entrada datada por teste** (`## YYYY-MM-DD — <escopo>`), sem
  sobrescrever entradas anteriores. Isso permite consultar a evolução via "meu progresso".

## Observações
- **Fidelidade ao texto**: não invente conteúdo nem perguntas fora do escopo lido.
- Ajuste a extensão se o usuário pedir teste curto/longo.
- Mantenha o tom **direto, estruturado e desafiador** — o aluno é sênior; não subestime.
