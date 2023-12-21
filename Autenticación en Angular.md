
## ¿Cómo funciona?

Desde la parte del cliente mandamos al servidor los datos de autenticación para que se comprueben desde el servidor. Los datos de autenticación nunca se pueden situar en el lado del cliente ya que se pueden manipular e incluso desactivar.

En una página tradicional HTML trabajaríamos con una `Sesión`, pero con una SPA y usando una `API RESTful`, esta no guarda estado alguno de los clientes, por lo que no podemos hacer uso de `Sesiones`.

Para una SPA vamos a usar un `Web Token`, generado por el servidor a partir de un algoritmo y una frase secreta que sólo conoce el servidor. Las peticiones del lado del cliente se hacen con ese `Token`, y es el servidor el que va a validar el `Token` antes de dar respuestas en la API.

Entonces, como hemos dicho, ni los datos del cliente ni su validación se realizan desde el `FrontEnd`, y por tanto no se realizan desde Angular, pero eso no significa que no tengamos nada que hacer.

## ¿Entonces? ¿Qué podemos hacer desde Angular?

Primero, debemos tener los formularios y validaciones tanto para registrarse como para hacer `login`. Luego, una vez registrado o hecho `login`, como hemos dicho, si todo ha ido bien vamos a recibir nuestro `WebToken` y debemos guardarlo en el navegador del cliente para que recuerde nuestros datos de sesión. 

La autenticación, como hemos dicho, no puede restringir datos de ser vistos dentro del propio `frontend`, lo que se manda en el `frontend` está disponible para todo el mundo con los conocimientos suficientes para buscarlo. Pero lo que va a conseguir la autenticación es restringir, incluso rechazar, las peticiones que se hagan al `API RESTful` limitando así los datos y las funciones que la API puede realizar a petición del cliente.

Esta autenticación puede ser enviada en cada petición que se hace al servidor, para que el servidor sea el que compruebe si el cliente está o no correctamente autenticado, y dar una respuesta acorde a eso. Y ahí entra de nuevo Angular, ya que si hemos dicho que cada petición va a llevar estos datos de Autenticación, como puede ser el `WebToken` , aquí podemos hacer uso de algo que hemos visto en la sección de peticiones: `Interceptores`.

El uso de `interceptores` para agregar el `webtoken` nos va a ahorrar la tarea de tener que repetir el código de agregar dicho `token` a cada petición al `backend`, por lo que es una herramienta muy recomendable a usar en este campo.

Así que, resumiendo, desde Angular, la autenticación va a suponer formularios de `login`, registro, guardar `tokens` o `cookies` una vez recibida la respuesta del `backend` y luego agregar dicho `token` o `cookie` a cada petición que necesite autenticación, y si no la necesita no está de más agregarla también mediante el uso de un `interceptor`.

## Creación de formularios de registro y `login`.

No tienen nada de especial que no se haya visto ya en la sección de formularios. En todo caso, si nuestro `backend` al darnos error tras un intento de `login` o un registro fallido, debemos de gestionar la respuesta para darle al usuario la información que consideremos oportuna para su experiencia de usuario.

## Creando y guardando los datos del usuario

Para poder gestionar los datos del usuario una vez nos hemos registrado, ya que no es extraño tras el `login`, además del `webtoken` recibamos datos del propio usuario tales como su nombre, etc. que los tengamos en caché de nuestra aplicación Angular.

Para ello puede ser recomendable primero crear un modelo de Usuario para establecer los tipos de datos que se van a usar para ese usuario, para lo que vamos a crear una clase de Usuario:

```typescript
export class User {
  constructor(
    public email: string, 
    public id: string, 
    private _token: string, 
    private _tokenExpirationDate: Date
  ){}

  // Función getter: Se ejecuta cuando accedemos a la propiedad con el nombre de la función: User.token
  // Entonces es algo entre propiedad y método. Funciona como método pero se accede a ella como propiedad.
  get token() {
    
  }


}
```