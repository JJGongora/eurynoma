# Documentación - Hephaestus Server

- [Introducción](#introducción)
- [Configuración inicial](#configuración-inicial)
    - [Particionado de discos](#particionado-de-discos)
    - [Usuarios](#usuarios)
    - [Grupos](#grupos)
    - [Configuración de red](#configuración-de-red)
- [Configuración del servidor](#configuración-del-servidor)
    - [Node.JS](#nodejs)

## Configuración del servidor

### Node.JS

Para instalar Node.js, sigue las instrucciones de [Node.js installation](https://nodejs.org/es/download/current), usando NVM con NPM o sigue los pasos a continuación:

1. Descargar e instalar NVM:
~~~
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
~~~

2. Ejecuta el siguiente comando para evitar reiniciar la shell:
~~~
\. "$HOME/.nvm/nvm.sh"
~~~

3. Descarga e instala Node.js:
~~~
nvm install 24
~~~

4. Verifica que se ha instalado Node.js:
~~~
node -v
~~~

5. Verifica la versión de NPM:
~~~
npm -v
~~~
