
Angular nos permite utilizar `Routing` para que, aunque se use una aplicación SPA, parezca que estamos navegando entre diferentes páginas.

Para utilizar `routes` debemos de incluir una constante que llamaremos `appRoutes` en nuestro archivo *app.modules.ts* por encima del decorador y va a contener un array the tipo `Routes` (importado de '@angular/router')

```typescript
import { Routes } from '@angular/router'; 

const appRoutes: Routes = [
  { path: 'users', component: UsersComponent }
]

```

Pero aún debemos decirle a Angular que use esta constante como rutas.

Para esto necesitamos incluir el módulo `RouterModule` en los imports del decorador @NgModule, y dicho módulo a su vez es importado también desde `@angular&/router`. En la propia declaración de importación usamos el método .forRoot(*nombreDeNuestraConstanteRoutes*) para indicarle a Angular donde están las rutas.

```typescript
import { Routes, RouterModule } from '@angular/router';

const appRoutes: Routes = [
  {path: '', component: HomeComponent},
  {path: 'users', component: UsersComponent},
  {path: 'servers', component: ServersComponent}
];

@NgModule({
  declarations: [
  ...,
  ...,
  ],
  imports: [
    ...,
    ...,
    RouterModule.forRoot(appRoutes)
  ],
})
```


La configuración está terminada, pero aún debemos indicarle a Angular donde se cargan los componentes. Es decir, en que parte de nuestra aplicación, o en que parte de `app.component.ts` o el archivo que sea, se van a cargar los componentes seleccionados para cargar en cada ruta.

Para esto usamos una directiva especial llamada: `<router-outlet></router-outlet>` que nos permite indicarle a Angular donde exactamente se van a cargar los componentes del router.


```html
<!-- Cambiamos esto -->
<div>
  <app-home></app-home>
</div>
<div>
  <app-servers></app-servers>
</div>
<div>
  <app-users></app-users>
</div>

<!-- Por esto -->
<div>
  <router-outlet></router-outlet>
</div>
```

Con eso ya podríamos acceder directamente con la URL a ver el contenido de cualquier componente. Sin embargo, si queremos ver dichos componentes al hacer *click* en algún enlace no usaremos la ruta de la URL para mostrar dicho contenido, ya que si lo hacemos la aplicación se estaría recargando. Para evitar este comportamiento debemos usar la directiva `routerLink=""` dentro de las etiquetas `<a>` que queramos sustituir su `href` para evitar la recarga de la página.

```html
<ul>
  <li role="presentation" class="active"><a routerLink="/">Home</a></li>
  <li role="presentation"><a routerLink="/servers">Servers</a></li>
  <li role="presentation"><a [routerLink]="['/users']">Users</a></li>
</ul>
```

## Agregar clases a la ruta activa

Tenemos una directiva propia de Angular que nos permite agregar una clase de forma condicional a elementos html cuando la ruta activa sea la que indica el elemento. Se puede usar para los elementos que contienen la directiva `routerLink` y para sus contenedores, y se especifica la clase usando la directiva: `routerLinkActive="NombreDeLaClase"`

```html
<ul>
  <li role="presentation" routerLinkActive="active"><a routerLink="/">Home</a></li>
  <li role="presentation" routerLinkActive="active"><a routerLink="/servers">Servers</a></li>
  <li role="presentation" routerLinkActive="active"><a [routerLink]="['/users']">Users</a></li>
</ul>
```

El problema de la anterior configuración es que `Home` siempre tiene la clase "active" ya que el elemento "/" forma parte de **todas** las rutas. Para evitar esto, podemos usar la directiva `routerLinkActiveOptions` usando `[]` para pasarle un objeto para configurarlo donde le decimos que la ruta debe ser exacta y no parte de otra.

```html
<ul>
  <li role="presentation" 
      routerLinkActive="active"
      [routerLinkActiveOptions]="{exact: true}"
	  >
    <a routerLink="/">Home</a>
  </li>
  <li role="presentation" routerLinkActive="active">
    <a routerLink="/servers">Servers</a>
  </li>
  <li role="presentation" routerLinkActive="active">
    <a [routerLink]="['/users']">Users</a>
  </li>
</ul>
```

## Como dirigir o activar un enlace en la lógica de nuestra aplicación


Para poder hacer que se cargue un componente desde la lógica debemos acceder de alguna forma a nuestro `router` y para eso inyectamos en el componente donde vamos a realizar la lógica el objeto Router`:

```typescript
import { Router } from '@angular/core';

 ...
 constructor(private router: Router) {}


 onLoadServers(){
   //COmplex calculations
   this.router.navigate(['/servers']);
 }
```

Este método es diferente respecto del uso de `routerLink` ya que no tiene referencia relativa de la ruta en la que está, por lo que las rutas relativas se tomarán como absolutas.
Para que funcione como una ruta relativa debemos agregarle la información de a qué es relativa dicha ruta.

Además, podemos obtener la ruta actual en el navegador inyectando el objeto ActivatedRoute en el componente:

```typescript
import { Router, ActivatedRoute } from '@angular/router';
...
@Component({
...
})

...
...

constructor(private router: Router, private route: ActivatedRoute) {}

onLoadServer(){
  //Complex calculations
  this.router.navigate(['servers'], {relativeTo: this.route})
}
```

## Usar parámetros en nuestras rutas

Podemos usar parámetros en nuestras rutas usando `:` en el objeto de definición de la ruta seguido del nombre de la variable que queramos usar de manera dinámica.

```typescript
const appRoutes: Routes = [
  {path: 'users/:id/:name', component: UserComponent},
]
```

### Acceder al parámetro de la ruta dentro de nuestro componente cargado

Si queremos usar dicho parámetro tenemos que usar el objeto `ActivatedRoute`, por lo que lo tenemos que inyectar en nuestro componente:

```typescript
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-user',
  templateUrl: './user.component.html',
  styleUrls: ['./user.componet.css']
})
export class UserComponent implements OnInit {

  user: {id: number, name: string};
  
  constructor(private route: ActivatedRoute){}

  ngOnInit() {
    this.user = {
      id = this.route.snapshot.params['id'],
      name = this.route.snapshot.params['name']
    } 
  }
}
```


#### Acceder al parámetro de forma reactiva

El acceso anterior nos permitía coger los parámetros de la ruta cuando se carga el componente, pero si queremos volver a recoger los parámetros de nuestra ruta aunque el componente siga siendo el que está cargado.

Para realizar esto, además de usar `this.route.snapshot.params['nombreDelParametro']` para la inicialización, vamos a usar `this.route.params` que es un <i><b>Observable</b></i> que nos ayuda a trabajar con tareas asíncronas, que usa el método **.subscribe()** que se ejecutará cuando se detecta el cambio en este *Observable* 
```typescript
ngOnInit() {
  this.route.params
    .subscribe({
      next: (data: Params) => {
        this.user.id = data['id'];
        this.user.name = data['name'];
      },
      error: () => { console.log('Error') }
    });
}
```

#### Acceder a otros tipos de datos usados en nuestra ruta

Nuestra ruta puede contener diferentes tipos de datos agregados a nuestros parámetros tales como `/users/10/Anna?mode=editing#loading`

Para poder definir estos parámetros en nuestro `[routerLink]` debemos usar  propiedades extra en nuestro elemento HTML `queryParams`  que nos va a permitir agregar los elementos `allowEdit=1` de parámetros y `fragment` que nos va a permitir usar los elementos `#loading` de referencias locales

```html
<a
   [routerLink]="['servers', 5, 'edit']"
   [queryParams]="{allowEdit: '1'}"
   [fragment]="'loading'"
  >
</a>
```


Para poder acceder a estos parámetros tenemos dos posibilidades

1. Acceder al `snapshot` de *ActivatedRoute*
2. Subscribirse al valor del observable .queryParams en *ActivatedRoute*

```typescript
contructor(private route: ActivatedRoute){}

ngOnInit(){

  // Primer método
  console.log(this.route.snapshot.queryParams);
  console.log(this.route.snapshot.fragment);

  // Segundo método
  this.route.queryParams.subscribe({
    next: (data) => { console.log(data); }
  })
  this.route.fragment.subscribe({
    next: (data) => { console.log(data); }
  })

}
```


#### Mantener parámetros cuando se visitan enlaces posteriores

SI queremos preservar los parámetros que tenemos, podemos usar en el objeto de propiedades de `router.navigate` una propiedad llamada `queryParamsHandling: 'preserve'` que hará que se mantengan nuestros parámetros en vez de deshacerse de ellos, que es el comportamiento normal de `router.navigate`

```typescript
onEdit() {
  this.router.navigate(['edit'], {relativeTo: this.route, queryParamsHandling: 'preserve'})
}
```


## Anidar Routers, o Children Routes

Para poder anidar routes, vamos a usar las rutas que incluyen parte de un componente y las vamos a incluir dentro de la definición de esa ruta en nuestro app-module, dentro de la propiedad `children`, que contiene una array con todas las rutas anidadas 

```typescript
// Cambiar esto
const appRoutes: Routes = [
  {path: '', component: HomeComponent},
  {path: 'users', component: UsersComponent},
  {path: 'users/:id/:name', component: UserComponent},
  {path: 'servers', component: ServersComponent},
  {path: 'servers/:id/edit', component: EditServerComponent},
  {path: 'servers/:id', component: ServerComponent},
  
];
// por esto
const appRoutes: Routes = [
  {path: '', component: HomeComponent},
  {path: 'users', component: UsersComponent, children: [
    {path: ':id/:name', component: UserComponent}, // Ya no lleva /users
  ]},
  {path: 'servers', component: ServersComponent, children: [
    {path: ':id', component: ServerComponent}, // Ya no lleva /servers
    {path: ':id/edit', component: EditServerComponent}, //Ya no lleva /servers
  ]},
];
```

Luego, dentro del componente, añadimos el elemento `router-outlet` donde queremos que se rendericen nuestros `Route Children`

```html
<div class="col-xs-12 col-sm-4">
    <!-- Aquí se ha cambiado una etiqueta que definía un componente por una etiqueta de ruta, ya que antes esa ruta cargaba el componente a pantalla completa en vez de como componente anidado. Ahora como ruta anidada carga correctamente -->
    <!-- <app-user></app-user> -->
    <router-outlet></router-outlet>
  </div>
```

## Usar redirecciones

Para hacer que una ruta redirija a otra debemos configurar esa ruta de manera distinta en nuestro app-module. En vez de que el `path` cargue un componente tendrá la propiedad `redirectTo: '/ruta-a-redirigir'`

```typescript
import { Routes, RouterModule } from '@angular/router';

import { PageNotFoundComponent } from './page-not-found/page-not-found.component';

const appRoutes: Routes = [
  {path: 'not-found', component: PageNotFoundComponent},
  {path: 'something', redirectTo: '/not-found'},
]
```

## Usar comodines

Para poder usar comodines que cubran cualquier ruta no contemplada entre nuestras rutas podemos incluir la ruta `**` que funciona como comodín, pero **OJO con el ORDEN en el que se coloca**  ya que las rutas se interpretan de arriba a abajo, y si la colocamos la primera hará que no funcione ninguna de nuestras rutas ya que todas serán *atrapadas* por el comodín.

```typescript
const appRouter: Routes = [
  {path: 'not-found', component: PageNotFoundComponent},
  {path: '**', redirectTo: '/not-found'}
]
```

Para aumentar la seguridad de que la ruta a la que redirigimos no es un resultado parcial de otra, por ejemplo si tuviésemos un `redirectTo` en nuestra ruta `{path: '', redirectTo: '/index'}` , resulta que **TODAS** las rutas tienen el caracter `''` así que de nuevo todas las rutas se dirigirían hacia esa. Para evitarlo podemos especificar en el funcionamiento del `redirecTo` que la concordancia sea exacta agregando `pathMatch: 'full'` 

```typescript
const appRouter: Routes = [
  {path: '', redirectTo: '/index', pathMatch: 'full'},
  {path: 'index', component: HomeComponent},
  {path: 'not-found', component: PageNotFound},
  {path: '**', redirectTo: '/not-found'},
]
```


## Usar un archivo externo para controlar nuestras rutas

Si queremos evitar obtener una archivo de rutas bastante grande debemos usar un archivo externo para organizar nuestras rutas.

Para eso crearemos un nuevo archivo tipo module llamado `app-routing.module.ts` junto a nuestro `app.module.ts` y en el vamos a:
1. Crear la clase exportable `AppRoutingModule`
2. Agregarle el decorador `@NgModule`, con su importación desde '@angular/core'
3. Trasladar nuestro array de rutas por delante del decorador
4. Realizar las importaciones de los componentes y del tipo `Routes` (sin mover las de los componentes de `app.module`)
5. Trasladar del array imports de `app.module` el módulo `RouterModule.forRoot(appRoutes)` al array imports de `app-routing.module` 
6. Incluir en el array exports de `app-routing.module` el módulo `RouterModule` **sin** `.forRoot(appRoutes)`   
7.  Importar (incluyéndolo en el array imports) nuestro módulo en `app.module`

## Route Guards

Son una función que se ejecuta antes de que se cargue el componente. Nos pueden servir para proteger una ruta o para otra función o comprobación que queramos realizar antes de que se cargue el componente. O incluso para proteger que podamos salir de una ruta.

### canActivate y canActivateChild

Para implementar una protección de acceso a una ruta vamos a crear un servicio que nos va a servir de `middleware`, al que vamos a llamar por convención `auth-guard.service.ts` y al que vamos a crear dos métodos: `canActivate` y `canActivateChild`. El primero nos sirve para controlar las rutas padre  o raíz mientras que el segundo nos sirve para controlar las rutas hijas de una ruta padre.

Aquí usaremos una función que se comunicará con un servicio que, en el ejemplo nos va a decir si un usuario está autenticado o no:

```typescript
import { Injectable } from "@angular/core";
import {
        ActivatedRouteSnapshot,
        CanActivate,
        CanActivateChild,
        Router,
        RouterStateSnapshot,
        UrlTree
      } from "@angular/router";
import { Observable } from "rxjs";

import { AuthService } from "./auth.service";

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate, CanActivateChild {
  constructor(private authService: AuthService, private router: Router){}

  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): boolean | UrlTree | Observable<boolean | UrlTree> | Promise<boolean | UrlTree> {

	// Se comunica con el servicio authService y le pregunta si está autenticado o no, que devuelve una promesa. Una vez recuperada ka contestación, que será true/false, va a devolver a su vez true/false, pero además en el caso de no estar autenticado va a hacer que se rediriga a la ruta home.
    return this.authService.isAuthenticated()
      .then(
        (authenticated: boolean) => {
          if (authenticated) {
            return true;
          } else {
            this.router.navigate(['/']);
            return false
          }
        }
      );
  }

  

  canActivateChild(childRoute: ActivatedRouteSnapshot, state: RouterStateSnapshot): boolean | UrlTree | Observable<boolean | UrlTree> | Promise<boolean | UrlTree> {
    return this.canActivate(childRoute, state);
  }

  
  

}
```

Pongamos que tenemos el siguiente archivo de rutas:

```typescript
const appRoutes: Routes = [

  {path: '', component: HomeComponent},
  {path: 'users', component: UsersComponent, children: [
    {path: ':id/:name', component: UserComponent},
  ]},
  {
    path: 'servers',
    // canActivate: [AuthGuard] ,
    canActivateChild: [AuthGuard], //Actua de middleware antes de realizar el enrutamiento, por lo que si la función devuelve false NO se lleva a cabo el enrutado.
    component: ServersComponent, children: [
    {path: ':id', component: ServerComponent},
    {path: ':id/edit', component: EditServerComponent},
  ]},
  {path: 'not-found', component: PageNotFoundComponent},
  {path: '**', redirectTo: '/not-found'},
];

@NgModule({
  declarations: [],
  imports: [RouterModule.forRoot(appRoutes),],
  exports: [RouterModule],
})
export class AppRoutingModule{
}
```
 
 
### canDeactivate

Podemos usar este método por ejemplo para preguntar al usuario si estamos seguros de que queremos salir de un componente.

## Uso de transmisión de datos a través de la ruta

Podemos pasar datos estáticos a través de la ruta usando `data: {propiedad: 'dato'}`

```typescript
const appRoutes: Routes = [
  {path: 'not-found', component: ErrorPageComponent, data: {message: 'Page not found'}},
  {path: '**', redirectTo: '/not-found'}
]
```

Podemos luego utilizarlo en el componente que estamos cargando

```typescript
export class ErrorPageComponent implements OnInit {

  errorMessage: string;
  
  constructor(private route: ActivatedRoute){}
  
  ngOnInit(){
    this.route.data.subscribe(
      (data: Data) => {
        this.errorMessage = data['message'];
      }
    )
  }
}
```

```html
<p>{{errorMessage}}</p>
```

## Uso de resolución de datos a través la ruta con Resolver

Usar un Resolver no es más que usar un método que se va a ejecutar anter del renderizado, y que nos puede servir para pasar de forma dinámica datos al componente a cargar.

Primero, vamos a crear un `servicio resolver`
```typescript
import { Injectable } from '@angular/core';
import { ActivatedRouteSnapshot, RouterStateSnapshot, Resolve } from '@angular/route';
import { Observable } from 'rxjs';

import { ServersService } from "../servers.service";

interface Server {
  id: number;
  name: string;
  status: string;
}

@Injectable({
  providedIn: 'root'
})
export class ServerResolver implements Resolve<Server> {

  constructor (private serversService: ServersService){}

  resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<Server> | Promise<Server> | Server {
    return this.serversService.getServer(+route.params['id']);
  }

}
```

El método que el resolver ejecutará será `resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot)` y lo hará antes del renderizado. Pararemos el dato retornado por `resolve` a través de la ruta en el archivo de rutas:

```typescript
{path: 'servers', component: ServersComponent, children: [
  {path: ':id', component: ServerComponent, resolve: {server: ServerResolver}}
]}
```

Necesitaremos importar nuestro servicio resolver `ServerResolver` al archivo de rutas.

Luego, en el componente podemos hacer uso de ese dato a través del `Router`:

```typescript
export class ServerComponent implements OnInit {
  server: {id: number, name: string, status: string};
  serverId: number;

  constructor(private serversService: ServerService, private route; ActivatedRoute, private router: Router){}
  
  ngOnInit(){
    this.route.data.subscribe(
      (data: Data) => {
        this.server = data['server']  //Devuelta por el resolver
      }
    )
  }
}
```

