Angular dispone de una librería propia para aplicar animaciones que funciona mediante modificaciones de estado de componentes y que nos permite aplicar animaciones desde Angular sin tener que manipular el DOM

## Configurando `Animations` en Angular 17 `Standalone`

Para poder trabajar con animaciones en Angular 17 debemos añadir a nuestro método `bootstrapApplication()`, que generalmente está en `main.ts` a nuestro array de `providers`:

```typescript
//...
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async'


bootstrapApplication(AppComponent, {
  providers: [
    provideAnimationsAsync(),
  ]
});
```

Pero si queremos tener una animación inmediatamente se carga nuestra aplicación debemos cambiar de `provider` a `provideAnimations()` de `@angular/platform-browser/animations`, que se carga de forma inmediata. 

## Configurando `Animations` cuando usamos `NgModules`

Entonces debemos de importar `BrowserAnimationsModule` al array de `imports` del decorador `@NgModule`


```typescript
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';

@NgModule({
  imports: [
    BrowserModule,
    BrowserAnimationsModule
  ],
  declarations: [ ],
  bootstrap: [ ]
})

export class AppModule { }
```


## Configurando `Animations` en el componente:  importación de las funciones de `Animations`

Esta parte es común tanto para componentes `standalone` como componentes para `NgModules`. Para poder usar las animaciones de Angular debemos importar al componente los métodos con los que vamos a definir los metadatos del decorador para configurar nuestra animación.

```typescript
import { Component, HostBinding } from '@angular/core';
import {
  trigger,
  state,
  style,
  animate,
  transition,
  // ...
} from '@angular/animations';
```


## Uso de `Animations` en el componente

Una vez tenemos todo configurado para poder usar las animaciones, debemos configurar cada una de nuestras animaciones en el propio componente.

Las animaciones en Angular se configurar en el decorador `@Component`, donde vamos a agregar una propiedad llamada `animations` que va a contener un array. En este array primero vamos a colocar el método `trigger()` que va a contener dos argumentos, un `identificador` que vamos a colocar en el elemento HTML que queremos animar con esta animación, y un array que a su vez va a contener *estados* y *transiciones*.

Los estados van a su vez ser un método `state()` que contiene dos argumentos, el primero es un `string` que define el nombre del estado y el segundo será un método `style()` que contiene un objeto con los estilos en `css` en `camelCase`: (`backgroundColor`) de ese estado.

Luego hay elementos que definirán la duración de las diferentes transiciones entre estados:

```typescript
@Component({
  standalone: true,
  selector: 'app-open-close',
  animations: [
    trigger('openClose', [
      state('open', style({
        height: '200px',
        opacity: 1,
        backgroundColor: 'yellow'
      })),
      state('closed', style({
        height: '100px',
        opacity: 0.8,
        backgroundColor: 'blue'
      })),
      transition('open => closed', [
        animate('1s')
      ]),
      transition('closed => open', [
        animate('0.5s')
      ]),
...
    ]),
  ],
  templateUrl: 'open-close.component.html',
  styleUrls: ['open-close.component.css']
})
export class OpenCloseComponent {
...
  isOpen = true;
  toggle() {
    this.isOpen = !this.isOpen;
  }
...
}
```

Luego en la plantilla podemos definir el estado de un componente basándonos en algún condicional. En el ejemplo evaluamos si la variable  `isOpen` es verdadera o falsa

```html
<div [@openClose]="isOpen ? 'open' : 'close' ">
  <p>La caja está {{isOpen ? 'abierta' : 'cerrada'}}</p>
</div>
```

Otra manera en que podemos usarla es colocando directamente alguna variable que contenga el valor del estado. Por ejemplo, puede comenzar en estado `state='normal'` como valor inicial y luego pasarlo a `state='open'` para después pasarlo a `state='close'` en nuestro componente.

```html
<div [@openClose]="state">
  <p>La caja está {{isOpen ? 'abierta' : 'cerrada'}}</p>
</div>
```

### Uso de `Alias` en el componente.

Tenemos disponible el uso de algunos alias a usar en vez de los cambios de estado que nos permiten definir animaciones a aplicar cuando un `@if `o `*ngIf` activan un elemento. No definen estados, sino directamente las transiciones, donde la transición para `:enter` acepta como primer elemento del array un método `style({cssProp: cssValue})` que define el estilo inicial desde la que parte la primera animación de entrada.

```typescript
@COmponent({
  selector: 'app-mi-ejemplo',
  templateUrl: './mi-ejemplo.component.html',
  styleUrls: ['./mi-ejemplo.component.css'],
  animations: [
    trigger('apareceDesaparece', [
      transition(':enter', [
        style({height: '0px', opacity: 0}),
        animate('0.5s', style({height: '200px', opacity: 1 }))
      ]),
      transition(':leave'), [
      animate('0.5s', style({height: '0px', opacity: 0}))
      ]
    ])
  ]
})
export class MiEjemplo {
  isOpen = false;
  toggle(){
    this.isOpen = !this.isOpen;
  }

}
```

```html
@if(isOpen){
<!-- No lleva binding al estado porque usa alias -->
<div @apareceDesaparece>
  <p>La caja está {{isOpen ? 'abierta' : 'cerrada'}}</p>
</div>
}
```


Otro alias que tenemos es `void` que identifica el estado de un elemento cuando no está presente en el DOM, por lo que hace algo similar a `:enter`, pero podemos definir la animación no para toda aparición del elemento, sino específicamente para cuando aparece hacia un estado o hacia otro:

```typescript
@Component({
  selector: 'app-example',
  templateUrl: './example.component.html',
  styleUrls: ['./example.component.css'],
  animations: [
  trigger('aparece', [
    state('normal', style({backgroundColor: 'green'})),
    state('yello', style({backgroundColor: 'yellow'})),
    transition('void => normal', animate(300)),
    transition('void => yello', animate(600)),  // La duración es diferente
  ],
})
```

## Transiciones

Ya hemos visto un poco cómo funciona el método `transitions` en el que primero indicábamos la dirección de la transición a la que queremos aplicar la animación y la duración de dicha animación:

```typescript
transition('open => closed', animate('1s'))
```

Pero hay más opciones. Podemos definir ambas direcciones para una transición:

```typescript
transition('open <=> closed', animate('1s'))
```

O podemos hacer una animación en varias fases usando un array como segundo argumento:

```typescript
transition('open <=> closed', [
  style({'background-color' : 'orange' }), //Primera fase: cambio brusco de color de fondo.
  animate(500, style({backgroundColor : 'green')}), //Segunda fase: cambio suave a verde
  animate(500, style({transform : 'scale(0.5)')), //Tercera fase: reducimos tamaño a la mitad.
  ])
```

