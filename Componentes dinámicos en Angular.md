
Los componentes dinámicos son componentes que se generan durante el tiempo de ejecución para acciones muy concretas como modales, `popups`, avisos, etc...

Son componentes que están ahí, pero no siempre se muestran, sino que se muestran cuando ocurre algo específico, y parte del contenido que se muestra está gestionado por el código que se activa al ocurrir ese evento específico.


## Creando un  componente de aviso "semi-dinámico"

Vamos a crear primero un componente para usarlo como modal de alerta o aviso

```typescript
\\/modales/alert.component.ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-alert',
  templateUrl: './alert.component.html',
  styleUrls: ['./alert.component.css']
})
export class AlertComponent {
  @Input() message: string;
}
```

```html
\\/modales/alert.component.html
<div class="backdrop"></div>
<div class="alert-box">
  <p>{{ mensaje }}</p>
  <div class="alert-box-actions">
    <button class="btn btn-primary">Cerrar</button>
  </div>
</div>
```

```css
\\/modales/alert.component.css

.backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0,0,0,0.75);
  z-index: 50;
}

.alert-box {
  position: fixed;
  top: 30vh;
  left: 20vw;
  width: 60vwM;
  padding: 16px;
  z-index: 100;
  background: white;
  box-shadow: 0 2px 8px rgba(0,0,0,0.26);
}

.alert-box-actions {
  text-align: right;
}

```


Y lo agregamos al array `declarations` de nuestro `app-module` para tenerlo disponible en nuestra aplicación.

## Activando el componente de aviso desde el código de un servicio o un componente.


Si nos imaginamos un componente en cuyo `html` ya teníamos una caja de aviso:

```html
<div>
  <div class="alert alert-danger" *ngIf="error">
    <p>{{ error }}</p>
  </div>
</div>
```

Lo sustituiríamos por

```html
<div>
  <app-alert [message]="error" *ngIf="error"></app-alert>
</div>
```

Y ya tendríamos nuestro componente semi-dinámico

## Haciendo el componente dinámico

El componentes se pude terminar de hacer dinámico si hacemos que se cierre desde si mismo para poder seguir con el uso de la aplicación. Para ello vamos a hacer dos cosas, emitir un evento desde el propio componente y desde el componente padre recibir dicho evento para modificar la condición de  `*ngIf` para que no se muestre el modal

```typescript
\\/modales/alert.component.ts
import { Component, Input, Output, EventEmmiter } from '@angular/core';

@Component({
  selector: 'app-alert',
  templateUrl: './alert.component.html',
  styleUrls: ['./alert.component.css']
})
export class AlertComponent {
  @Input() message: string;
  @Output() close = new EventEmmiter<void>();

  onClose() {
   this.close.emit();
  }
}
```

```html
<div>
  <app-alert [message]="error" *ngIf="error" (close)="removeError()"></app-alert>
</div>
```

Donde `removeError()` sería un método del componente padre que simplemente vuelve la variable `error=null` haciendo que el componente `app-alert` no se muestre.

## Otra forma de crear un componente dinámico

Para hacer el contenido del componente dinámico vamos a utilizar una utilidad de Angular llamada `ComponentFactoryResolver`, que nos permite instanciar nuestro componente sin instanciar una nueva instancia del componente alerta,  ya que el componente alerta es, como todas las clases de componentes y servicios de Angular, `singleton`, por lo que no debemos instanciarlo de la manera normal de JavaScript (usando el constructor).

Para ello, vamos a incluir en el componente padre un método desde el que usar el factoryResolver

Nuestro primer paso va a ser crear una directiva para que podamos usarla para localizar el lugar del DOM donde vamos a generar nuestro componente

```typescript
\\alert.directive.ts


@Directive({
  selector: '[appAlertDirective]'
})
export class AlertDirective {
  //La directiva nos da acceso directo a una referencia en el lugar del DOM donde se utiliza.
  // Al hacer pública esta propiedad la podemos usar como referencia desde el componente donde se use la directiva.
  constructor(public viewContainerRef: ViewContainerRef){}
  
}

```

Debemos declarar esta directiva en nuestro `app-module` dentro del array de declaraciones para usarla.

```html
\\padre.component.html
<div>
 <!-- <app-alert [message]="error" *ngIf="error" (close)="removeError()"></app-alert> -->
 <ng-template appAlertDirective></ng-template>
</div>
```

Ahora ya tenemos una referencia del DOM (Las referencias locales no funcionan para lo siguiente) por lo que podemos pasar a generar el componente dentro de nuestro componente padre de forma programática.


```typescript
\\padre.component.ts

import { Component, ComponentFactoryResolver } from '@angular/core';
import { Subscription } from 'rxjs';

//Importamos el componente AlertComponent que ya hemos creado anteriormente, que vamos a usar a modo de plantilla
import { AlertComponent } from './alert.component'
import { AlertDirective } from './alert.directive'

@Component({
  selector: 'app-component',
  template: './padre.component.html',
  styleUrls: ['./padre.component.css']
})

export class PadreComponent  implements OnDestroy {

  // Para tener acceso al lugar donde hemos colocado la directiva la 'vigilamos' con un ViewChild, donde encontrará el primer sitio donde se use. Por eso debemos usar Directivas separadas para diferentes componentes dinámicos en un mismo componente.
  @ViewChild(AlertDirective, { static: false}) alertHost: AlertDirective;

  // Propiedad para gestionar una subscripción para cerrarla cuando corresponda
  private closeSubscription: Subscription


  constructor (private componentFactoryResolver: ComponentFactoryResolver){}

  someError() {
    this.showErrorAlert('Ha habido un error');
  }


  private showErrorAlert(message: string) {

	// Creamos un factory del componente Alerta usando un ComponentFactoryResolver
    const alertCmpFactory = this.componentFactoryResolver.resolveComponenteFactory(AlertComponent);

    // Tenemos disponible la referencia al haberla hecho pública en la directiva. Será donde generemos el componente
    const hostViewContainerRef = this.alertHost.viewContainerRef;

    // Primero despejamos la referencia
    hostViewContainerRef.clear();

	// Ahora usamos el método para crear el componente, pero esto en versiones antiguas de Angular (<9) nos puede dar errores por no tener declarada este "componente de entrada", para lo cual debemos agregarlo en nuestro app-module en el array de entryComponents, que es una propiedad del objeto NgModule que vamos a tener que agregar si no tenemos.
	// Dentro del array, al igual que el de declarations o imports, añadimos el nombre del componente: AlertComponent.
	 const alertCmp = hostViewContainerRef.createComponent(alertCmpFactory);

     // Ahora debemos definir los inputs y outputs que ese componente tiene
     
     // Comenzamos por el mensaje 
     alertCmp.message = message;
     
     // Close es un evento, por lo que no hay que definirlo, sino escucharlo, por lo que nos subscribimos a él.
      this.closeSubsciption  = alertCmp.close.subscribe( () => {
		// Nos desubscribimos
		this.closeSubscription.unsubscribe();
        
        // Despejamos el componente creado de nuestra referencia:
        hostViewContainerRef.clear()
      })
  }


 // Por si acaso se cierra el componente nos desubscribimos en NgDestroy
 ngOnDestroy() {
   this.closeSubscription.unsubscribe();
 }

}

```
