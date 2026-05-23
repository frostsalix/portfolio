# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```sh
npm run dev          # Vite dev server (host 0.0.0.0:5173)
npm run build        # Production build: type-check then vite build
npm run preview      # Vite preview server
npm run p            # build + preview
npm run lint         # ESLint with auto-fix
npm run format       # Prettier on src/
npm run c            # lint + format
npm run type-check   # vue-tsc type checking only
```

There are no tests in this project.

## Architecture

This is a personal portfolio SPA built with **Vue 3 + TypeScript + Vite 7**. It has no routing — all sections live on a single scrollable page.

### Component hierarchy

```
App.vue
└── MainView.vue
    └── LayoutView.vue          ← scroll-driven backgrounds, modal system, back-to-top
        ├── TitleBar.vue        ← fixed top bar, appears when scrolled past hero
        ├── HeaderView.vue      ← hero section (avatar, terminal-style intro, app links)
        ├── AboutView.vue       ← "About Me" + CSS column masonry photo gallery
        ├── WorkView.vue        ← "My Work" + GitHub chart + project cards grid
        └── FotterView.vue      ← social links, contact button, large brand text
```

### Key patterns

- **`@` alias** resolves to `./src` (configured in both `vite.config.ts` and `tsconfig.app.json`).
- **Dependency injection**: `LayoutView` calls `provide('openModal')` to expose an image modal. `HeaderView` and `FotterView` consume it via `inject`. Contact/QQ buttons open a QR code image in this modal.
- **Scroll animations**: Components use `IntersectionObserver` to add `.animate-in` class triggering CSS `translateY` + `opacity` transitions. `LayoutView` uses `requestAnimationFrame` to lerp background overlay opacities based on scroll position.
- **PWA**: `vite-plugin-pwa` with `registerType: 'autoUpdate'`. The SW is enabled in dev mode. `main.ts` manually triggers SW updates on visibility change and hourly intervals.
- **Vite build**: Manual chunks split into `vue-vendor` (vue, vue-router, pinia) and `utils-vendor` (axios). No router is currently used despite vue-router being a dependency.
- **Fonts**: Google Sans Code and Google Sans Flex loaded from Google Fonts CDN in `font.css`. A local `ZhuZiAWan.woff2` is in static assets but not referenced in CSS.
- **Analytics**: 51.la (`//sdk.51.la`) and Umami in `index.html`.

### Avatar system

`src/assets/avatar.ts` exports a 79×36 2D RGB pixel array. `HeaderView.vue` renders it as colored `█` characters (`span.pixel`) into a container, batched at 60 pixels per rAF frame, creating a typewriter-style pixel-art effect. The companion `avatar2rgb.py` script (Python) generates this array from `input.jpg`.

### SEO

Static meta tags (OG, Twitter Card, canonical, hreflang) in `index.html`. A `generate-seo` script (`jiti scripts/generate-seo.ts`) exists but the script file is not present.

## Code style

- ESLint flat config via `@vue/eslint-config-typescript` with Prettier skip-formatting
- Rule: blank line required before/after function and class declarations
- `@typescript-eslint/no-explicit-any` is off
- Prettier: single quotes, 100 print width, semicolons, trailing commas (ES5), `bracketSameLine: true`
- TypeScript `noImplicitAny: false` (project-wide in `tsconfig.json`)
