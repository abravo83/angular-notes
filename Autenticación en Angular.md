
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

    // Podemos comprobar si el token ya ha caducado
    if ( !this._tokenExpirationDate || new Date() > this._tokenExpirationDate ) {
      return null;
    }
    
    return this._token
  }



}
```


Luego, en un servicio podemos guardar el Usuario como un `Subject,` al que nos podemos subscribir desde otros componentes

```typescript
import { Injectable } from '@angular/core';
import { throwError, BehaviorSubject } from 'rxjs';
import { catchError, tap } from 'rxjs/operators';

import { User } from './user.model';

export interface AuthResponseData {
  kind: string;
  idToken: string;
  email: string;
  refreshToken: string;
  expiresIn: string;
  localId: string;
  registered: boolean;
}


@Injectable({
  providedIn: 'root'
})
export class AuthService {

  // Crearemos un usuario cuando usemos next al registrarnos y al hacer login
  user = new BehaviorSubject<User>(null);
  //token: string = null;
  
  signUp(email: string, password: string){
    return this.http.post<AuthResponseData>(
        'https://identitytoolkit.googleapis.com/v1/accounts:signUp?key=tHISiSyOURkEY-Q',
        { 
          email: email,
          password: password,
          returnSecureToken: true
        }
        ).pipe(
          catchError(this.handleError)
          tap((resData: any) => {
            this.handleAuthentication(resData.email, resData.localId, resData.idToken, +resData.expiresIn);
          } 
       )
   }
   
  logIn(email: string, password: string): Observable<any> {
     const body = {
       email: email,
       password: password,
       returnSecureToken: true
     }

     return this.http.post('https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=ThisIsYOURaPIkEY', body
     ).pipe(
      tap((resData: any) => {
        this.handleAuthentication(resData.email, resData.localId, resData.idToken, +resData.expiresIn);
      }
     ),
     catchError(this.handleError))
   }

// Para no repetir el código en login y signup creamos un método aparte que realice la inicialización del usuario una vez autenticado.
  private handleAuthentication(email: string, localId: string, token: string, expiresIn: number) {
    const expirationDate = new Date(new Date().getTime() + expiresIn * 1000);
    const user = new User(email, localId, token, expirationDate);
	// Aquí asignamos el valor al Subject
    this.user.next(user);
  }
  
}

```


## Agregando la Cabecera de Autorización

Llegado a este punto ya hemos conseguido dos cosas: Registrar a un usuario nuevo y `loguear `a un usuario existente. Pero, como hemos dicho, la verdadera funcionalidad de estar autenticado es la restricción de la respuesta del `backend`. Para ello nuestras subsecuentes peticiones deben y con el encabezado de la autenticación para que sea nuestro `backend` el que valide o no si estamos autorizados a recibir la respuesta de la petición que le estamos realizando.


Para ello, ya que vamos a agregar dicha cabecera a todas nuestras peticiones, lo ideal es usar un Interceptador con los datos de identificación de nuestro token de autorización que hemos recibido al `loguearnos `o registrarnos, pero primero vamos a ver el caso en el que no usamos dicho interceptador y agregamos  la cabecera a nuestra petición de manera individual (Es decir, lo agregamos a una petición en concreto, ya que el interceptador haría que se agregase a todas nuestras peticiones sin necesidad de modificar nada de nuestras peticiones.);

Petición sin cabecera de autorización

```typescript
storeRecipes() {
  const recipes = this.recipeService.getRecipes();
  this.http.put('https://route.to.api.com/recipes', recipes).subscribe(response => console.log(response));
}
```


Misma petición con cabecera de autorización. El token lo tenemos almacenado en caché como propiedad de nuestro objeto user, que a su vez lo cogerá del **`BehaviorSubject`** `user` del servicio `AuthService`


```typescript
import { HttpHeaders, HttpClient } from '@angular/common/http';

import { recipeService } from './recipe.service';
import { authService } from './auth.service';

export class DataStorage {
  
  constructor(private http: HttpClient, private recipeService, private authService: AuthService) {}
  
  storeRecipes() {
	// Al usar el método pipe con el operador take(1) lo que hacemos es una subscripción única donde recibimos el dato una vez y cancelamos la subscripción.
	// El siguiente operador: exhaustMap hace que se combinen los dos observables, el del Behavior Subject y el de la petición, en un único observable
	
    this.authService.user.pipe(take(1), exhaustMap).subscribe(user => {
    
       const recipes = this.recipeService.getRecipes();
	   //Ya habíamos creado un getter para token que accede a _token 
       const authToken = this.user.token;
       
       //Creamos la cabecera de la petición
       const headers = new HttpHeaders({
       'Content-Type': 'application/json',
       'Authorization':: 'Bearer ' + authToken
       });
       
       // Agregamos la cabecera a la petición
       this.http.put('https://route.to.api.com/recipes', recipes, { headers: headers }).subscribe(response => console.log(response));
    
    } )
  }
}
```


## Usando Interceptores para agregar la cabecera

Para  manipular todas las peticiones salientes vamos a crear un nuevo archivo que va a contener nuestra lógica del interceptor: `auth-interceptor.service.ts`

```typescript
import { Injectable } from '@angular/core';
import { Interceptor, HttpHeader, HttpClient } from '@angular/common/http';
import { take, exhaustMap}

import { AuthService } from './auth.service';


// No le agregamos el 'providedIn': 'root' porque vamos a usar otra forma de provisionarlo.
@Injectable()
export class AuthInterceptorService implements HttpInterceptor {


  // Usamos el constructor para injectar el servicio AuthService

  constructor(private authService: AuthService){}

  intercept(req: HttpRequest<any>, next: HttpHandler) {
    // Tenemos que obtener el Subject usuario para lo que tenemos que subscribirnos a él.
    // El problema es que aparte tenemos otra subscripción por lo que debemos juntar ambas
    // Lo que vamos a hace usando el pipe take(1) y el pipe exahustMap de rxjs
    return this.authService.user.pipe(
      take(1),
      exhaustMap(user => {
        
        // Creamos la cabecera
	    const headers = new HttpHeaders({
	     'Content-Type': 'application/json',
         'Authorization':: 'Bearer ' + user.token
        });
        
        // Clonamos la petición y la modificamos.
	    const modifiedRequest = req.clone({headers: headers})
        return next.handle(modifiedRequest);
      })
    )
  }
}
```

Para que el interceptor funcione interceptando las peticiones debemos incluirlo como `provider` en `AppModule`

```typescript
//app.module

import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';

import { AuthInterceptorService } from './auth-interceptor.service';

@NgModule({
  declarations: [
    //...
  ],
  providers: [
  // ...,
  {provide: HTTP_INTERCEPTORS, useClass: AuthInterceptorService, multi: true} //multi nos permite usar más de un interceptor.
  ]
})

```



### Gestión del Interceptor cuando no nos hemos logueado ni registrado


El problema es que debemos  de poder de alguna forma sacar a nuestras peticiones de registro y `logueo` del interceptador ya que no puedes usar el objeto usuario porque no se define hasta que se ejecutan esas peticiones, por lo que estas peticiones fallan al ser usadas por el interceptor.

Para ello, modificamos el interceptor incluyendo el condicional de si no tenemos usuario:

```typescript
// auth-interceptor.service.ts

  //...

  intercept(req: HttpRequest<any>, next: HttpHandler) {
    // Tenemos que obtener el Subject usuario para lo que tenemos que subscribirnos a él.
    // El problema es que aparte tenemos otra subscripción por lo que debemos juntar ambas
    // Lo que vamos a hace usando el pipe take(1) y el pipe exahustMap de rxjs
    return this.authService.user.pipe(
      take(1),
      exhaustMap(user => {

		// AGREGAMOS EL CONDICIONAL SI NO HAY USUARIO PARA PODER MANDAR LA PETICIÓN CUANDO NO HAY USUARIO
		// Si no hay usuario devolvemos la petición original
		if (!user) {
          return next.handle(req);
		}
        
        
        // Creamos la cabecera
	    const headers = new HttpHeaders({
	     'Content-Type': 'application/json',
         'Authorization':: 'Bearer ' + user.token
        });
        
        // Clonamos la petición y la modificamos.
	    const modifiedRequest = req.clone({headers: headers})
        return next.handle(modifiedRequest);
      })
    )
  }
```



## Haciendo `Logout`

En estos ejemplos, los inicios de sesión se hacen cuando se obtiene y se inicializa el objeto `user`, por lo tanto, para hacer el `logout` sólo debemos hacer que `user=null` en el servicio. Además, si estamos en alguna parte restringida de la aplicación, o igualmente si no, debemos hacer que la aplicación navegue a la parte inicial importando `Router` en el servicio y haciendo que en el mismo método que se ejecuta al hacer `logout` hagamos `this.router.navigate('/inicio')` 


## Guardar el Token en el navegador para hacer `Auto-Login`


### Guardando el Token

En el navegador podemos utilizar el almacenamiento local o las `cookies` (Alguna de estas dos utilidades) para guardar información en el navegador que no queremos que desaparezca cuando recargamos la página.

Podemos utilizar esto para guardar la información de nuestro `token` de autorización y así poder gestionar un `auto-logueo` para que no perdamos la información de este token al recargar la página.

En la función en la que gestionábamos la respuesta de ambas peticiones de registro y logueo para fijar el valor del **Subject** `user` vamos a añadir métodos para guardar el token en el navegador:

```typescript
//Hello
private handleAuthentication (
  email: string,
  userId: string,
  token: string,
  expiresIn: number
){
  const expirationDate = new Date(new Date().getTime() + expiresIn * 1000);
  const user = new User(email, userId, token, expirationDate);
  this.user.next(user);
  //Aquí guardamos el token
  localStorage.setItem('authToken', JSON.stringify(user.token));
  localStorage.setItem('expirationDate', JSON.stringify(user.expirationDate));
  
}
```

### Reiniciando sesión al recargar la aplicación


Vamos a crear un método para que haga un inicio de sesión en nuestro servicio de autenticación `AuthService`

```typescript

autoLogin(){
  const authToken = JSON.parse(localStorage.getItem('authToken'));
  const expirationDate = JSON.parse(localStorage.getItem('expirationDate'))
  
  // Si no hay ningún token en el navegador o está caducado no intentamos el inicio de sesión
  if (!authToken || expirationDate < new Date() ) {
    return;
  }
  // Debemos de tener algún endpoint que nos permita a partir de un token de autorización recuperar el usuario
  // Si lo hay y nos devuelve al usuario iniciamos sesión.
  this.http.get('http://api.endpoint/userfromtoken', authToken).subscribe(user => {
    if (user) {
       this.handleAuthentication(user.email, user.userId, authToken, expirationDate) 
    }
  })
}
```


#### ¿ Dónde podemos llamar a este método ?

Para que el método `autoLogin` se ejecute al reiniciar o iniciar la aplicación podemos llamar a su ejecución desde nuestro componente raíz `app.component.ts`  usando el ciclo `NgOnInit` para llamar a su ejecución.


## Mejorando nuestro `LogOut`

Al hacer `Log-out` debemos también preocuparnos de eliminar nuestro token y otros datos, para que no funcione el `auto-login` cada vez que recargamos.

Entonces, en nuestro método `logOut` vamos a agregar funcionalidad que elimine el `localStorage` de nuestro token y fecha de caducidad.

```typescript
logout(){
  this.user.next(null);
  this.router.navigate('/auth');
  localStorage.removeItem('token');
  localStorage.removeItem('expirationDate');
}
```

También podemos crear un método que se ejecute con un `TimeOut` y que fuerce un `autoLogOut` cuando el momento de caducidad llegue mientras estamos usando la aplicación. Aunque esto puede ser un fastidio para la experiencia de usuario.

## Protegiendo las rutas que requieren autenticación


Puede que en nuestra aplicación hayamos "ocultado" enlaces a las secciones que no queremos mostrar cuando no estamos autenticados, pero eso no hace que, si tienen ruta, podamos acceder a esas secciones usando directamente la ruta en la barra de direcciones. Por ello necesitamos proteger dichas rutas con un `Guard` que compruebe si estamos o no autenticados.

Podemos crear un archivo tipo `guard`, que es un tipo especial de servicio:

```typescript
import { Router, UrlTree, } from '@angular/router';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { map, tap, take } from 'rxjs/operators';

import { AuthService } from './auth.service';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {

  constructor(private authService: AuthService, private router: Router){}


  canActivate(route: ActivatedRouteSnapShot, router: RouterStateSnapshot): | boolean | UrlTree | Promise<boolean | UrlTree> | Observable<boolean | UrlTree> {
    // Realizamos una petición para ver si tenemos login al contar con Subject user
    return this.authService.user.pipe(
      // Para que la subscripción se cancele tras la primera respuesta
      take(1),
      map(
        user => {
          const isAuth = !!user;
          if (isAuth) {
            return true;
          }
	      //Al devolver una ruta estamos sustituyendo lo que hace el router de forma nativa
	      return this.router.createUrlTree(['/auth'])
        }
      ),
    // Antiguamente no podíamos devolver un UrlTree, entonces lo que se hacía es que el map anterior devolvía no solo true sino también false cuando no se estaba autenticado, entonces podíamos usar tap para evaluar esa respuesta de map y hacer que el navegador vaya a la página de registro e inicio de sesión.
    //tap ( isAuth => {
      //if (!isAuth) {
       //return this.router.navigate(['/auth'])
      //}
    //)
    )
  }  
}

```

Y luego en nuestro archivo de rutas protegemos la ruta con el `Guard` usando el método `canActivate`

```typescript
\\ app-routing.module.ts
import { AuthGuard } from './auth/auth.guard'

const appRoutes: Routes = [

  { path: '', redirectTo: '/recipes', pathMatch: 'full' },
  { path: 'recipes', component: RecipesComponent, canActivate: [AuthGuard],  children: [
    { path: '', component: RecipeStartComponent },
    { path: 'new', component: RecipeEditComponent },
    { path: ':id', component: RecipeDetailComponent, resolve: [RecipesResolverService] },
    { path: ':id/edit', component: RecipeEditComponent, resolve: [RecipesResolverService] },
  ] },

  { path: 'shopping-list', component: ShoppingListComponent },
  { path: 'auth', component: AuthComponent },
];

```
