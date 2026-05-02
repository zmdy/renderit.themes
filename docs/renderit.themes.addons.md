# Sistema de Addons do Renderit: Guia de Criação e Uso

Os Addons são a espinha dorsal da modularidade no ecossistema **renderit.builder**. Eles permitem que desenvolvedores criem componentes complexos, interativos e estilizados — como sliders, mapas, carrosséis ou formulários — que podem ser injetados em qualquer template HTML com uma única linha de código.

Este guia detalha como consumir addons existentes em seus temas e, principalmente, as regras arquiteturais estritas para criar novos addons que funcionem perfeitamente dentro do motor de compilação.

---

## 1. O que é um Addon no Renderit?

Um Addon não é um plugin de CMS tradicional. É simplesmente um arquivo `.html` que contém:
1.  **Marcação (HTML):** Estrutura pontuada por *Magic Keys*.
2.  **Estilo (CSS):** Regras encapsuladas para a aparência do componente.
3.  **Comportamento (JavaScript):** Lógica isolada para interatividade.

A beleza do sistema é que o Addon é resolvido *antes* do código chegar ao Lexer/Parser. O `AddonManager` busca o componente, injeta seu código bruto no template principal e extrai as *Magic Keys* internas para o Wizard (permitindo que o usuário preencha os dados do addon).

---

## 2. Como Usar Addons em um Tema

Para o desenvolvedor de um Tema, usar um addon é trivial.

### A Sintaxe Básica
Em qualquer lugar do seu `index.html`, declare a diretiva:
```html
%ADDON whatsapp%
```
O builder procurará por `addons/whatsapp.html` localmente e, se não encontrar, buscará no repositório oficial no GitHub.

### B. Múltiplas Instâncias
Você pode usar o mesmo addon várias vezes. O builder é inteligente o suficiente para não quebrar IDs ou estilos, desde que o addon tenha sido construído seguindo as regras de escopo (explicadas abaixo).

### C. Usando Dados Externos Específicos
Por padrão, o Wizard pedirá ao usuário para preencher os dados do addon globalmente no site. No entanto, se você quiser forçar um addon a usar uma base de dados pré-configurada, use o atributo `src`:
```html
<!-- Injeta o addon 'team' e instrui o builder a preencher os dados 
     usando o arquivo leadership.json da pasta do projeto -->
%ADDON team src="data/leadership.json"%
```

---

## 3. Como Criar Novos Addons (Guia de Desenvolvimento)

Criar addons exige aderência a regras rígidas de arquitetura (definidas no `CLAUDE.md`). Como o seu código será injetado diretamente no DOM do usuário, um addon mal codificado pode destruir o site inteiro.

### 3.1. As Regras Absolutas de Arquitetura

1.  **Tag de Identificação:** A linha 1 do arquivo **deve** ser o comentário de identificação:
    ```html
    <!-- Addon: nome-do-addon -->
    ```
2.  **Proibição de Dependências Locais (`src/`):** O addon não pode fazer `import` ou `require` de módulos locais ou node_modules. Se você precisa de uma biblioteca (como *Anime.js* ou *Leaflet*), ela deve ser carregada via **CDN oficial**.
3.  **Escopo JavaScript (IIFE):** É proibido poluir o escopo global do browser (`window`). Todo script deve ser envolvido em uma Immediately Invoked Function Expression.
    ```javascript
    <!-- ❌ PROIBIDO: -->
    const btn = document.querySelector('.btn'); 

    <!-- ✅ OBRIGATÓRIO: -->
    (function() {
      const btn = document.querySelector('.btn');
    })();
    ```
4.  **CSS Encapsulado:** Não estilize tags genéricas (`h1`, `div`). Use um prefixo consistente para todas as classes do seu addon (ex: `.renderit-gallery-container`).

### 3.2. A Regra do `%INDEX%` (Essencial para loops)
Se o seu addon for colocado dentro de um `%FOREACH%` pelo usuário, o HTML dele se repetirá várias vezes. Para garantir que o seu JavaScript saiba qual elemento manipular, use a tag especial `%INDEX%` nos IDs e `data-attributes`.

```html
<div id="meu-addon-%INDEX%" class="renderit-meu-addon">...</div>
<script>
(function() {
  // Pega exatamente a instância atual injetada pelo motor
  const el = document.getElementById('meu-addon-%INDEX%');
})();
</script>
```

---

## 4. Exemplo Completo: Addon de Contador (Counter)

Abaixo, a estrutura real e aprovada de um addon simples que anima números quando aparecem na tela (IntersectionObserver).

**Arquivo: `addons/counter.html`**
```html
<!-- Addon: counter -->
<div id="renderit-counter-%INDEX%" class="renderit-counter" data-target="%counter.value%">
  <div class="renderit-counter-number">
    <span class="prefix">%counter.prefix%</span>
    <span class="value js-value">0</span>
    <span class="suffix">%counter.suffix%</span>
  </div>
  <div class="renderit-counter-label">%counter.label%</div>
</div>

<style>
  /* Uso do Design System global */
  .renderit-counter {
    text-align: center;
    font-family: %design.fonts.primary%;
    color: %design.colors.on-surface%;
    padding: 20px;
  }
  .renderit-counter-number {
    font-size: %design.typography.h2.fontSize%;
    font-weight: 800;
    color: %design.colors.primary%;
  }
  .renderit-counter-label {
    font-size: %design.typography.body-md.fontSize%;
    text-transform: uppercase;
    letter-spacing: 1px;
    margin-top: 8px;
    opacity: 0.8;
  }
</style>

<script>
(function() {
  const container = document.getElementById('renderit-counter-%INDEX%');
  if (!container) return;

  const valueEl = container.querySelector('.js-value');
  const target = parseInt(container.getAttribute('data-target')) || 0;
  
  function animateNumber() {
    let startTimestamp = null;
    const duration = 2000;
    const step = (timestamp) => {
      if (!startTimestamp) startTimestamp = timestamp;
      const progress = Math.min((timestamp - startTimestamp) / duration, 1);
      // Easing simple easeOutQuart
      const easeProgress = 1 - Math.pow(1 - progress, 4);
      valueEl.innerText = Math.floor(easeProgress * target);
      if (progress < 1) window.requestAnimationFrame(step);
    };
    window.requestAnimationFrame(step);
  }

  // Só anima quando entra na tela
  const observer = new IntersectionObserver((entries) => {
    if (entries[0].isIntersecting) {
      animateNumber();
      observer.unobserve(container);
    }
  });
  observer.observe(container);
})();
</script>
```

---

## 5. Testes e Validação (Fixtures)

Se você estiver desenvolvendo no repositório `renderit.themes`, há um contrato social e arquitetural: **Todo addon deve ter um arquivo de teste correspondente (Fixture).**

O motor de testes do `renderit.builder` usará este JSON para simular a injeção do seu addon.

Para o addon de `counter` acima, você deve criar o arquivo `tests/fixtures/addon-counter.json`:
```json
{
  "counter": {
    "value": 1500,
    "prefix": "+",
    "suffix": "M",
    "label": "Usuários Ativos"
  }
}
```

Isso garante que, a cada *commit*, o engine consiga renderizar seu addon e assegurar que as chaves coincidem e que não há erros de parse no HTML que você construiu.
