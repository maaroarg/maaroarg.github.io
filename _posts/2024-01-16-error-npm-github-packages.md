---
layout: post
title: "Error 401 con GitHub Packages en React Native"
date: 2025-01-16
categories: [react-native, npm, github]
tags: [npm, github-packages, troubleshooting, react-native]
author: "Mario Romero"
---

## El Problema

Durante la instalación de un proyecto React Native, me encontré con un error 401 al intentar instalar un paquete privado:

```bash
npm ERR! code E401
npm ERR! 401 Unauthorized - GET https://npm.pkg.github.com/download/@rigup/themes-react-native-paper/1.0.1/cc617bf4fbb89ec87bf450489f8607fb623e11d7 - unauthenticated: User cannot be authenticated with the token provided.
```

## El Diagnóstico

El problema estaba en el archivo `.npmrc` del proyecto, que contenía:

```bash
@rigup:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GITHUB_ACCESS_TOKEN}
```

La variable de entorno `GITHUB_ACCESS_TOKEN` contenía un token expirado.

## La Solución

1. Generar un nuevo Personal Access Token (PAT) en GitHub con los permisos `read:packages`

2. Configurar la variable de entorno de forma persistente en `~/.zshrc` (Mac con Zsh):

   ```bash
   echo 'export GITHUB_ACCESS_TOKEN=TOKEN_NUEVO' >> ~/.zshrc
   ```

   O en `~/.bashrc` si usas Bash:

   ```bash
   echo 'export GITHUB_ACCESS_TOKEN=TOKEN_NUEVO' >> ~/.bashrc
   ```

3. Recargar la configuración:
   ```bash
   source ~/.zshrc  # o source ~/.bashrc
   ```

## Lección Aprendida

Es importante notar que el comando `export` por sí solo:

```bash
export GITHUB_ACCESS_TOKEN=TOKEN_NUEVO
```

solo mantiene la variable durante la sesión actual de la terminal. Para una solución permanente, es necesario añadirla al archivo de configuración del shell.
