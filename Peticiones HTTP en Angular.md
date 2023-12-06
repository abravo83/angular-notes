
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

También vamos a tener que crear los `endpoints` en nuestra API en Firebase
```
https://angular-course-db3f3-default-rtdb.europe-west1.firebasedatabase.app/posts.json //.json es un requisito de Firebase (supongo que para especificar que tipo de datos recibe)
```


## Configurar Angular para realizar peticiones HTTP

Para poder realizar peticiones HTTP a través de Angular necesitamos usar un módulo nativo llamado `HttpClientModule`, para lo cual vamos a importarlo en nuestro `AppModule`:

```typescript
...
import { HttpClientModule } from '@angular/common/http';
...

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, HttpClientModule],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

Una vez hemos hecho esto ya tendremos disponible para inyectar en nuestro componente el módulo

```typescript
...
import { HttpClient } from '@angular/common/http'
...
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {

  constructor(private http: HttpClient){}
}

```

Ahora, a través de nuestra propiedad privada `http` del componente tenemos a disposición los diferentes métodos que se corresponden con los verbos de las peticiones HTTP.

## ¿Cómo se realizan peticiones?

Para las diferentes peticiones HTTP usando `HttpCliente` tenemos disponibles los métodos `.get(), .delete(), .post(), etc...`

Además, para que la petición se mande, debemos **subscribirnos** a la respuesta, que va a ser un `Observable`.

```typescript
export class AppComponent implements OnInit {

  postsCargados = [];

  constructor(private http: HttpClient){}

  ngOnInit(){
    this.http.get('https://rutaAl/endpoint')
      .subscribe( datos => {
        this.postCargados = datos;
      }, error => {
        console.log(error);
      })
  }
}

```


## Usando un servicio para nuestras peticiones HTTP

Es una buena práctica trasladar las peticiones http a nuestros servicios. La razón principal sería la de disponer de estas peticiones de forma centralizada para distintos componentes, pero, ya que algunas de nuestras peticiones que se usen en diferentes componentes se van a usar en servicios, por pura organización es mejor tener todas nuestras peticiones en servicios.

```typescript
//En mi servicio

import { Injectable } from '@angular/core';
import { Post } from './Modelos/post.model';
import { map } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})

export class PostsService {
  
  constructor (private http: HttpClient) {}

  createAndStorePost(title: string, content: string){
    const postData: Post = {title: title, content: content};
    this.http.post<{name: string}>('https://my-url-to-api/posts.json', postData)
      .subscribe(responseData => {
        console.log(responseData);
      })
  }

  fetchPosts() {
    this.http.get('https://my-url-to-api/posts.json')
      // Esto siguiente es porque FireStore no devuelve un array, sino un objeto con todos los datos
	 // de nuestra  colección como propiedades de ese objeto
	  .pipe(
	    map(responseData => {
	     const postsArray: Post[] = [];
	     for (const key in responseData) {
	       postsArray.push({...responseData[key], id: key})
	     }
	     return postsArray;
	    })
	  )
	  .subscribe(responseData => {
  	     
		//Estrategia para guardar los datos y que estén disponibles en el componente		
		  return responseData;
       })
  }

}
```

```typescript
//En mi componente
import { PostService } from './post.service'

...

export class ComponentePepito {

  constructors(private postService: PostService){}

  onCreatePost(postData: Post){
    this.postsService.createAndStorePost(postData.title, postData.content)
  }

}



```


### Posibles estrategias para guardar las respuestas de peticiones en los servicios.

##### Devolviendo los resultados

Simplemente usando

```typescript
return this.http... //En el servicio


// En el componente:
onCreatePost(postData: Post){
    this.postsService.createAndStorePost(postData.title, postData.content)
      .subscribe((response) => {
       this.variableLocal = response
      })
  }


```
##### Usando `Subjects`

Esta estrategia nos permite que se actualicen los valores entre los diferentes componentes. Guardaríamos el resultado de la petición en el servicio como un `Subject` que al tener un cambio propagaría eventos que modificarían los valores en los diferentes componentes que a su vez estén suscritos a ese `Subject`

## Manejo de errores en las peticiones

En las peticiones, cuando nos subscribimos, el primer argumento es el dato de respuesta, mientras que un segundo argumento correspondería a un objeto error que podemos usar para gestionar los errores en nuestra aplicación.

```typescript

onCreatePost(postData: Post){
  this.postsService.createAndStorePost(postData.title, postData.content)
    .subscribe((response) => {
      this.variableLocal = response
    }, (error) => {
      console.log(error.message);
    }
  )
}

```


##### Usando `Subjects` para gestionar errores

La estrategia de utilizar **`Subjects`** (Ahora explico que son) para gestionar errores es indicada cuando NO estamos devolviendo la petición en el servicio. Entonces podemos guardar el resultado de nuestra petición en un `Subject`

```typescript
//Servicio


@Injectable({ providedIn: 'root' })
export class PostService {

  error = new Subject<string>()

  constructor(private http: HttpClient) {}


  createAndStorePost(title: string, content: string) {
    const postData: Post = {title: title, content: content};

	this.http.post<{name: string}>('https://ng-url-to-backend/post.json', postData)
	  .subscribe(response => {
	    console.log(response);
	  }).error(error => {
	    this.error.next(error.message)
	  })
  }
}

```



```typescript
// COMPONENTE
import { Component, OnDestroy, OnInit } from '@angular/core';
import { Subject } from 'rxjs';
import { Subscription } from 'rxjs';



exports class ComponenteEjemplo implements OnInit, OnDestroy

error: boolean = false;
private errorSubscription!: Subscription;


ngOnInit(){
  this.errorSubscription = this.postService.error.subscribe(errorMessage => {
     this.error = true;
     console.log(erroMesssage);
  })
}


ngOnDestroy(){

this.errorSubscription.unsubscribe();

} 
```


### Uso del pipe `catchError` para gestionar un error.


Cuando usamos `Observables` ya habíamos visto la posibilidad de usar operadores para transformar el resultado de la petición.
Hay un operador de `Pipe` específico para errores, llamado `catchError` que nos permitiría interceptar un error y realizar las operaciones necesarias (p.e. escribir en un log) y después podemos devolver ese error usando el método `throwError` (Que no es un operador de pipe de '`rxjs/operators`', sino un método de '`rxjs`')


```typescript
import { Subject, throwError } from 'rxjs';
import { map, catchError } from 'rxjs/operators'

...
...
.pipe(
  map((responseData)=>{
    // Do something with data
    return changedData
  }),
  catchError((errorResponse)=>{
    //Do something with error data
	// Throw error
	return throwError(errorResponse)
  })
)
```



## Configuración de `Headers` en las peticiones

Las `Headers` o *cabeceras* de las peticiones son una de las dos partes que contiene una petición HTTP: Cabeceras y cuerpo (`Header` y `Body`). En el cuerpo encontramos los datos que estamos transmitiendo en nuestra petición (generalmente algún objeto JSON), mientras que todo lo demás se encuentra en la cabecera:

* Dirección de la petición
* Verbo de la petición
* Autorización del emisor de la petición
* etc


En cualquier petición usando Angular tenemos la dirección y el verbo de la petición, pero si queremos incluir cualquier otro dato de nuestra cabecera debemos usar una configuración especial de nuestra petición.

En nuestro `HttpClient` hemos visto que teníamos disponibles los métodos `.get(), post(), ...` para enviar las peticiones, y que el primer argumento era una cadena de texto que contenía la URL de la petición.

Pues bien, la configuración de las opciones será el último argumento de ese método usado (el segundo argumento en un GET, el tercero en un POST, por ejemplo), y le pasaremos un objeto con pares nombre-valor. Uno de esos pares será "`Headers` que será un objeto `HttpHeaders` que contendrá los pares nombre-valor de nuestra cabecera

```typescript

import { HttpClient, HttpHeaders} from '@angular/common/http';


@Injectable({ providedIn: 'root'})
export class PostsService {

  constructor(private http: HttpClient){}

  postSomething(title: string, content: string) {
    const postData = {title: title, content: content};
    http.post(
      'https://my-url-to-api', 
      postData, 
      {headers: new HttpHeaders({
        'Custom-header': 'Hello',
      })}
    )
  }
}
```

Podemos además agregar parámetros de petición a nuestra URL usando el mismo último objeto bajo la propiedad `params`

```typescript
postSomething(title: string, content: string) {
  const postData = {title: title, content: content};
  http.post(
    'https://my-url-to-api', 
    postData, 
    {
    headers: new HttpHeaders({'Custom-header': 'Hello',})
    params: new HttpParams().set('print', 'pretty')
    
    }
  )
}
```

Si quisiéramos concatenar múltiples parámetros de búsqueda la forma es un poco más enrevesada. Debemos crear previamente el objeto `HttpParams` y usar su método `.append('prop', 'valor')`, para ir agregando parámetros de dos en dos:

```typescript

import { HttpParams, HttpClient, HttpHeaders } from '@angular/common/http'
...

postSomething(title: string, content: string) {
  const postData = {title: title, content: content};

  let searchParams = new HttpParms();
  searchParams = searchParams.append('print', 'pretty');
  searchParams = searchParams.append('even', 'prettier');
  //equivale a https://my-url-to-api?print=pretty&even=prettier

  http.post(
    'https://my-url-to-api', 
    postData, 
    {
    headers: new HttpHeaders({'Custom-header': 'Hello',})
    params: searchParams,
    }
  )
}

```

Otra opción más que podemos mandar en nuestra petición es la propiedad `observe` que nos permite definir el tipo de respuesta a la que nos estamos subscribiendo.

Por defecto nos subscribimos a un `observe: 'body'` donde el dato que recibimos en nuestro primer argumento del `callback` de `.subscribe((data)=>{})` (el `data`) es el cuerpo de la respuesta. Pero si queremos recibir por ejemplo también la cabecera de la respuesta podemos modificar el `observe` a `observe: 'response'` donde recibiremos toda la respuesta, es decir, cuerpo y cabecera.

```typescript

http.post(
    'https://my-url-to-api', 
    postData, 
    {
      headers: new HttpHeaders({'Custom-header': 'Hello',})
      params: searchParams,
      observe: 'response'
    }
  )

```


Ahora la respuesta tendrá el formato de un objeto con dos propiedades: `body` y `header`:

```JSON
{
  "header": {},
  "body": {},
  "status": 200,
  "url":  "https://my-url-api-answering",
  ...
}
```

También podemos usar `observe` con el evento de nuestra petición, que nos puede informar de eventos como el progreso de una descarga o subida de archivos, si se ha mandado una petición, si se ha recibido la petición, etc.

```typescript

import { HttpEventType, HttpClient} from '@angular/common/http';


deletePosts() {

  return http
    .delete('https://url-to-api', {observe: 'event'})
    .pipe(
      tap( (event) => {
        console.log(event);
	    if (event.type === 0) {
          console.log('Petición enviada');
        }
        if (event.type === HttpEventType.Response) {
          console.log('Respuesta recibida');
          console.log(event.body);
        }
      }
    ))
}
```


### Uso de `Interceptors` para configurar cabeceras de autorización


Para usarlo primero vamos a crear un servicio llamado `auth-interceptor`

```typescript

import { HttpInterceptor, HttpRequest, HttpHandler } from '@angular/common/http'

export class AuthInterceptorService implements HttpInterceptor {

  intercept( req: HttpRequest<any>, next: HttpHandler){
    console.log('Request on its way!');
    return next.handle(req);  
  }
}
```


Ahora lo proveemos en nuestro array de `providers`, de una forma muy concreta para indicarle a Angular que se trata de un servicio interceptador: 

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpClientModule. HTTP_INTERCEPTORS } from '@angular/common/http';

import { AppComponent } from './app.component';
import { AuthInterceptorService } from './auth-interceptor.service';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, FormsModule, HttpClientModule],
  providers: [{provide: HTTP_INTERCEPTORS, useClass: AutInterceptorService, multi: true}],
  bootStrap: [AppComponent]
})
export class AppModule {}
```

Esto hará que todas las peticiones que se hagan ejecuten el método `intercept` de nuestro servicio interceptador, por lo que podríamos usarlo para agregar opciones de *Autenticación* a nuestras peticiones.

Dentro del interceptador podemos modificar la petición, pero no directamente, porque el objeto `req` que le hemos pasado es inmutable. Pero si que podemos crear uno nuevo a partir de el original, modificar ese nuevo y reemplazarlo por el original al enviarlo.

```typescript

export class AuthInterceptorService implements HttpInterceptor {

  intercept( req: HttpRequest<any>, next: HttpHandler){
    console.log('Request on its way!');
    const modifiedRequest = req.clone({headers: req.headers.append('Auth', 'something')})
    //return next.handle(req);
    return next.handle(modifiedRequest);
  }
}
```

Además, podemos usar el interceptor de la petición para tener un interceptor para la respuesta de esa petición mediante el uso de un método `pipe` de `next` ya que tendrá como respuesta un observable. Un detalle importante a la hora de gestionar la respuesta recibida en el interceptor es que la respuesta es de tipo **`event`**

```typescript
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEventType } from '@angular/common/http';
import { tap } from 'rxjs/operators';

export class AuthInterceptorService implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler){
    console.log('Request is on its way!');
    const modifiedRequest = req.clone({headers: req.headers.append('Auth', 'something')});
	return next
	         .handle(modifiedRequest)
	         .pipe(tap( (event) => {
	           if (event.type === HttpEventType.Response){
	             console.log(event);
	             console.log('Response arrived');
	           }
	         }));
  }
}

```

Si quisiéramos usar mas *Interceptadores* lo configuraríamos igual, pero en el `AppModule` lo agregaríamos así

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpClientModule. HTTP_INTERCEPTORS } from '@angular/common/http';

import { AppComponent } from './app.component';
import { AuthInterceptorService } from './auth-interceptor.service';
import { LoggingInterceptorService } from './log-interceptor.service';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, FormsModule, HttpClientModule],
  providers: [
    {provide: HTTP_INTERCEPTORS, useClass: AutInterceptorService, multi: true},
    {provide: HTTP_INTERCEPTORS, useClass: LoggingInterceptorService, multi: true},
  ],
  bootStrap: [AppComponent]
})
export class AppModule {}

```

Y en este caso primero se ejecuta el interceptor `AutInterceptorService` y después el `LoggingInterceptorService`