---
name: resumir-capitulo
description: Lê e resume um capítulo de um livro em PDF. Use quando o usuário pedir para ler, resumir ou sintetizar um capítulo de um PDF/livro (ex.: "resuma o capítulo 3", "resumir capítulo do PDF", "faça um resumo do cap 2 do livro"). Localiza as páginas do capítulo pelo sumário do próprio PDF, lê o conteúdo e salva o resumo no vault de notas.
---

# Resumir capítulo de PDF

Lê um capítulo de um livro em PDF e gera um resumo genérico, salvando-o no vault
Obsidian do projeto.

## Ponto técnico importante

A ferramenta **Read lê PDFs nativamente** através do parâmetro `pages`
(ex.: `pages: "25-44"`). **Máximo de 20 páginas por chamada** — capítulos maiores
exigem múltiplas chamadas `Read` encadeadas. Não use bibliotecas externas
(pdfplumber, PyPDF, etc.) nem escreva scripts; basta o `Read`.

## Passo a passo

### 1. Identificar o PDF
- Por padrão, use `kubernetes_upandrunning3e.pdf` na raiz do projeto.
- Se houver mais de um `*.pdf` na raiz (use Glob `*.pdf`), liste-os e pergunte
  qual o usuário quer, a menos que ele já tenha especificado.

### 2. Mapear capítulo → páginas (ler o sumário)
- Leia as primeiras páginas do PDF para encontrar o sumário / Table of Contents
  (ex.: `Read` com `pages: "1-15"`; se o sumário não aparecer, tente `pages: "16-30"`).
- Extraia o intervalo de páginas do capítulo pedido (página inicial do capítulo até
  o início do próximo capítulo, menos 1).
- **Cuidado com o offset**: a numeração impressa no livro normalmente é diferente do
  número de página físico do PDF (capa, prefácio, etc. deslocam a contagem). Se o
  conteúdo lido não corresponder ao capítulo esperado, ajuste o intervalo e releia.

### 3. Ler o conteúdo do capítulo
- Leia o intervalo identificado em lotes de **no máximo 20 páginas** por chamada
  `Read` (ex.: `pages: "25-44"`, depois `pages: "45-58"`).
- Encadeie os lotes até cobrir o capítulo inteiro antes de resumir.

### 4. Gerar o resumo (formato genérico e flexível)
Escreva o resumo **em português**, de forma livre — sem o formato rígido de SÍNTESE
do CLAUDE.md. Inclua, conforme o conteúdo:
- **Visão geral**: o que o capítulo aborda e por quê.
- **Conceitos principais**: os tópicos-chave explicados de forma concisa.
- **Pontos práticos**: exemplos, comandos ou YAML relevantes, quando houver.
Adapte profundidade e seções ao capítulo; não force seções vazias.

### 5. Salvar no vault e exibir
- Anexe o resumo ao arquivo acumulador `up-and-running/notes/Resumos PDF.md`
  (crie-o se não existir, com um título `# Resumos PDF` no topo).
- Use um cabeçalho por entrada: `## Capítulo N - <título do capítulo>`.
- Se já houver um resumo daquele capítulo no arquivo, pergunte antes de sobrescrever
  ou anexar uma nova versão.
- Exiba o resumo também no chat.

## Observações
- Mantenha o resumo fiel ao texto; não invente conteúdo que não está no capítulo.
- Se o usuário pedir um nível de detalhe específico (curto/longo), ajuste a extensão.
