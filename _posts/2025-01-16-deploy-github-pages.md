---
layout: post
title: "Cómo deployar GitHub Pages desde un branch específico"
author: "Tu Nombre"
date: 2025-01-16 14:00:00 -0500
categories: jekyll github-pages tutoriales
---

Hoy me encontré con un problema interesante: necesitaba deployar mi blog de Jekyll en GitHub Pages, pero desde un branch específico en lugar del tradicional `main`. Aquí está la solución paso a paso.

## El Problema

Por defecto, GitHub Pages espera usar el branch `main` para hacer el deploy del sitio. Sin embargo, a veces queremos:

- Probar cambios en un branch separado
- Mantener diferentes versiones del sitio
- Trabajar en una renovación completa sin afectar el sitio actual

## La Solución

La solución involucra dos partes: configurar GitHub Actions y crear un workflow específico para Jekyll.

### 1. Habilitar GitHub Actions

1. Ve a tu repositorio en GitHub
2. Navega a Settings → Pages
3. En "Source", selecciona "GitHub Actions"

### 2. Crear el Workflow

Crea un nuevo archivo en tu repositorio en la siguiente ruta:
`.github/workflows/jekyll-gh-pages.yml`

```yaml
name: Deploy Jekyll site to Pages

on:
  push:
    branches: ["nombre-de-tu-branch"] # Cambia esto a tu branch
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Build with Jekyll
        uses: actions/jekyll-build-pages@v1
        with:
          source: ./
          destination: ./_site
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 3. Comandos Git necesarios

Si estás creando esto desde la línea de comandos:

```bash
# Crear el directorio para el workflow
mkdir -p .github/workflows

# Añadir y commitear los cambios
git add .github/workflows/jekyll-gh-pages.yml
git commit -m "Add GitHub Actions workflow"
git push origin tu-branch
```

## Verificación

Una vez que hayas pusheado los cambios:

1. Ve a la pestaña "Actions" en tu repositorio
2. Verás el workflow ejecutándose
3. Si todo está correcto, tu sitio se publicará en `username.github.io`
4. El proceso completo puede tomar unos minutos

## Configuración de Reglas de Despliegue

Un error común es recibir el mensaje:

```
Branch "your-branch" is not allowed to deploy to github-pages due to environment protection rules.
```

Para solucionarlo, configura las reglas de despliegue usando patrones:

1. Ve a Settings -> Environments
2. Click en "github-pages"
3. En "Deployment branches":
   - Click en "Add deployment branch rule"
   - Selecciona "Custom branch name pattern"
   - Usa patrones como:
     - `*-blog` - permite cualquier branch que termine en -blog
     - `feature/*` - permite cualquier branch que comience con feature/
     - `deploy/*` - permite cualquier branch que comience con deploy/
   - Click en "Save rules"

Esto te da la flexibilidad de crear múltiples branches de desarrollo sin necesidad de actualizar las reglas cada vez.

## Troubleshooting

Si el deploy falla, los pasos más comunes a revisar son:

- Asegurarte que el nombre del branch en el workflow coincida con tu branch actual
- Verificar que todos los plugins necesarios estén en el `_config.yml`
- Revisar que la estructura del sitio Jekyll sea correcta

## Conclusión

Usar GitHub Actions para deployar desde un branch específico nos da más flexibilidad para trabajar en nuestro sitio. Podemos mantener múltiples versiones y probar cambios sin afectar el sitio en producción.
