
Son componentes que funcionan de forma totalmente independiente. Se incluyó a partir de la versión 14 y se usa por defecto en la versión 17.

No sólo los componentes puede ser `Standalone`, también las directivas y los `Pipes`

En vez de usar `appModule`, cada componente contiene su propio `módulo` dentro de si mismo.

Entonces, su decorador `@component` va a contener un array de `imports` para importar los módulos que se usen en el componente, los componentes hijos, y luego exportamos el componente y aquello que debemos exportar.

De esta forma nos libramos totalmente del uso de @NgModules

Un componente `standalone` tiene un decorador `@Component` algo distinto:

```typescript
@Component({
  standalone: true,
  selector: 'app-componente-ejemplo',
  templateUrl: './componente-ejemplo.component.html',
  styleUrls: ['./componente-ejemplo.component.css'],
})
```

Con esa primera propiedad, `standalone: true`, ya hemos definido a nuestro componente como `standalone`, y a partir de aquí su comportamiento va a ser diferente.

Ahora no va a permitir que este componente se declare en ningún otro módulo (recordemos que sólo se puede declarar un componente en un módulo, para usarlo en otros módulos debemos importar el módulo que lo declara). 

Porque ahora este componente ya se declara *en sí mismo*.

Además ya no va a tener disponibles todos los módulos que hemos importado en `AppModule` sino que necesita sus propias importaciones:

```typescript
@Component({
  standalone: true,
  imports: [CommonModule, BrowserModule, Router],
  provider: [],
  selector: 'app-componente-ejemplo',
  templateUrl: './componente-ejemplo.component.html',
  styleUrls: ['./componente-ejemplo.component.css'],
})
export class ComponenteEjemplo {}
```

Dentro del array de importaciones también podemos importar módulos propios (como `AppModule`).

## Proceso de migración de un componente a `Standalone`

Para migrar un componente debemos cambiar su decorador para que tenga la propiedad `standalone: true`, y además debemos cambiar la declaración de ese componente en un módulo y **cambiarla por una importación** ya que en realidad ese componente se está comportando ahora mismo como un módulo para ese otro módulo.

```typescript
 @NgModule({
   declarations: [NonStandaloneComponent],
   imports: [BrowserModule, EjemploStandaloneComponent],
   providers: [],
   bootstrap: [AppComponent];
 })
 export class AppModule {}
```

Además debemos importar en ese componente `standalone` cualquier módulo o componente que esté usando.

### Haciendo `Standalone` a `AppComponent`

Un componente algo diferente a la hora de convertir a `standalone` es el componente `AppComponent`, porque es en el que hacemos `bootstraping`.

Para que funcionen correctamente debemos modificar la función que realiza el `bootstraping` usando un nuevo método llamado `bootstrapApplication` en `main.ts`

```typescript
\\main.ts

import { enableProdMode } from '@angular/core';
import { bootstrapApplication } from '@angular/platform-browser';

import { AppComponent } from './app/app.component';
import { environment } from './environments/environment';

if (environment.production) {
  enableProdMode();
}

bootstrapApplicaciont(AppComponent);
```

## `Routing` con componentes `Standalone`

Lo primero que debemos hacer es importar el módulo `RouterModule` en nuestro componente `standalone`.

Después en nuestro `main.ts` debemos indicarle a toda la aplicación (Ya que no nos sirve a partir de los `NgModules`, ya que los componentes `standalone` no los usan), dónde ubicar nuevamente el archivo principal de `routing` (que es un módulo convencionalmente llamado `AppRoutingModule`) para que esté disponible a toda la aplicación sin pasar por ningún módulo y lo vamos a hacer usando un método llamado **`importProvidersFrom`**:

```typescript
import { enableProdMod, importProvidersFrom } from '@angular/core';
import { bootstrapApplication } from '@angular/platform-browser';

import { AppRoutingModule } from './app/app-routing-module';
import { AppComponent } from './app/app.component';
import { environment } from './environment/environment';

if (environment.production) {
  enableProdMode();
}

bootstrapApplication(AppComponent, {
  providers: [
    importProvidersFrom(AppRoutingModule);
  ]
})

```

Una vez definido el archivo de rutas principal podemos definir las rutas anidadas que pueda tener definidas un componente `standalone` o las rutas que deben cargarse de forma `Lazy loading` como ya vimos.

La **novedad** es una **nueva sintaxis** para hacer `LazyLoading` de componentes `standalone` que en vez de usar el método `loadChildren` usamos **`loadComponent`**, y nos permite hacer `Lazy loading` directamente de componentes `standalone` sin necesidad de separarlos en módulos de rutas independientes que se cargan de forma `lazy`, como ocurre al usar el método `loadChildren`

```typescript
import { NgModule } from '@angular/core';

import { AboutComponent } from './app/about.component';
import { DashboardRoutingModule } from './app/dashboard-routing.module';

const routes = [
  {path: '', component: 'InicioComponent', fullPathMatch: true},
  {path: 'about', loadComponent: () => import('./app/about.component').then((m) => m.AboutComponent)},
  {path: 'dashboard', loadChildren: () => import('./app/dashboard.module').then((m) => m.DashboardRoutingModule)},
]

@NgModule({
  imports: [RouterModule.forRoots(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}

```

Otra estrategia nueva para cargar de forma `lazy` componentes `standalone` es usar `loadChildren` pero apuntando a un archivo `.ts` que exporte una constante donde definamos un objeto de rutas para esa parte de la aplicación que queremos que se cargue de forma `lazy`, de forma similar a como hacíamos cuando definíamos un módulo independiente para esa parte:

```typescript
\\ dashboard/routes.ts

//Necesitamos definir el objeto de rutas con el tipo Route
import { Route } from '@angular/router';

// Necesitamos importar los componentes de cada ruta
import { DashboardComponent } from './app/dashboard.component';
import { TodayCoponent } from './app/today.component';

export const DASHBOARD_ROUTES: Route[] = [
  {path: '', component: DashboardComponent},
  {path: 'today', component: TodayComponent},
];

```

Entonces modificaríamos nuestro `AppRoutingModule` para importar este objeto de rutas en el `loadChildren`:

```typescript
...

const routes = [
  {path: '', component: 'InicioComponent', fullPathMatch: true},
  {path: 'about', loadComponent: () => import('./app/about.component').then((m) => m.AboutComponent)},
  {path: 'dashboard', loadChildren: () => import('./app/dashboard/routes').then((m) => m.DASHBOARD_ROUTES)},
]

...

```

