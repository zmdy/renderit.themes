# Renderit Themes: Guia Completo para Desenvolvedores e Designers

Bem-vindo à documentação oficial do ecossistema de temas e addons do **renderit.builder**. Este guia é destinado a desenvolvedores web e designers que desejam criar temas, componentes reutilizáveis (addons) e presets (samples) para a plataforma Renderit.

---

## 1. O Conceito de Tema no Renderit

No ecossistema Renderit, a definição de "Tema" difere de plataformas tradicionais como WordPress. Adotamos o princípio de que **"Templates são ativos permanentes. Dados são o que muda."**

Um Tema completo é composto por duas partes perfeitamente isoladas:

1. **O Template (HTML):** O esqueleto estático. Define o layout, a estrutura semântica e os espaços onde o conteúdo será injetado através de *Magic Keys* (ex: `%site_name%`).
2. **O Theme/Sample (JSON):** A alma do tema. Contém a configuração do **Design System** (cores, tipografia, bordas) e os dados reais estruturados em seções. É este JSON que transforma um esqueleto genérico em um "Tema de Restaurante" ou "Tema de Portfólio".

Desta forma, você pode criar um único template HTML incrivelmente flexível e gerar dezenas de "temas" visuais apenas alterando o arquivo JSON.

---

## 2. Magic Tags e o Design System

A magia da flexibilidade visual no Renderit acontece através das *Magic Keys* de design. O schema JSON gerado pelo builder possui um objeto obrigatório `design`, baseado na especificação padrão. 

### 2.1. Referenciando Variáveis de Design no HTML

Ao criar um template HTML, você **não deve** fixar cores (hardcode). Em vez disso, utilize as Magic Keys de design dentro de blocos `<style>` ou atributos, permitindo que o construtor substitua esses valores em tempo de build.

```html
<style>
  :root {
    /* Cores Primárias */
    --color-primary: %design.colors.primary%;
    --color-on-primary: %design.colors.on-primary%;
    
    /* Superfícies e Backgrounds */
    --color-surface: %design.colors.surface%;
    --color-on-surface: %design.colors.on-surface%;
    --color-neutral: %design.colors.neutral%;
    
    /* Feedback */
    --color-error: %design.colors.error%;
    --color-success: %design.colors.success%;

    /* Tipografia */
    --font-primary: %design.fonts.primary%;
    --font-secondary: %design.fonts.secondary%;

    /* Arredondamentos (Border Radius) */
    --radius-sm: %design.rounded.sm%;
    --radius-md: %design.rounded.md%;
    --radius-lg: %design.rounded.lg%;
    --radius-full: %design.rounded.full%;

    /* Sombras (Elevation) */
    --shadow-sm: %design.elevation.sm%;
    --shadow-md: %design.elevation.md%;
    --shadow-lg: %design.elevation.lg%;
    --shadow-xl: %design.elevation.xl%;
  }

  body {
    font-family: var(--font-primary);
    background-color: var(--color-surface);
    color: var(--color-on-surface);
  }

  h1 {
    font-size: %design.typography.h1.fontSize%;
    font-weight: %design.typography.h1.fontWeight%;
  }
</style>
```

### 2.2. A Importância da Abstração

Ao utilizar variáveis genéricas como `--color-primary`, você garante que o usuário no Wizard do `renderit.builder` possa alterar a cor de todo o site com um único clique no editor de Design System, propagando a alteração de forma segura.

---

## 3. Modos de Operação e Adaptação de Temas

O `renderit.builder` compila temas para três modos distintos. Ao criar um tema ou addon, você deve garantir que seu código funcione graciosamente nestes cenários.

### 3.1. Modo Static
Neste modo, o builder gera um site HTML/CSS/JS 100% estático. Todas as *Magic Keys* são substituídas definitivamente.
*   **O que o criador do tema deve saber:** O site gerado será incrivelmente rápido e pronto para ser servido por CDNs. Se o seu tema usa JavaScript para carrosséis, menus ou animações (como IntersectionObserver), todo esse JS será incluído no build.

### 3.2. Modo Live
Neste modo, o builder cria um "shell" estático e delega partes do HTML para hidratação no lado do cliente (via `renderit-live.js` e um Service Worker). Isso é ativado usando zonas.
*   **Como adaptar seu tema:** Se uma seção do tema exibe dados que mudam constantemente (ex: preços, disponibilidade), envolva essa seção com a diretiva `data-renderit-zone`.
    ```html
    <div data-renderit-zone="pricing-table">
      <!-- O conteúdo gerado pelas Magic Keys aqui será convertido em Base64 e 
           reidratado no runtime consumindo o JSON em tempo real -->
      %FOREACH pricing.items% ... %ENDFOREACH%
    </div>
    ```

### 3.3. Modo Manager
O Modo Manager compila o site de forma estática (como no Modo Static), mas injeta atributos especiais (`renderit_manager_area`, `renderit_manager_collection`, etc.) no HTML final.
*   **O que o criador do tema deve saber:** O motor de build cuida da injeção de atributos automaticamente. Você só precisa se preocupar em não modificar o DOM de forma destrutiva com JavaScript de terceiros que apague os atributos injetados pelo renderizador, pois o `renderit.manager` dependerá desses atributos para edição WYSIWYG.

---

## 4. Sistema de Addons

Addons são os "blocos de montar" do ecossistema Renderit. Eles são componentes funcionais reutilizáveis (sliders, formulários, mapas, temporizadores) que encapsulam HTML, CSS isolado e Vanilla JS.

### 4.1. Como usar Addons em um Tema

Para inserir um addon em um template HTML, basta usar a diretiva `%ADDON nome%`.

```html
<main>
  <!-- Estrutura nativa do tema -->
  <section class="hero">
    <h1>%hero.title%</h1>
  </section>

  <!-- Injetando um Addon de Formulário de Contato -->
  %ADDON form%
  
  <!-- Injetando um Addon de Galeria -->
  %ADDON gallery%
</main>
```

Durante a Fase de Pré-processamento, o `AddonManager` localizará o arquivo `addons/form.html`, injetará o código dele na posição especificada e exporá as *Magic Keys* do addon (ex: `%form.title%`, `%form.fields%`) para o Wizard do builder.

### 4.2. Usando JSONs Externos para Addons

Você pode instruir o builder a buscar os dados do addon a partir de um arquivo JSON externo, em vez de depender dos dados globais. Útil para reaproveitamento de componentes.

```html
%ADDON team src="data/leadership.json"%
```

---

## 5. Como Criar Addons: O Guia Definitivo

Criar um addon exige disciplina arquitetural. Como o código do addon será injetado diretamente no template do usuário final, existem regras de segurança e escopo que não podem ser quebradas.

### 5.1. Regras de Ouro (Checklist)

1.  **Tag de Identificação:** O arquivo *deve* começar com o comentário `<!-- Addon: nome-do-addon -->`.
2.  **Zero Dependências Locais:** Proibido usar módulos ES6 locais ou requerer bibliotecas via npm. Libs externas são permitidas **apenas** via CDN (ex: Anime.js, Leaflet).
3.  **Encapsulamento de CSS:** Todo CSS deve estar dentro de uma tag `<style>` e as classes **devem ser prefixadas** (ex: `.renderit-slider-container`) para evitar conflitos com o tema do usuário.
4.  **Escopo JavaScript (IIFE):** Todo script *deve* ser isolado em uma *Immediately Invoked Function Expression* (IIFE) para não poluir o escopo global.
5.  **Namespacing de Variáveis:** As *Magic Keys* criadas dentro do addon devem ser agrupadas sob o nome do addon. (ex: `%form.title%`, `%form.button_label%`).

### 5.2. Anatomia de um Addon (Exemplo Prático)

Vamos criar um addon chamado `toast-message` (`addons/toast-message.html`).

```html
<!-- Addon: toast-message -->
<!-- 1. Marcação HTML -->
<div id="renderit-toast-%INDEX%" class="renderit-toast-wrapper" data-duration="%toast.duration%">
  <div class="renderit-toast-icon">
    <!-- SVGs inline são bem-vindos -->
    <svg viewBox="0 0 24 24"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/></svg>
  </div>
  <div class="renderit-toast-content">
    <h4 class="renderit-toast-title">%toast.title%</h4>
    <p class="renderit-toast-desc">%toast.description%</p>
  </div>
</div>

<!-- 2. Estilos Isolados e Dependentes do Design System -->
<style>
  :root {
    --toast-bg: %design.colors.surface%;
    --toast-text: %design.colors.on-surface%;
    --toast-accent: %design.colors.success%;
    --toast-radius: %design.rounded.md%;
    --toast-shadow: %design.elevation.lg%;
    --toast-font: %design.fonts.primary%;
  }

  .renderit-toast-wrapper {
    position: fixed;
    bottom: 20px;
    right: 20px;
    background: var(--toast-bg);
    color: var(--toast-text);
    border-radius: var(--toast-radius);
    box-shadow: var(--toast-shadow);
    font-family: var(--toast-font);
    display: flex;
    align-items: center;
    padding: 16px;
    z-index: 9999;
    transform: translateY(100px);
    opacity: 0;
    transition: transform 0.4s ease, opacity 0.4s ease;
  }

  /* Classes ativas para JavaScript */
  .renderit-toast-wrapper.is-visible {
    transform: translateY(0);
    opacity: 1;
  }
  
  /* Prefixos consistentes evitam vazamento de estilo */
  .renderit-toast-icon svg {
    fill: var(--toast-accent);
    width: 24px;
    height: 24px;
    margin-right: 12px;
  }
</style>

<!-- 3. Lógica JavaScript Isolada -->
<script>
(function() {
  // Evitar redeclaração se o addon for usado multiplas vezes
  // %INDEX% é convertido no ID numérico do loop se estiver dentro de um FOREACH,
  // ou resolvido vazio se não estiver, garantindo unicidade básica.
  const elementId = 'renderit-toast-%INDEX%';
  const toastEl = document.getElementById(elementId);
  
  if (!toastEl) return;

  // Lógica atômica e Vanilla JS
  const durationStr = toastEl.getAttribute('data-duration');
  const duration = parseInt(durationStr) || 3000;

  function showToast() {
    toastEl.classList.add('is-visible');
    
    setTimeout(() => {
      toastEl.classList.remove('is-visible');
    }, duration);
  }

  // Exemplificando gatilho: mostrando ao carregar a página
  window.addEventListener('DOMContentLoaded', () => {
    // Atraso sutil para permitir a renderização inicial
    setTimeout(showToast, 500); 
  });
})();
</script>
```

### 5.3. Integração com o Workflow de Testes

Ao criar um addon em `renderit.themes`, você **deve** prover um arquivo JSON com dados realistas (Mock/Fixture) em `tests/fixtures/addon-nome-do-addon.json`. O builder usa isso para validações de AST e testes E2E.

**Exemplo (`tests/fixtures/addon-toast-message.json`):**
```json
{
  "toast": {
    "title": "Configuração Salva",
    "description": "Suas preferências foram atualizadas com sucesso.",
    "duration": 5000
  }
}
```

---

## Conclusão

Criar para o Renderit é adotar uma arquitetura de blocos de montar puros (Vanilla JS + HTML). Mantenha seus templates agnósticos usando as variáveis do Design System, mantenha seus addons fechados e prefixados, e confie que o `renderit.builder` compilará esses blocos em uma fundação sólida, veloz e pronta para qualquer modelo de distribuição (Estático, Dinâmico ou Gerenciável).
