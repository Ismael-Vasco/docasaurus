# Website

This website is built using [Docusaurus](https://docusaurus.io/), a modern static website generator.

## Installation

```bash
yarn
```

## Local Development

```bash
yarn start
```

This command starts a local development server and opens up a browser window. Most changes are reflected live without having to restart the server.

## Build

```bash
yarn build
```

This command generates static content into the `build` directory and can be served using any static contents hosting service.

## Deployment

Using SSH:

```bash
USE_SSH=true yarn deploy
```

Not using SSH:

```bash
GIT_USER=<Your GitHub username> yarn deploy
```

If you are using GitHub pages for hosting, this command is a convenient way to build the website and push to the `gh-pages` branch.

#### DOCUMENTATION
https://gist.github.com/andrescortesdev/6f7cf37dd45e68dac7f78248e0fffb2a

https://tutorialmarkdown.com/guia

### Execute from 0 to deplay with firebase
Primero, necesitas instalar la herramienta de línea de comandos de Firebase de forma global:
```
1. npm install -g firebase-tools
```

Este comando nos permite iniciar sesión en Firebase usando nuestra cuenta de Google. Es necesario para poder desplegar el sitio en Firebase Hosting desde tu máquina loca
```
2. firebase login
```
Construye una versión optimizada y lista para producción de tu sitio Docusaurus. El resultado se guarda en la carpeta build/, que será la que Firebase publique
```.
3. npm run build
```
Inicia la configuración de Firebase en tu proyecto. Te guía paso a paso para vincular tu sitio con un proyecto de Firebase y preparar el entorno de despliegue.
```
4. firebase init
```
Este comando despliega tu sitio a Firebase Hosting. Toma el contenido de la carpeta configurada (por defecto build/) y lo publica en la web. Al finalizar, Firebase te mostrará la URL pública donde estará disponible tu sitio.
```
5. firebase deploy
```

#### Docasaurus URL
```url
https://project-firebase-a36d6.web.app
```