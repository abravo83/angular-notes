
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