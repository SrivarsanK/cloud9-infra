# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
npm run dev     # Start dev server on http://localhost:3000

# Build
npm run build   # Production build
npm run start   # Start production server

# Linting
npm run lint    # Run ESLint
```

## Architecture

- **Framework**: Next.js 16 (App Router)
- **Styling**: Tailwind CSS v4 with `@tailwindcss/postcss`
- **Language**: TypeScript with strict mode
- **Fonts**: Geist Sans + Geist Mono (via `next/font`)

## Key Patterns

- **Path aliases**: `@/*` maps to project root (e.g., `@/components` → `./components`)
- **Entry point**: `app/page.tsx` is the home page
- **Layout**: `app/layout.tsx` defines root layout with metadata and fonts
- **Styling**: Global styles in `app/globals.css` use CSS variables for theming
- **ESLint**: Flat config format (`eslint.config.mjs`) with Next.js + TypeScript presets

## Development Notes

- Server components by default (no `"use client"` unless interactivity needed)
- Hot reload on edits to `app/page.tsx`
- Build output in `.next/` (gitignored)
