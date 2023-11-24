
Las peticiones HTTP que nos permiten comunicarnos con un `backend` que nos va a dar acceso a los datos y archivos del servidor tales como Bases de Datos, imágenes, etc.

## Anatomía de una petición HTTP

*  URL o `Endpoint` de la API

```
/posts/1
```

* Verbo de la petición:

```
POST, PUT, DELETE, GET...
```
 
 `GET` es para solicitar datos existentes
 `POST` es para crear nuevos datos
 `PUT` es para actualizar datos existentes
 `DELETE` es para borrar datos existentes
 
* Cabeceras de la petición:

	Añaden determinada información a la petición, como credenciales o como tipo de respuesta que espera.

* Cuerpo de la petición:

	Contienen los datos que estamos enviando al servidor para, por ejemplo, crear un nuevo dato en el servidor.

## ¿Cómo nos permite Angular realizar peticiones HTTP?

Primero creamos un `backend` para pruebas usando `Firebase` > `Real Time Database` > (`Testing Mode`: Sin Autenticación)

Una vez tengamos el enlace de nuestro `backend` lo copiamos para nuestro proyecto:

```
https://angular-course-db3f3-default-rtdb.europe-west1.firebasedatabase.app/
```


