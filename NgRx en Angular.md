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

Los efectos colaterales son, en el contexto de Angular, cualquier cosa que cambia y que no está directamente relacionada con un cambio en la IU.

Nuestras funciones reductoras NO deben contener efectos colaterales. Deben de ser totalmente síncronas, por lo que no deben de contener peticiones http ni hacer `console.log()`. Deben ceñirse sólo a la función de generar nuevos estados del dato del almacén.

Pero, como a veces necesitamos poder realizar tales acciones,  para esto usamos los `Effects`

### Instalando `Effects`

En nuestra consola debemos usar el siguiente comando:

```bash
ng add @ngrx/effects
```

Esto no sólo va a instalar el paquete, sino que modificará `AppModule` ( o `main.ts` si tenemos componentes `standalone` ) para incluir en el array de importaciones en el primer caso:

```typescript
imports: [
  //...,
  EffectsModule.forRoot([]),
],
```

o en el segundo caso, incluir `provide effects` en el array de `providers`:

```typescript
providers: [..., provideEffects()],
```

ambos importados de `ngrx/effects`

### Definiendo un `Effect`

Vamos a crear un nuevo archivo en nuestra carpeta `store` al que vamos a llamar `counter.effects.ts`.

En este archivo vamos a crear una nueva clase que vamos a llamar `CounterEffects`, con el decorador @Injectable(),  que va a contener varias propiedades para cada efecto que queramos ejecutar

```typescript
import { @Injectable } from '@angular/core';
import { createEffect, Actions } from '@ngrx/effects';
import { tap } from 'rxjs';

import { increment } from './counter.actions';

@Injectable()
export class CounterEffects {

  //action$.pipe hace que tengamos acceso a un "disparador" que se activa cada vez que se realiza un Action.
  saveCount = createEffect( 
      () => this.actions$.pipe(
        //El método ofType nos permite delimitar que tipo de acción hace que se "dispare" el efecto
        ofType(increment),
        tap((action)=>{
          console.log(action);
          localStorage.setItem('count', action.value.toString());
        }
      )
    ),
    // Tras el callback lleva un objeto de configuración que no se van a despachar nuevas acciones.
    // El valor por defecto es true.
    { dispatch: false };
  )


  constructor(private action$: Actions) {}
}
```

Por último, debemos decirle a Angular que hemos creado un nuevo efecto, que de momento está en un archivo aislado, y eso lo hacemos dentro de las propiedades que modificó `ng add @ngrx/effect

Bien en `AppModule`:

```typescript
import { CounterEffects } from './store/counter.effect'

//...
imports: [
  //...,
  EffectsModule.forRoot([CounterEffects]),
],

```

o bien en `main.ts`:

```typescript
import { provideEffects } from '@ngrx/effects';

import { CounterEffects } from './store/counter.effect'

//...
providers: [..., provideEffects([CounterEffects])],
```

### Usando datos del almacén de Efectos

En el anterior apartado no hemos usado el dato que actualmente figuraba en el almacén para `counter`, sino la cantidad por la que se modifica dicho dato en el almacén, es decir, la cantidad del `action` (`action.value`).

Si queremos usar realmente la cantidad del almacén, para poder hacer uso de esa cantidad, tenemos disponible el método: `withLatestFrom` de `rxjs`. Este nos permite obtener un valor que posteriormente estará disponible en el siguiente operador del `pipe` de `actions$`.

Inyectamos nuestro almacén a través del constructor, y entonces podemos acceder al `Select` con el método:

```typescript
withLatestFrom(this.store.select(selectCount)),
tap(...
```

y ahora como argumento de `tap` vamos a recibir un arau, donde el primer elemento es nuestro action, y el segundo elemento será el retorno de `withLatestFrom`

```typescript
withLatestFrom(this.store.select(selectCount)),
tap(([action, counter])=>{
  console.log(action);
  localStorage.setItem('count', counter.toString());
})
```

### Añadiendo un segundo `Effect`

Vamos a crear un segundo efecto, pero en esta ocasión, nuestro efecto si va a despachar una nueva acción: vamos a recuperar el dato almacenado en `local storage` y vamos a realizar una acción con el: guardarlo en el almacén de `NgRx`.

Así, cuando recarguemos la aplicación podemos recuperar el último valor del contador.

Para esto vamos a añadir dos nuevas acciones a nuestro '`counter.actions.ts`':
1. `init`: va a "disparar" el efecto que va a conseguir el dato que hemos almacenado en el `local storage` y no lleva valor aparejado a la acción.
2. `set`: Usa ese valor recuperado y lo fija en nuestro almacén.

```typescript
//counter.actions.ts
export const init = createAction('[ Counter ] init');
export const set = createAction(
  '[ Counter ] set',
  props<{value: number}>(),
  )
```

Ahora, en el `Reducer` añadiremos un nuevo disparador para nuestra acción `set` que lo único que va a realizar es devolver el valor del `action`:

```typescript
//counter.reducer.ts
import { decrement, increment, set } from './counter.action.ts';

const initialState = 0;

export const counterReducer = createReducer(
  initialState,
  on(increment, (state, action)=> state + action.value),
  on(decrement, (state, action)=> state - action.value),
  on(set, (state, action)=> action.value),
);

```

Ahora debemos de agregar la lógica para nuestro archivo de efectos:

```typescript
//counter.effects.ts

//...
@Inyectable()
export class CounterEffects {
  loadCount = createEffect(()=>this.action$.pipe(
    ofType(init),
    switchMap(()=>{
      const storedCounter = localStorage.getItem('count');
      if (storedCounter){
        //of() se importa de rxjs y sirve para convertir en objervable un retorno.
        return of(set({value: +storedCounter}))
      }

      return set({value: 0});

    })
  ))
}
```

Ya solo nos queda despachar la acción `init`, que va a recoger el dato guardado en el `local storage` y devolverlo usando la acción `set`. Esta acción `init` va a ser despachada en el ciclo `onInit` del componente principal `AppComponent`, aunque la situación en la que vamos a despachar esta acción depende de cada caso, y la podríamos llamar desde cualquier componente donde tenga sentido colocarlo:

```typescript
//app.component.ts
import { Component, onInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { init } from './store/counter.action';

@Component({
  selector: 'app-coponent',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {

  constructor (private store: Store){}

  ngOnInit(): void {
    this.store.dispatch(init());
  }
}

```