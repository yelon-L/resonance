---
name: animejs-v4-svelte
description: Guide for integrating Anime.js v4 and Svelte 5 within Cloudflare Workers Assets templates. Use when building interactive animations, scroll-linked canvas nodes, or scaffolding new SvelteKit projects on Cloudflare.
---

# Anime.js v4 & Svelte 5 Integration Guide

## Svelte 5 & Cloudflare Non-interactive Project Creation

When scaffolding Svelte 5 with Cloudflare Adapter non-interactively using Svelte CLI (`sv create`), use the complete option configuration to bypass prompts:

```bash
npx -y sv@latest create ./ \
  --template minimal \
  --types ts \
  --add sveltekit-adapter="adapter:cloudflare+cfTarget:workers" \
  --install npm \
  --no-dir-check \
  --no-download-check
```

*Note: Generating Wrangler types file using `npx wrangler types` is required before building, to prevent compilation failure due to missing `worker-configuration.d.ts`.*

---

## Anime.js v4 API Changes (Environment Trap Mitigation)

Anime.js v4.x has migrated to a fully modular package with major breaking changes from v3.x.

### 1. Imports
Do **NOT** use default import. Always use named imports.
* **Incorrect**: `import anime from 'animejs';`
* **Correct**: `import { animate, createTimeline, stagger } from 'animejs';`

### 2. Basic Animation Syntax
The single-object argument is deprecated. The target(s) must be passed as the **first** parameter, and animation parameters as the **second** parameter. The property name is now `ease` instead of `easing`.
* **Incorrect**:
  ```typescript
  anime({
    targets: element,
    translateX: 250,
    easing: 'easeOutQuad'
  });
  ```
* **Correct**:
  ```typescript
  animate(element, {
    translateX: 250,
    ease: 'easeOutQuad'
  });
  ```

### 3. Timeline Animation Syntax
Initialize timelines using `createTimeline()` and chain items using `.add(targets, parameters, position)`.
* **Incorrect**:
  ```typescript
  anime.timeline({ easing: 'easeOutExpo' })
    .add({ targets: '.box', translateY: 100 });
  ```
* **Correct**:
  ```typescript
  createTimeline({ ease: 'easeOutExpo' })
    .add('.box', { translateY: 100 }, '-=100');
  ```

### 4. Custom rendering & Properties (onRender / onUpdate)
In Canvas loops, animate a proxy object and utilize `onRender` or `onUpdate` to read the transition progress.
* **Pattern**:
  ```typescript
  const proxy = { progress: 0 };
  animate(proxy, {
    progress: 1,
    duration: 1000,
    ease: 'linear',
    onRender: () => {
      // Access proxy.progress directly via closure
      const currentVal = proxy.progress;
    }
  });
  ```

---

## Svelte 5 Accessibility (a11y) Warnings
Svelte 5 compiles strictly and will raise errors for click handlers placed on non-interactive elements (like a `div` with `onclick`).
* Always prefer using `<button>` instead of a `<div onclick={...} role="button">` to ensure clean builds.
