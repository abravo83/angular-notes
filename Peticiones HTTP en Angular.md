
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
