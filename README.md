# BioLyzer

Aplicativo desktop **offline** para apoiar quem está começando em bioinformática. Reúne, em um só lugar, conversões de sequências, um comparador/buscador e atalhos para ferramentas online de análise de biomoléculas.

A bioinformática é uma área de nicho e muitas plataformas são complexas para iniciantes. A ideia do BioLyzer é facilitar esse primeiro contato, inclusive com alunos do ensino médio.

<!-- Para adicionar um print, salve a imagem em uma pasta "imagens" e descomente a linha abaixo:
![Tela inicial do BioLyzer](imagens/tela-inicial.png)
-->

## Índice

- [Como executar o BioLyzer](#como-executar-o-biolyzer)
- [Guia de uso](#guia-de-uso)
  - [Conversor DNA ↔ RNA](#1-conversor-dna--rna)
  - [Tradutor RNA → Proteína](#2-tradutor-rna--proteína)
  - [Comparador de sequências](#3-comparador-de-sequências)
  - [Ferramentas online](#4-ferramentas-online)
- [Como o código funciona (por dentro)](#como-o-código-funciona-por-dentro)
- [Estrutura de arquivos](#estrutura-de-arquivos)
- [Limitações conhecidas](#limitações-conhecidas)
- [Créditos](#créditos)

## Como executar o BioLyzer

1. Baixe/clone o projeto.
2. Entre na pasta `dist/` (ela já vem junto com o repositório).
3. Abra o executável correspondente ao seu sistema:
   - **Windows** → `BioLyzer.exe` (portátil — dá dois cliques e já abre, não precisa instalar).
   - **Linux** → `BioLyzer.AppImage` (dê permissão de execução se precisar: `chmod +x BioLyzer.AppImage`, depois dois cliques ou `./BioLyzer.AppImage`).
  
Pronto — o app abre numa janela própria, offline, sem depender de navegador nem de nada instalado além do sistema operacional.
### Gerando o executável você mesmo (opcional)

A pasta `dist/` já vem pronta no repositório, então normalmente você **não precisa** rodar isso. Mas se você mudou o código e quer gerar uma nova versão do executável:

```bash
npm install
npm run build
```

Isso baixa o Electron pra sua plataforma e recria a pasta `dist/` com o app atualizado (`.exe` no Windows, `.AppImage` no Linux, `.dmg` no macOS).

Na raiz do projeto há arquivos de teste que já podem ser carregados no app pra você experimentar sem digitar nada:

- `exempodna.fasta`: sequências de DNA
- `exemporna.fasta`: sequências de RNA
- `teste.fasta`: duas sequências curtas de RNA, boas pra testar o comparador

## Guia de uso

O app é dividido em abas (menu lateral/superior). Cada uma abre uma "página" (`div.page`) diferente — só uma fica visível por vez.

### 1. Conversor DNA ↔ RNA

**Pra que serve:** trocar T por U (DNA → RNA) ou U por T (RNA → DNA), respeitando a regra biológica de transcrição.

**Como usar:**
1. Abra a aba de conversão.
2. Digite a sequência diretamente na caixa de texto **ou** clique em carregar um arquivo `.fasta`/`.fa` (pode ter mais de uma sequência dentro, no formato `>nome` seguido da sequência).
3. Clique no botão de converter.
4. O resultado aparece já em formato FASTA (`>nome_RNA` ou `>nome_DNA` + a sequência convertida), pronto pra copiar ou baixar em arquivo.

Se você digitar um texto solto (sem `>nome`), o app tenta separar automaticamente várias sequências coladas uma atrás da outra, mesmo sem o cabeçalho `>`.

Qualquer caractere que não seja A, T, G ou C (no caso do DNA) ou A, U, G, C (no caso do RNA) é descartado na limpeza da sequência.

### 2. Tradutor RNA → Proteína

**Pra que serve:** ler uma sequência de RNA em grupos de 3 letras (códons) e converter cada códon no aminoácido correspondente, usando a tabela padrão do código genético.

**Como usar:**
1. Abra a aba de tradução.
2. Cole a sequência de RNA (ou carregue um `.fasta`).
3. Clique em traduzir.
4. O resultado sai como `>nome_Proteina` + a sequência de aminoácidos (uma letra por aminoácido). Onde aparecer um **códon de parada** (`UAA`, `UAG` ou `UGA`), o app marca com `*` em vez de parar a tradução ali — assim você vê onde a proteína realmente termina, mesmo que tenha códons depois na sequência bruta.
5. Um códon incompleto no final (sobrou 1 ou 2 letras) é simplesmente ignorado.

### 3. Comparador de sequências

Essa é a aba mais completa. Ela tem duas finalidades bem diferentes, controladas pela caixa **"Modo Buscar Trecho"**:

#### 3.1. Modo comparação (caixa desmarcada)

Compara sequências entre si e calcula o quanto elas se parecem.

**Como usar:**
1. Escolha o **tipo de sequência**: botão **DNA/RNA** ou **Proteína** (isso só muda as cores e a legenda mostradas na tabela — não bloqueia mais nenhuma função, mesmo que você esqueça de trocar).
2. Adicione quantas sequências quiser (botão de adicionar campo, ou carregue um `.fasta` com várias entradas de uma vez — cada `>nome` vira uma sequência separada).
3. Clique em **Comparar**:
   - A primeira sequência é usada como referência ("base") e comparada contra cada uma das outras.
   - Aparece uma tabela colorida com cada posição das sequências lado a lado, e uma linha **"100% conservadas"** embaixo marcando com `*` as colunas em que todas as sequências têm exatamente o mesmo caractere.
   - Abaixo da tabela, um resumo em % de similaridade de cada sequência contra a base, com cores (verde ≥ 90%, amarelo ≥ 50%, vermelho abaixo disso). Dá pra ordenar por maior ou menor similaridade.
4. Clique em **Matriz de Similaridade** em vez de Comparar se quiser o "todos contra todos": uma tabela onde cada célula é a % de similaridade entre duas sequências específicas, não só contra a primeira.

**Como a similaridade é calculada:** o app **não** faz um alinhamento com inserção de espaços no meio (não é um Needleman-Wunsch/Smith-Waterman). Ele completa (`padEnd`) as sequências mais curtas com `-` no final até todas ficarem do mesmo tamanho, e depois compara **posição por posição**: `% = (posições iguais) / (posições comparáveis) × 100`. É simples e rápido, mas funciona melhor quando as sequências já estão "alinhadas" naturalmente (mesmo tamanho ou uma só um pouco maior que a outra) — se houver uma inserção/deleção no meio de uma sequência, a comparação a partir dali "desalinha" e a % de similaridade cai mesmo que a sequência seja parecida.

#### 3.2. Modo "Buscar Trecho" (caixa marcada)

Em vez de medir similaridade, esse modo **procura um trecho exato** dentro de uma ou várias sequências carregadas — útil pra achar um motivo, um domínio conhecido, um trecho de primer, etc.

**Como usar:**
1. Marque a caixa **"Modo Buscar Trecho"**.
2. Cole, no campo que aparece, o trecho que você quer encontrar (pode ser DNA, RNA, proteína ou qualquer sequência de letras — a busca não depende do botão DNA/RNA vs Proteína).
3. Carregue/digite uma ou várias sequências onde procurar (igual ao modo comparação).
4. Clique em **Comparar**. O resultado mostra, pra cada sequência em que o trecho foi encontrado:
   - A **ordem** da sequência na lista e o **nome** dela (do cabeçalho `>nome`, ou "Sequência N" se não tiver nome).
   - Quantas vezes o trecho aparece e, pra cada ocorrência, a **posição exata** (índice do caractere onde o trecho começa) e um **trecho de contexto** ao redor do match, com o trecho encontrado destacado.
   - Sequências onde não encontrou nada aparecem separadas, sem misturar com as que tiveram match.

A busca:
- ignora maiúscula/minúscula;
- ignora espaços, quebras de linha e qualquer caractere que não seja letra (então cola o texto do jeito que estiver que o app limpa sozinho);
- é por **trecho exato** — não é uma busca "aproximada"/fuzzy, então um erro de digitação no alvo não vai encontrar um trecho quase igual.

### 4. Ferramentas online

O app organiza links pra sites externos de bioinformática (não roda nada disso localmente, só abre no navegador padrão): bancos de peptídeos antimicrobianos (AMPs), alinhamento (Clustal Omega, MAFFT, BLAST), domínios conservados, peptídeo sinal, atividade antimicrobiana, localização subcelular, pontes dissulfeto, propriedades físico-químicas, motivos conservados, estruturas secundárias, docagem e modelagem.

Também tem uma aba **Ajuda**, com as regras de uso e como interpretar os resultados, direto no app.

## Como o código funciona (por dentro)

Sem framework — só HTML, CSS e JavaScript puro, organizado dentro de um objeto único `BioApp` no `src/app.js`, com sub-objetos por responsabilidade.

### `main.js` — processo principal do Electron

- Cria a janela (`BrowserWindow`) com `nodeIntegration: false`, `contextIsolation: true` e `sandbox: true` — a página não tem acesso direto a APIs do Node, o que é mais seguro.
- Intercepta qualquer tentativa de abrir um link (`setWindowOpenHandler` e `will-navigate`) e manda pro navegador padrão do sistema (`shell.openExternal`) em vez de navegar dentro da janela do app — é assim que os "atalhos pra ferramentas online" funcionam sem transformar o BioLyzer num navegador.
- Fecha o app quando todas as janelas fecham (exceto no macOS, por convenção).

### `src/index.html`

Toda a interface: abas, botões, textareas, tabelas de resultado e o CSS embutido. Os elementos são referenciados pelo `app.js` por `id` (ex: `dnaInput`, `searchModeCheckbox`, `similarityResult`) e por `class` (ex: `.seqInput` pra cada campo de sequência do comparador, `.page` pra cada aba).

### `src/app.js` — a lógica, dividida em blocos dentro de `BioApp`

- **`BioApp.fasta`** — utilitários de parsing compartilhados por todo o app:
  - `parse(content)`: lê um arquivo `.fasta` real, separando por `>`.
  - `cleanSequence(texto, regexValida)`: remove tudo que não é um caractere válido pro tipo de sequência esperado (ex: só `ATGC` pra DNA).
  - `parseLoose(texto, regexValida)`: separa um texto colado com **várias sequências** mesmo que o usuário não tenha usado `>nome` — detecta uma nova sequência quando aparece uma linha que não é uma sequência válida (interpretando-a como o novo "nome"), fechando a anterior.
- **`BioApp.navigation`** — troca de abas/submenus (mostra/esconde `div`s).
- **`BioApp.dnaRna`** — `convertToRNA()` e `convertToDNA()`: usam `fasta.cleanSequence`/`parseLoose`, trocam `T`↔`U` com regex, e montam a saída em FASTA.
- **`BioApp.protein`** — guarda a **tabela de códons** completa (todos os 64 códons → aminoácido, com o símbolo de uma letra) e `translateToProtein()`, que percorre a sequência de RNA de 3 em 3 caracteres e monta a proteína.
- **`BioApp.comparator`** — o núcleo do app:
  - `currentMode`: guarda se a interface está em modo "dnaRna" ou "protein" (só afeta cores/legenda).
  - `setMode(modo)`: troca `currentMode` e atualiza os estilos dos botões.
  - `addSequence()` / `removeSequence()`: adicionam/removem campos `.seqInput` dinamicamente.
  - `compareSequences()`: função principal, chamada pelo botão "Comparar". Primeiro verifica se o **modo Buscar Trecho** está ligado:
    - **Se sim** → roda a lógica de busca: limpa cada sequência pra só letras (`_lettersOnly`), procura todas as ocorrências do trecho-alvo com `indexOf` em loop, monta o contexto ao redor de cada match (`_buildMatchContext`) e renderiza a lista de resultados por sequência.
    - **Se não** → roda a comparação normal: monta `sequenceData` (nome + sequência limpa de cada input), completa (`padEnd`) todas até o mesmo tamanho, desenha a tabela posição-por-posição colorida, calcula a % de similaridade da primeira sequência contra as demais (`renderNormalResults`) e guarda o resultado (`_lastNormalResults`) pra poder reordenar sem recalcular.
  - `showMatrix()`: mesma ideia da comparação normal, mas calculando a % de similaridade **entre todos os pares** de sequências (matriz N×N), não só contra a primeira.

### Onde entram os arquivos `.fasta` de exemplo

`exempodna.fasta`, `exemporna.fasta` e `teste.fasta` (na raiz do projeto) e `teste_busca.fasta` (gerado durante o desenvolvimento pra testar o modo Buscar Trecho) servem só como massa de teste — não são carregados automaticamente, é preciso escolher o arquivo manualmente em cada aba.

## Estrutura de arquivos

```
Biolyzer/
├── main.js          # processo principal do Electron
├── package.json
└── src/
    ├── index.html   # interface e estilos
    └── app.js       # lógica do aplicativo
```

## Limitações conhecidas

- A comparação de similaridade é posição-por-posição (com padding no final), **não** um alinhamento com inserção de gaps no meio da sequência — sequências parecidas mas de tamanhos diferentes ou com uma inserção/deleção no meio podem dar uma % de similaridade mais baixa do que "deveriam" biologicamente.
- A busca de trecho é exata (sem tolerância a erros de digitação/mismatches).
- Alinhamentos muito longos (acima de 1000 posições) só mostram as primeiras 1000 colunas na tabela visual, por performance — o cálculo de % de similaridade continua usando a sequência inteira.

## Créditos

Desenvolvido por Leonardo José Figueiredo de Oliveira e Tarcio Tomé da Silva.

<!-- Escolha uma licença antes de publicar. Sem arquivo de licença, o código fica visível, mas os direitos continuam reservados. Para permitir o reuso, uma opção comum é a MIT: no GitHub, Add file > Create new file > nome "LICENSE" > "Choose a license template". -->
