# Como os Temas Funcionam no Renderit

O motor do **renderit.builder** foi desenhado com um princípio inegociável: **A separação absoluta entre Estrutura e Conteúdo.**

Em ecossistemas clássicos (como WordPress ou frameworks monolíticos), o código visual frequentemente se mistura à lógica de banco de dados e à gestão de estado. No Renderit, a filosofia é diferente: **Templates são ativos permanentes. Dados são mutáveis.**

Este documento detalha exatamente como o motor junta essas duas partes para gerar um site final.

---

## 1. A Anatomia Dupla de um Tema

No Renderit, o que chamamos informalmente de "Tema" é, na verdade, a fusão de dois artefatos independentes durante a etapa de *build*:

### A. O Template (HTML)
O template é o esqueleto da página. Ele é composto por marcação HTML estática pontuada por **Magic Keys** (`%...%`). O template não contém "informação do cliente". Ele define *onde* a informação vai aparecer.

*   **Responsabilidades:** Layout grid, espaçamentos macro, marcação semântica (`<header>`, `<main>`, `<footer>`), acessibilidade estrutural, injeção de Addons.
*   **Permutabilidade:** O mesmo template HTML de um restaurante pode ser usado para uma clínica veterinária, contanto que o JSON seja substituído.

### B. O Sample / Theme Configuration (JSON)
O arquivo JSON é a alma do site. É ele que preenche os espaços em branco deixados pelo template.

*   **Responsabilidades:** Dados textuais, URLs de imagens, configurações de repetição (itens de um slider, membros da equipe) e, cruciamente, o **Design System**.
*   **O Objeto `design`:** O JSON carrega variáveis estruturais que governam as cores, fontes e elevações do tema.

---

## 2. O Pipeline de Construção (O que acontece "por baixo do capô")

Quando o usuário clica em "Build" no Wizard, o `renderit.builder` não faz uma simples substituição de strings (RegEx). Ele executa um compilador *Vanilla JS* em três etapas rigorosas:

### Passagem 1: Pré-processamento (AddonManager)
Antes do motor olhar para as variáveis de texto, ele resolve a estrutura. O `AddonManager` varre o template em busca de tags `%ADDON nome%`.
1.  Ele busca o HTML correspondente localmente (`/addons/nome.html`) ou remotamente no repositório.
2.  Ele **injeta o código HTML bruto do addon no lugar da tag**.
3.  Opcionalmente, se a tag for `%ADDON nome src="dados.json"%`, ele também atrela contextos de dados extras.

Ao final desta fase, o template é um documento gigante de HTML sem nenhum `%ADDON%` restante, pronto para ter os dados injetados.

### Passagem 2: Análise Sintática (Lexer → Parser)
O HTML gigante é transformado em uma Árvore de Sintaxe Abstrata (AST).
*   O **Lexer** divide o texto: "Isto é HTML", "Isto é uma Magic Key".
*   O **Parser** entende a lógica estrutural: "Isto é um loop `%FOREACH%`", "Isto é uma condicional `%IF%`". Se houver erro de sintaxe (um loop não fechado), o Parser aborta com erro explícito.

### Passagem 3: Renderização (Renderer)
A AST é unida ao arquivo JSON. O motor viaja pelos "galhos" da árvore:
*   Se for um nó de variável (ex: `%hero.title%`), ele busca em `json.hero.title`. Se não existir, imprime vazio (fail-safe).
*   Se for um nó `%FOREACH items%`, ele clona o interior do bloco X vezes (uma para cada item no array `json.items`).

---

## 3. Comportamento em Diferentes Modos

A beleza do sistema é que o *mesmo template* e o *mesmo JSON* podem gerar sites com naturezas totalmente diferentes, dependendo do Modo selecionado no passo 1 do Wizard:

### 3.1. Static Mode
**Gera:** Um arquivo `.html` limpo.
*   Todas as Magic Keys são substituídas pelo valor de texto definitivo.
*   Nenhum JavaScript de framework ou de hidratação do Renderit é anexado. O site resultante depende apenas do browser nativo.

### 3.2. Live Mode
**Gera:** Um "Shell" estático com lacunas preparadas para hidratação assíncrona.
*   Se o desenvolvedor do tema marcou uma tag com `data-renderit-zone="xyz"`, o Renderer **não** preenche essa seção estaticamente.
*   O código original (com as Magic Keys) é convertido em Base64 e salvo em um atributo.
*   O script `renderit-live.js` e um Service Worker são acoplados ao output. Quando o usuário acessa o site, a zona consulta o JSON do servidor em tempo real e re-renderiza aquela porção isolada no lado do cliente.

### 3.3. Manager Mode
**Gera:** HTML Estático super-anotado para integração com o CMS.
*   Renderiza o site da mesma forma que o Static Mode, garantindo indexação e velocidade máximas.
*   Aplica um pós-processamento: Onde quer que uma Magic Key tenha sido usada (ex: `<h1>%title%</h1>`), o motor injeta um rastreador: `<h1 renderit_manager_area="title">Meu Título</h1>`.
*   Estes atributos invisíveis são a "ponte" que permite ao `renderit.manager` (o CMS do ecossistema) saber exatamente onde clicar para editar os textos do site em produção.

---

## 4. O Sistema de Design Dinâmico

Para que um tema seja verdadeiramente customizável, o desenvolvedor do template não deve usar valores CSS diretos (`#FF0000` ou `16px`). Ele deve se amarrar ao objeto `design` do JSON.

Durante o build, o `<style>` do template pode conter:
```css
:root {
  --color-primary: %design.colors.primary%;
  --radius: %design.rounded.md%;
}
```

Isso garante que um painel de UI (como o Passo 4 do Wizard do Builder) possa oferecer um "Seletor de Cor" simples. A troca do valor no JSON muda magicamente a aparência de todo o site, de botões de Addons até backgrounds principais.

## Resumo

Um Tema no Renderit é a promessa de que o HTML (Template) e o JSON (Sample) podem viver separadamente. Esta fundação é o que torna o *renderit.builder* uma ferramenta capaz de entregar sites estáticos ultrarrápidos e, simultaneamente, prontos para sistemas dinâmicos e editores CMS complexos.
