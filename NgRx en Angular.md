## ¿Qué es `NgRx`?

Es una librería de Angular que nos permite guardar estado de nuestra aplicación. Por tanto se define como una `Solución de Administración de Estado` o `State Management Solution`.

## ¿Cómo funciona `NgRx`?

`NgRx` crea un almacén de datos o `Store`.

Cada vez que un componente necesita leer un dato va a solicitarlo al almacén de datos. También puede escuchar cambios en los datos para hacer las modificaciones en la IU. Y entre medias de estas transacciones nos encontramos con un concepto propio de `NgRx` llamado `Selector`.

![[NgRx selector.png]]

Y al mismo tiempo, cuando el componente necesita modificar datos que están en el almacén tiene un método estandarizado de hacerlo mediante `Actions` que disparan otro método llamado `Reducer` que se encarga de la lógica de cambio de estado.

Además, otro elemento llamado `Effect` realiza cambios "colaterales" o `Side effects` que realizan cambios que no tienen que ver con el almacén de datos.

## ¿Cuándo usar `NgRx`?

En un proyecto con un tamaño medio a grande donde guardar el estado sea algo más complejo y merezca la pena la complicación de usar `NgRx`

## ¿ Cómo instalar `NgRx`

```bash
ng add @ngrx/store
```

El instalador incluirá si estamos trabajando con `@NgModule` al array `imports`:

```typescript
import { StoreModule } from '@ngrx/store';
...

imports: [..., StoreModule.forRoot({}, {})]

```

Que fijará el `Store` en nuestro proyecto.

Si estamos en un proyecto `standalone` lo que se modificará es el archivo `main.ts`

```typescript
import { provideStore } from '@ngrx/store';
...

bootstrapApplication(AppComponent, {
  providers: [provideStore()]
})

```

Ambos métodos lo que hacen es añadir un `Store` vacío en nuestra aplicación.

## Agregando el `Store` al proyecto

Ahora debemos crear una carpeta donde va a quedar contenida la lógica de nuestro `Store`:

```bash
/mydocuments/myapp/src/app mkdir store
```

En esta carpeta vamos a ir añadiendo las diferentes lógicas

### Añadiendo la lógica del `Reducer`

En nuestra carpeta vamos a crear un archivo llamado `counter.reducer.ts` con el siguiente código para un único valor que vamos a guardar en la `Store` llamado `counter`

```typescript
import { createReducer } from '@ngrx/store';


const initialState = 0; // El estado puede ser cualquier tipo de dato.
export const counterReducer = createReducer(initialState);  //Definimos el estado inicial al crear nuestro reducer.

```

### Conectando el `Reducer` a la aplicación

En nuestro `AppModule` (Si no es una aplicación `Standalone`) vamos a definir el dato que estamos guardando en el store:

```typescript
...
import { counterReducer } from './store/counter.reducer';

@NgModule({
  declarations: [
    AppComponent,
    CounterOutputComponent,
    CounterControlComponent,
  ],
  imports: [
    BrowserModule,
    StoreModule.forRoot({
      counter: counterReducer,
      //auth: authReducer, //Por ejemplo
      //user: userReducer, //Por ejemplo
      ...
    })
  ]
})

```

Si estamos usando componentes `Standalone` la lógica va a ser similar, pero en `main.ts`

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { provideStore } from '@ngrx/store';

import { AppComponent } fomr './app/component';
import { counterReducer } from './app/store/counter.reducer';

bootstrapApplication(AppComponent, {
  providers: [provideStore({
    counter: counterReducer,
    // auth: authReducer,
    // user: userReducer,
  })]
})
```

Ahora bien, si estamos trabajando con versiones antiguas de Angular es posible el método `createReducer()` no esté disponible , e igualmente no habrá componentes `standalone` así que de esa parte los olvidamos de momento, por lo que crear un `reducer` es algo diferente, aunque es en realidad lo que realiza la función `createReducer` por nosotros. En nuestro archivo `counter.reducer.ts` vamos a escribir:

```typescript
const initialState = 0;

export function counterReducer(state = initialState) {
  return state;
}

```

Y esta forma sirve para las aplicaciones que no tienen `counterReducer` pero también nos serviría para los que lo tienen, porque al final lo que está haciendo `createReducer` es fijar un prototipo de esta función (`counterReducer`) por nosotros.

## ¿ Cómo se accede a los datos almacenados en el `Store`?

Para acceder a los datos del `Store` podemos acceder a nuestro `Store` vamos inyectar en nuestro componente el servicio `Store` de ' @ngrx/store'

```typescript
import { Component } from '@angular/core';
import { Observable } from 'rxjs';
import { Store } from '@ngrx/store';
import { AsyncPipe } from '@angular/common';

@Component({
  selector: 'app-counter-output',
  templateUrl: './counter-output.component.html';
  styleUrls: ['./counter.output.component.css'],
  standalonte: true,
  imports: [AsyncPipe]
})
export class CounterOutputComponent {
  //El decorador $ al final en una convención para variables que guardan Observables
  count$: Observable<number>;

  constructor(private store: Store<{counter: number}>){
    this.count$ = store.select('counter');  // El counter que definimos para nuestro counterReducer
  }
}
```

Y en el `html` lo usaríamos así

```html
<p>{{ count$ | async}}</p>
<!-- Usamos el pype async para decirle a angular que es una variable que puede modificarse posteriormente por un proceso asíncrono al ser un observable -->
```

