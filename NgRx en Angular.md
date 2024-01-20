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
Esta forma es útil para ver lo que `createReducer` hace por nosotros, pero si tenemos una versión donde este último está disponible es recomendable usarlo en vez de definir nosotros mismos la función `reducer`

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

## ¿ Cómo se modifican los datos del `Store` ?

Para modificar datos del `Store` necesitamos `Actions` que llamen a los `Reducers`, ya que nosotros no tenemos acceso directo al `Reducer`

### Creando nuestra `Action` o Acción

Para crear nuestro `Action` vamos a hacer al igual que hicimos con `Reducer` y crear un nuevo archivo en nuestra carpeta `Store` que defina el `Action` de nuestra variable `counter` y lo llamaremos `counter.actions.ts`, y como nuestra variable es un contador que necesita ser incrementado cuando ocurre algún evento, vamos a definir el método que incrementa dicho `counter`

```typescript
import { createAction } from '@ngrx/store';

// La función createAction toma un argumento necesario: un identificador único para la acción, que por convención usa el formato '[nombre de la propiedad del Store] Nombre de la acción'.
export const increment = createAction(
  '[Counter] Increment'
)
```

Entonces, nuestro `Reducer` o Reductor 'escuchará' el `Action` o Acción  para modificar el dato del `Store` o Almacén. Y escuchará usando el método `on()` que importamos de `@ngrx/store`, que definimos como argumento dentro de la función `createReducer` y que toma como primer argumento nuestra acción y como segundo argumento el `callback` que va a ejecutar cuando reciba el evento de la acción y este `callback` a su vez toma como argumento el estado actual y devuelve el nuevo estado. 


```typescript
import { createReducer, on } from '@ngrx/store';

import { increment } from './counter.actions';

const initialState = 0;

export const counterReducer = createReducer(
  initialState,
  on(increment, (state) => state + 1)
);
```

### Llamando a nuestra `Action` o Acción

¿Desde dónde vamos a llamar a nuestra Acción? Si recordamos el esquema, el Componente interactúa con la Acción y esta lo hace con el Reductor:

![[NgRx selector.png]]

En nuestro componente inyectaremos el Almacén y nuestro objeto Almacén tiene disponible el método `dispatch()` que nos permite "despachar" la Acción, que es la manera en la que expresamos la orden de ejecutar dicha Acción

El método `dispatch()` tomará como argumento el nombre de nuestra Acción importada que se ejecutará como de un método se tratara, y esto hará que el `callback` que tiene definida nuestra Acción se ejecute.

```typescript
import { Component } from '@angular/core';
import { Store } from '@ngrx/store'; 

import { increment } from './store/counter.actions';

@Component({
  selector: 'app-counter-control',
  templateUrl: './counter-control.component.html',
  styleUrls: ['./counter-control.component.css'],
  standalone: true,
})
export class CounterControlComponent {
  constructor(private store: Store){}

  increment(){
    this.store.dispatch(increment());
  }
}

```

Pero nuestra Acción no siempre se va a simplemente ejecutar. Es bastante común que desde el Componente le pasemos datos a nuestra Acción para que trabaje con ellos y a partir de esos datos actualice nuestra propiedad del Almacén.

Conseguimos agregar a nuestra Acción la capacidad de pasar datos al ser despachada usando el método `props` de `@ngrx/store`, definiendo el tipo de datos que va a portar y ejecutando dicho método como segundo argumento de nuestro método `createAction`

```typescript
import { createAction, props } from '@ngrx/store';

// La función createAction toma un argumento necesario: un identificador único para la acción, que por convención usa el formato '[nombre de la propiedad del Store] Nombre de la acción'.
export const increment = createAction(
  '[Counter] Increment',
  props<{value: number}>()
)
```

Modificamos el Reductor para usar como segundo argumento la acción

```typescript
import { createReducer, on } from '@ngrx/store';

import { increment } from './counter.actions';

const initialState = 0;

export const counterReducer = createReducer(
  initialState,
  on(increment, (state, action) => state + action.value)
);
```

Y en nuestro componente, para pasarle el dato usamos un objeto con la estructura de datos que definimos y el valor que queremos pasarle

```typescript
...
export class CounterControlComponent {
  constructor(private store: Store){}

  increment(){
    this.store.dispatch(increment({value: 2}));
  }
}

```

## Selectores ¿Qué son?

Ya hemos visto que para presentar los datos guardados en el almacén podemos usar el método `store.select('nombreDeVariableAlmacenada')` inyectando el `store` en el componente:

```typescript
import { Store } from '@ngrx/store';

@Component({
//...
})
export class EjemploComponent {
  contador$: Observable<number> = ;
  
  constructor(private store: Store<{contador: number}>){
    this.contador$ = store.select('contador');
  }
}

```

Pues bien, hay una forma adicional de obtener valores: Selectores.

Un selector es una función que obtiene el valor estado que obtiene todos los estados del almacén y devuelve el valor del almacén que nos interesa para ese selector

```typescript
export const selectorContador = (state: {contador: number}) => state.contador;
```

Si guardamos dicha función en un archivo independiente .ts y lo importamos a nuestro componente ahora podríamos usar el componente de una forma similar al anterior `store.select()`:


```typescript
import { Store } from '@ngrx/store';
import { selectContador } from './store/contador.selectors'

@Component({
//...
})
export class EjemploComponent {
  contador$: Observable<number> = ;
  
  constructor(private store: Store<{contador: number}>){
    this.contador$ = store.select(selectContador);
  }
}
```


De esta forma podemos reutilizar el selector cada vez que un componente distinto desee usar el selector, sin necesidad de tener que redefinir la función de selector cada vez que queremos solicitar el dato de nuestro almacén, o tener diferentes funciones que llamen al dato del almacén y realicen alguna operación sobre ese dato, teniendo todas estas funciones que interactúan sobre el mismo dato centralizadas en el mismo archivo.

Incluso, yendo un poco más allá, podemos crear nuevos métodos selectores a partir de otros métodos o combinaciones de ellos usando `createSelector()` de `@ngrx/store`

```typescript
import createSelector from '@ngrx/store';

export const selectCount = (state: {counter : number}) => state.counter;

// Usa el selector selectCount, y a su resultado lo multiplica por dos.
export const selectDoubleCount = createSelector(
  selectCount,
  (state) => state * 2
);

```

y en un componente que llame a los dos selectores nos quedaría así:

```typescript
import { Store} from '@ngrx/store';

import { selectCount, selectDoubleCount } from '../store/counter.selectors';

 @Component({
   selector: 'app-counter-output',
   templateUrl: './counter-output.component.html',
   styleUrls: ['./counter-output.component.css'],
 })
 export class CounterOutputComponent {
   count$: Observable<number>;
   doubleCount$: Observable<number>;

   contructor(private store: Store<{counter: number}>){
     this.count$ = store.select(selectCount);
     this.doubleCount$ = store.select(selectDoubleCount);
   }
 }
```

Y en su `html`

```html
<p class="counter">{{ count$ | async }}</p>
<p class="counter">Double: {{ doubleCount$ | async }}</p>
```

## `Effects` o Efectos

