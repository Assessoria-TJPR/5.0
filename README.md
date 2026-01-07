<div align="center">
<img width="1200" height="475" alt="GHBanner" src="https://github.com/user-attachments/assets/0aa67016-6eaf-458a-adb2-6e31a0763ed6" />
</div>

# Run and deploy your AI Studio app

This contains everything you need to run your app locally.

View your app in AI Studio: https://ai.studio/apps/drive/1RjGL6xAfQtkkJD-zhr57V1MYrOxLqjBC

## Run Locally

**Prerequisites:**  Node.js


1. Install dependencies:
   `npm install`
2. Set the `GEMINI_API_KEY` in [.env.local](.env.local) to your Gemini API key
3. Run the app:
   `npm run dev`

## Deploy to GitHub Pages

Build vai para `docs/` (Vite `base: './'`):

1. Build: `npm run build`
2. Publicar:
   - `npm run deploy:gh` (envia `docs/` para branch `gh-pages`)
   - ou `git subtree push --prefix docs origin gh-pages`
   - ou configure Pages para Source: `main` + folder `/docs`
   - ou habilite “GitHub Actions” (workflow `deploy.yml` já envia `docs/`)

Notes:
- `npm run build` gera `docs/404.html` (SPA fallback).
- `base: './'` garante assets em subpath `/repo-name/`.
