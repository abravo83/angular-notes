
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

