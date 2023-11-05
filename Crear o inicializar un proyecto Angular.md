
## Instalación de Angular CLI

Para crear un proyecto de Angular necesitamos tener instalada la aplicación de comandos de línea o CLI

```powershell
npm install -g @angular/cli
```

Pero como la ejecución de script en `PowerShell` está deshabilitada por defecto antes deberemos habilitarla con 

```
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

## Inicializar un proyecto en Angular

Una vez instalado Angular CLI de forma global, como hicimos en el paso anterior, vamos a iniciar el proyecto de trabajo que nos configurará el entorno de Angular en la termina, :

```
ng new my-app
```

Nos preguntará si queremos añadir el módulo de Routing y que tipo de CSS queremos utilizar (CSS normal, SasS, ...) y una vez contestado todo pasará a generar todos los archivos que sirven de andamiaje (boilerplate) de nuestra aplicación.

## Arrancar una aplicación Angular

Para arrancar la aplicación debemos situarnos con la terminal en la carpeta raíz del proyecto y ejecutar:
```shell
cd my-app
ng serve --open
```

El *flag* `--open` abrirá un navegador en la dirección por defecto: `http://localhost:4200`
