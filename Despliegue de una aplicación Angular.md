Vamos a ver cómo se hace un despliegue en un servidor.

## Preparación del despliegue

* Debemos comprobar nuestras variables de entorno
* Revisar el código
* Comprobar la aplicación
* Construir el paquete de instalación. (`ng build`)
* Desplegar el paquete a un servidor estático
* Comprobar todo tras el despliegue.

## Comprobar las variables de entorno.

En Angular, tenemos ya preinstalada una carpeta, dentro de `/src` llamada `/environments` con dos archivos: `environment.ts` y `environment.prod.ts` que definen un objeto llamado `environment` donde se almacenan nuestras variables de entorno, al que le podemos añadir los pares propiedad-valor que queramos.

```typescript
export const environment = {
  production: true,
}

```

Se usan dos archivos distintos para que podamos usar dos configuraciones diferentes: una para desarrollo y otra para producción.

Se pueden guardar cosas como llaves de APIs, direcciones a `backends` diferentes, etc.

Para poder usar las variables de entorno dentro de un componente, por ejemplo, debemos de importar el objeto `environment` dentro de ese componente:

```typescript
import { environment } from '../../environments/environment';

```

e importaremos el de desarrollo, pero se cambiará automáticamente al de producción cuando hagamos la compilación de producción.

## Compilación para producción

En nuevas versiones nuestro comando en la terminal será `ng build`, pero en versiones anteriores era necesario añadir `--prod` para indicar que era una compilación de producción.

Esto generará nuestros archivos dentro de la carpeta `/dist` que son los archivos que debemos mover a nuestro servidor estático.

Si usamos una aplicación sin SSR debemos configurar nuestro servidor para que siempre sirva el archivo `index.html`, mientras que si usamos SSR no será así, sino que tendremos nuestras diferentes rutas en carpetas, compiladas.

En la última versión de Angular, el uso de SSR se compila por defecto, por lo que dentro de nuestra carpeta `/dist` encontraremos dos subcarpetas, una con SSR y otra sin él y ya nosotros elegimos cual de las carpetas queremos desplegar en nuestro servidor.
