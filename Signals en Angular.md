
## ¿Para qué?

Son una forma de actualizar datos en la aplicación. Sin `Signals` las actualizaciones de datos se realizan por lo que llamamos `detección de cambios basasdos en zona` que viene de una librería llamada `zone.js` que se basa en la modificación de propiedades en las clases de los componentes y servicio.

La ventaja de la los cambios de zona es que son detectados de forma automática. y se actualiza el DOM de forma automática. Su desventaja es que el rendimiento y el tamaño podrían ser menor.

Las `Signals` nos ofrecen una forma distinta de gestionar y detectar esos cambios y nos permiten aligerar la aplicación y el rendimiento. En contra somos nosotros los que debemos de ocuparnos de que los cambios se transmitan y actualicen a lo largo del DOM.

## ¿Cómo funciona Signals?

Necesitamos usar Angular >= 16.

Para usar una señal vamos a definir en nuestro `AppComponent `un componente SignalComponent`

```typescript
import { Component } from '@angular/core';

import { DefaultComponent } from './default/default.component';
import { SignalsComponent } from './signals/signals.component';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  standalone: true,
  imports: [DefaultComponent, SignalsComponent]
})
export class AppComponent {}

```

Y en el componente `SubjectComponent` vamos a modificarlo para usar `Signals` en vez de `zone-based`, en el componente originalmente usábamos las propiedades `counter: number` y `actions: string[]`

```typescript
...
import { NgFor } from '@angular/common';
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-signals',
  templateUrl: './signals/signals.component.html',
  standalone: true,
  imports: [NgFor]
})
export class SignalsComponent {
  //actions: string[] = [];
  actions: signal<string[]>([]);
  
  //counter: number = 0;
  couter = signal(0);

  increment(){
    this.counter.update((counterAntiguo) => counterAntiguo + 1);
  }

  decrement(){
    this.counter.update((counterAntiguo) => counterAntiguo - 1);
  }

}


```

Para actualizar una `Signal` tenemos a disposición tres métodos:

* `.set()`  para fijar un  valor

```typescript
this.counter.set(2);
```

* `.update()` para modificar un valor basándote en el valor previo
 
```typescript
this.counter.update(antiguoValor => antiguoValor + 1 )
```

* `mutate()` para modificar el valor de datos que pueden ser mutados basándonos en el valor previo (también);

```typescript
this.counter.mutate(oldArray => oldArray.push('Nuevo elemento del array'))

//Posible
this.counter.update(oldArray => [oldArray..., 'Nuevo elemento del array'])

//!!!  NO POSIBLE !!!
this.counter.update(oldArray => oldArray.push('Nuevo elemento del array'))

// COn mutate podemos ejecutar métodos y modificar de manera directa el objeto mutable.

```


## ¿Cómo mostrar o consumir esas `Signals`

Para mostrar una `Signal` en nuestra plantilla o recuperar su valor en `.ts` no debemos de referenciarla directamente como hacíamos con las propiedades, sino que debemos de referenciarlas como un método: `counter()`

```html
<div>
  <p>Counter: {{ counter() }}</p>
</div>
```


```typescript
this.counter.set(this.counter() + 1);  //Nos sirve también para incrementar el valor anterior
```



## Usando `Signals` junto a `Computed`

`computed` es elemento importado de `@angular/core` que se usa para devolver valores basados en cálculos hechos sobre los valores que contienen `signals`.

Si necesitamos guardar un valor que depende de la lectura y modificación de otro valor que está contenido en un `signal` debemos usar `computed` para que recupere el valor de la `signal`, realice el cálculo y nos devuelva el valor.

La pregunta que nos deberíamos hacer entonces, es ¿por qué no simplemente recuperar el valor de la `signal` llamándola así:

```typescript
import { Component, signal, computed } from '@angular/core';

...

counter = signal(3);
doubleCounter = this.counter() * 2;
```

Y la respuesta es que si lo hacemos así, el valor de `doubleCounter` no se va a actualizar cada vez que se modifique la `signal`. Pero si usamos `computed` si que lo hará:

```typescript
import { Component, signal, computed } from '@angular/core';

...

counter = signal(3);
doubleCounter = computed(()=> this.counter() * 2);
```

## ¿ Cómo se usa un `computed`?

Como ya hemos visto en el ejemplo, un computed se usa ejecutando la función `computed()` usando como argumento un callback de una función anónima sin argumentos dentro de la cual recuperamos el valor de la signal que queremos usar y le aplicamos el cálculo que va a modificar su valor:

```typescript

example = computed( ()=> this.signalExample() * 3.1416^2  );
```


## ¿ Cómo consumiríamos un `computed`?

Para  ver el valor de un `computed` tanto en .ts como en el .html lo vamos a hacer de forma similar a una `signal`, ejecutándolo como un método de la clase del componente:

```html
<div>{{ doubleCounter() }}</div>
```

Y cada vez que la `signal` se modifique también lo hará el valor `computed`.


## Usando `signals` junto a `effect`

`effect` tiene similitudes a `computed` que hacen que ambos sigan el valor de un `signal`, pero `computed` evalúa que `signals` se llaman dentro del bloque de código de la función `effect` y ejecuta dicho código cada vez que se modifica el valor de una de esas `signals`.

Típicamente se ejecuta en el constructor o en `onInit`, y también se importa desde `@angular/core`;

```typescript
constructor(){
  effect(()=>{
    console.log(this.signalExample());
    this.updatesCounter += 1;
  })
}

```

