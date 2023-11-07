
## ¿Qué es un Observable?

Un Observable es una *construcción* a la que te "Subscribes" para recibir un dato. En esencia es algo similar a una Promesa de la esperamos recibir datos o un error.

Como las promesas, son también una forma de gestionar datos que recibimos de manera asíncrona, es decir, con un retardo de tiempo mientras que el resto de la aplicación sigue funcionando.

Un ejemplo
```typescript
import { HttpClient } from '@angular/common/http';
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-ejemplo-obs',
  templateUrl: './ejemplo-obs.html',
  styleUrls: ['./ejemplo-obs.css']
})
export class EjemploObs implements OnInit {

  variableLocal!: any;

  constructor(private httpClient: HttpClient){}
	
  ngOnInit(): void {
    this.httpClient.get('/thisApi/item').subscribe(
      (data) => {
        this.variableLocal = data;
      }
    )
  }
}

```

En el ejemplo estoy usando la petición GET a través de `httpClient` que devuelve un observable, por lo que estoy subscrito a él para guardar el dato en la variable `variableLocal`. La petición tardará algo de tiempo, y mientras tanto el resto de la aplicación está funcionando.

## Dos categorías de Observables: Observables de Angular y Observables de `RxJS`.

Originalmente el concepto de observable viene una librería externa llamada `RxJS`.

Podemos usar Observables directamente de esa librería, que tiene además muchos tipos distintos de observables (`Subjects, intervals, etc`) o podemos usar los Observables que ya ha creado Angular para nosotros.

Esta diferencia es **Muy importante** porque mientras que los Observables que se han creado con la librería `RxJS` necesitan *darse de baja en la subscripción* al destruir el componente (En `NgOnDestroy`), los que se crean directamente desde Angular no necesitan darse de baja.

Si no damos de baja una subscripción de `RxJS` la subscripción seguirá abierta todo el tiempo, se duplicará cada vez que reabramos el componente si esta subscripción se crea en `NgOnInit` (Ciclo de vida - Pendiente) y se producirán pérdidas de memoria.

## Darse de baja de Subscripciones

Para darse de baja de una subscripción necesitaremos haberla guardado previamente en una variable de tipo `Subscription`:

```typescript
@Component({
  selector: '...',
  templateUrl: '...',
  styleUrls: ['...']
})
export class EjemploUnsubscribeComponent implements OnInit, OnDestroy {
  userActivated: boolean = false;
  private activatedSub: Subscription;

  constructor(private userService: UserService) {
  }

  ngOnInit(): void {
    this.activatedSub = this.userService.subscribe(
      (didActivate) => {
        this.userActivated = didActivate;
      }
    )
  }

  ngOnDestroy(): void {
    this.activatedSub.unsubscribe();
  }

}
```



## Operadores de `RxJS`

Nos permiten modificar los datos al mismo momento de recibirlos en la subscripción, por lo que los datos recibidos en el `callback` de la subscripción ya son datos sobre los que se ha ejecutado la operación.

Son similares a los métodos que tenemos disponibles para los `arrays`: (`filter, map, ...`)

Se utilizan dentro del método pipe():

```typescript
import { Subscription, Observable } from 'rxjs';
import { map, filter } from 'rxjs/operators';

...

this.pipedSubscription = customObservable
  .pipe(filter(data => data > 0), map(data => data * 2)) //Podemos encadenar operadores separados por coma.
  .subscribe(data => console.log(data));

```
 
 Tras el método `pipe()` seguimos con nuestra subscripción común con el método `subscribe()`


## EventEmitter y Subject

Son dos tipos de observables que nos pueden ser útiles.

El primero es sencillamente un evento, que nos puede servir para comunicar componentes, pero también nos sirve para transmitir datos. Generalmente lo crearemos en un servicio y desde un componente emitiremos el evento al que estaremos susbcritos desde otros componentes. En el callback podremos renovar cualquier dato que nos interese y así utilizar la emisión del evento como señal de que se ha actualizado algún datos del servicio y que podemos proceder a actualizar el componente.
```typescript
 // En un componente
this.userService.actualizarComponente.emit()  //EventEmitter definido en UserService
```

```typescript
// En otro componente

this.userService.actualizarComponente.subscribe(
  data => this.variableLocal = this.userService.variableActualizada
)
```


El segundo, el `Subject` es de `RxJS` y funciona de forma similar, pero en vez de utilizar `.emit()` usaremos `.next(valor)` y es una manera *¿más recomendada?* de ser usado en vez de `EventEmmiter`, salvo en el caso de que lo estemos usando para emitir un evento a la plantilla HTML a través de 
