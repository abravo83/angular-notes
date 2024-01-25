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


## Configurando `Animations`:  importación directa de la funciones de `Animations` al componente

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


Las animaciones en Angular se configurar en el decorador `@Component`, donde vamos a agregar una propiedad llamada `animations` que va a contener un array. En este array primero vamos a colocar el método `trigger()` que va a contener dos argumentos, un `identificador` que vamos a colocar en el elemento HTML que queremos animar con esta animación, y un array que a su vez va a contener *estados* y *transiciones*.

Los estados van a su vez ser un método `state()` que contiene dos argumentos, el primero es un `string` que define el nombre del estado y el segundo será un método `style()` que contiene un objeto con los estilos en `css` en `camelCase`: (`backgroundColor`) de ese estado.

Luego hay elementos que definirán la duración de las diferentes transiciones entre estados:

```typescript
@Component({
  standalone: true,
  selector: 'app-open-close',
  animations: [
    trigger('openClose', [
      // ...
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

```typescript
@Component({
  selector: 'app-example',
  
})
```