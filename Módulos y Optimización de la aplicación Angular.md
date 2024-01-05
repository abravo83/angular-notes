
No es necesario tener en un único módulo de angular nuestra aplicación ( `appModule` ). 

Cuando nuestra aplicación alcanza un tamaño considerable es buena idea repartir partes de la aplicación que sean más o menos independientes en distintos módulos de características.

Los componentes que se importan en un módulo no están automáticamente disponibles para el resto de los módulos, por lo tanto la operación de división de una aplicación entre diferentes módulos debe de realizarse de una manera determinada para que en el resto de la aplicación esté disponible y funcione correctamente la parte que hemos dividido.

## Dividiendo la aplicación en distintos módulos.


Mientras que los componentes sólo están disponibles si se importan a los módulos, los servicios declarados  si que lo están por lo tanto tendrán un trato diferente.

Primero debemos de crear nuestro archivo de módulo


```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';
import { ReactiveFormsModule } from '@angular/forms';


@NgModule({
  //Declaramos los componentes que tenga este módulo
  declarations: [
    RecipesComponent,
    RecipeListComponet
  ],

  //Para tener disponible los enlaces de routing debemos importar RouterModule, o cualquier módulo que necesitemos como FormsModule
  imports: [
	// DEbemos incluir tódos los módulos que se vayan a usar en este módulo, salvo los que sólo contengan servicios, ya que estos si están disponibles en otros módules sólo importándolos en AppModule.

	// Para poder utilizar NgFor o NgIf en appModule se importa BrowserModule, pero no podemos importarlo a módulos diferentes de AppModule, para tener disponible esas directivas necesitamos importar un módulo diferente: CommonModule
    CommonModule
    ReactiveFormsModule,
    RouterModule
  ],

   // Y, muy importante, para que luego estén disponibles para la aplicación debemos expotar los componentes, cosa que con appModule no era necesaria
   exports: [
     RecipesComponent,
     RecipeListComponent
  ],
})
export class RecipesModule {}
```


Ahora debemos importar el nuevo módulo en nuestro `appModule` para que funcione con el resto de la aplicación


```typescript

...
// Añadimos en la cabecera la ruta del import
import { RecipesModule } from './recipes.module';


...
//Añadimos en el array imports el tipo o nombre de nuestra clase módulo.
imports: [
  ...
  RecipesModule
]

```


Ahora mismo la aplicación funcionaría sin problemas usando el `routing `de `AppModule,` pero , ¿Y si quisiéramos mover las rutas específicas de nuestra característica al nuevo módulo?

## Moviendo rutas a otro módulo

En nuestro nuevo módulo hemos incluido `RouterModule` para que funcionen las rutas. Si nos fijamos en cómo se usa `RouterModule` en `AppRoutingModule` veremos que se importa con un método en el que definimos el objeto que contiene todas las rutas la la aplicación, quedando así:

```typescript
RouterModule.forRoot(appRoutes)
```

Si nosotros quisiéramos mover parte de las rutas que pertenezcan a la característica que estamos modulando debemos usar un método diferente

```typescript
RouterModule.forChild(recipesRoutes)
```

Para mantener el nuevo módulo más ligero vamos a mantener el `routing` en un archivo aparte. El `Routing` parcial es también un módulo en sí mismo.

```typescript
\\recipes-routing.module.ts

import { NgModule } from '@angular/core';

import { RecipesComponent } from './recipes.component';
import { RecipesStartComponent } from './recipes-start.component';
import { RecipesEditComponent } from './recipes-edit.component';
import { AuthGuard } from './auth/auth.guard';


const routes: Routes = [
  {
    path: 'recipes', 
    component: RecipesComponent, 
    canActivate: [AuthGuard], 
    children: [
      {path: '', component: RecipesStartComponent},
      {path: 'new', component: RecipesEditComponent}      
    ]}
]


@NgModule({
  // ¡ATENTO! Esta es la parte diferente del primer AppRouting. Usar el método forChild en vez de forRoot
  imports: [RouterModule.forChild(routes)],
  //Tambien hay que exportalo
  exports: [RouterModule]
})
export class RecipesRouting {}

```


Ahora añadimos este módulo de `routing` a nuestro módulo de características.

```typescript
\\recipes.module.ts
import { RecipesRouting } from './recipes-routing.module';



imports: [
	CommonModule
    ReactiveFormsModule,
    RecipesRouting
]
```


## Posibles subdivisiones de una aplicación en distintos módulos

### Módulos de características

Donde mueves todos los componentes de una característica o sección a un módulo propio.

### Módulos comunes

Separas en un único módulos los elementos comunes, como por ejemplo componentes de popUps o alertas

### Módulo núcleo

En este módulo se pueden separar del `AppModule` aquellos servicios y componentes diferentes del `AppComponent`,
aunque si se usa en el decorador @Injectable con `providedIn` no tenemos declarados como `providers` ningún servicio. Luego importaríamos el módulo `Core` o `Nucleo` a `AppModule`


## Mejorar el rendimiento de la aplicación gracias a la subdivisión en Módulos y a la carga de módulos usando `Lazy Loading`

Una vez que tenemos nuestra aplicación subdividida en varios módulos podemos mejorar la primera carga de la aplicación haciendo que los módulos que no necesitemos en esa primera carga se carguen usando `Lazy Loading`, que permite retrasar la carga a un momento posterior.

### Implementando `Lazy Loading`

Para implementarlo necesitamos tener nuestra aplicación separada en submódulos. Para que un submódulo de característica se cargue de forma diferida debemos haber incluido sus rutas en el submódulo importando `RouterModule.forChildren({-objeto rutas})`.

Ahora debemos coger la primera ruta raíz, por ejemplo si nuestra característica es `/productos`, con `/productos/a`, `/productos/b`, etc. Debemos coger esa ruta `'productos'` de submódulo y dejarla como  un `string` vacío `''`

En el submódulo:
```typescript

const routes: Routes = [
  {
  path: '',  //Antes 'producto'
  component: ProductoComponent,
  canActivate: [AuthGuard],
  children: [
    {path: '', component: ProductosComponent},
    {path: 'a', component: ProductoA },
    ...
  ]
  }
]
```

Y en nuestro módulo principal donde está AppRouting

```typescript

const routes: Routes = [
  {path: '', redirectTo: '/inicio', pathMatch: 'full'},
  {path: 'producto', loadChildren},
  ...
]

```

Y con ese loadChildren hacemos LazyLoading