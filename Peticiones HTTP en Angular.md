
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