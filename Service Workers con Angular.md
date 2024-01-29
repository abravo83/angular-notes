
## ¿Qué son `Service Workers`?

Cuando cargamos una página web con `Javascript` en nuestro navegador, este trabaja en un único hilo de procesamiento al que vamos a llamar `hilo principal`. Sin embargo, el navegador nos permite agregar otra entidad que vaya en un hilo de procesamiento diferente y que se llama `Web Worker`.

Un `Web Worker` es un tipo especial de  `Service Worker`, que nos permite trabajar de forma paralela al navegador recibiendo notificaciones `push` o podemos hacer que escuche a cualquier petición saliente del hilo principal y puede ir almacenando en caché las respuestas y que si nos quedamos sin conexión sea capaz de devolvernos la petición almacenada en su caché.

## ¿Cómo agrego uno a mi proyecto Angular?

Tenemos disponible a través de la CLI de Angular  un complemento llamado `PWA` que nos va a permitir agregar de manera sencilla un `Service Worker`:

```bash
ng add @angular/pwa
```

Esta va a coger nuestro proyecto actual y va a preconfigurar un `Service Worker` en él.

Nos va a incluir la importación en `AppModule` la importación del módulo `ServiceWorkerModule` de `@angular/service-worker` donde lo agrega al array de `imports` y registra `ngsw-worker.js`:

```typescript
imports: [..., ServiceWorkerModule.register('/ngsw-worker.js', { enabled: environment.production })],
```

y en nuestro `angular.json` se modificará el modo de `production` con la propiedad `"service-worker" : true`

Para ver nuestro `Service Worker` en acción es mejor hacerlo funcionar en un servidor http sencillo (live-server, etc) mejor que usar `ng serve`.

## Haciendo que nuestro `Service Worker` almacene contenido estático para servirlo si estamos `offline`.

 Por ejemplo, si queremos agregar una fuente que cargamos desde una URL externa debemos modificar nuestro `ngsw-config.json` para incluir en los `assetGroups.resources` una nueva propiedad `"urls"`que acepta un array de `strings`:
 
```json
"urls": [
  "https://fonts.googleapis.com/css?faimily=Oswald:300,700"
]
```


## Haciendo que nuestro `Service Worker` almacene contenido dinámico (de una API, por ejemplo)

En nuestro archivo `ngsw-config.json` debemos agregar una nueva propiedad al mismo nivel de nuestro `assetGroups`, y esta propiedad se va a llamar `dataGroups` que va a aceptar un array de objetos con las siguientes propiedades: 

```json

}],
"dataGroups" : [
  {
    "name" : "posts",
    "urls" : [
      "https://jsonplaceholder.typicode.com/posts"
    ],
    "cacheConfig": {
      "maxSize": 5,  // Cantidad máxima de peticiones a almacenar
      "maxAge": "6h",  // Si han pasado más de 6 horas el contenido se renovará
      "timeout": "10s",  // Tiempo que espera antes de ir a buscar el contenido en caché
      "strategy": "freshness"  //freshness | performance - 
    }
  }
]

```


Y con esto, si no tenemos la conexión y si previamente se ha cargado en caché, tendremos disponibles las respuestas.