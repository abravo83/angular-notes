
## Dos categorías principales de directivas propias de Angular:

* Directivas estructurales
* Directivas de atributos

Las directivas estructurales se colocan precedidas de un asterisco
```
*ngIf=...
```

La razón del asterisco es para indicarle a Angular que se trata de una directiva estructural

### Directivas Estructurales:

Las principales son ngFor, ngIf y ngSwitch

#### `ngFor`

Nos permite recorrer y crear elementos html por cada elemento de un array que forme parte de la clase del elemento al que pertenece:

```html
<ul>
  <li
  *ngFor="let number of numbers"
   >
  {{number}}
  </li>
</ul>
```
#### `ngIf`

Nos permite mostrar u ocultar elementos de nuestro html en función de la condición que cumpla.

```html
<ul>
  <li
  *ngIf="number % 2 == 0"
   >
  Odd number
  </li>
  <li
  *ngIf="number % 2 != 0"
   >
  Even number
  </li>
</ul>
```

#### `ngSwitch`

Nos permite mostrar elementos en función de distintos valores posibles para una misma variable. En este ejemplo está enlazado con el valor de la propiedad `value`

```html
<div [ngSwitch]="value">
  <p *ngSwitchCase="5">Value is 5<p/>
  <p *ngSwitchCase="10">Value is 10<p/>
  <p *ngSwitchCase="100">Value is 100<p/>
  <p *ngSwitchDefault>Value is Default<p/>
</div>
```



**Las directivas estructurales no se pueden combinar en el mismo elemento. Es decir: sólo puede haber una directiva estructural por elemento HTML**

#### Creación de directivas estructurales propias

Nosotros también podemos crear nuestras propias directivas estructurales:

```bash
ng g d unless
```

En el archivo generado:

```typescript
import { Directive, Input } from '@angular/core';

@Directive({
  selector: '[appUnless]'
})
export class UnlessDirective {
  @Input() set appUnless(condition: boolean) {
    if (!condition) {
      this.viewContainerReference.createEmbeddedView(this.templateRef)
    } else {
      this.viewContainerReference.clear();
    }
  }

  constructor(private templateRef: TemplateRef<any>, private viewContainerReference: ViewContainerReference) {
  
  }
}
```

```html
<div *appUnless="!onlyOdd">
  <li *ngFor="let even of evenNumbers">
    {{even}}
  </li>
</div>
```

Con este código hemos primero creado la directiva estructural, hemos inyectado dos elementos claves de esta directiva estructural:
* `templateRef`: Hace referencia al contenido interno de la directiva estructural (se refiere al `ng-container`)
* `viewContainerReference`: Se refiere al elemento sobre el que se visualizará el `template`

Además, hemos enlazado nuestra directiva con un enlace de datos a la propiedad `unless` de la clase de nuestra directiva y lo hemos hecho definiendo un `setter` por lo que se ejecutará la función de ese setter cada vez que se modifique el valor de `unless`



### Directivas de Atributos

Son directivas que afectan a los atributos de los elementos HTML. Las dos directivas principales de esta categoría son:

+ ngClass
+ ngStyle

Las directivas de atributo se colocan dentro de corchetes:
```
[ngClass]=...
[ngStyle]=...
```


#### ngClass

Nos permite agregar una clase a un elemento HTML si cumple la condición que la directiva establece:

```html
<main>
  <article
    [ngClass]="{className: condition === true }"
    >
  </article>
</main>
```

Otros usos puede se usar un string con varias clases o un array con diferentes clases:

```html
[ngClass] = "'active primary'"
```

```html
[ngClass] = [class1, class2, ...]
```

#### ngStyle

Nos permite añadir un estilo a un componente HTML de forma condicional:

```html
<div>
	<p [ngIf] ="{backgroundColor: saldo < 0 ? 'red' : 'green'}"
	  >
	Tu saldo es {{saldo}}</p>
</div>
```

La sintaxis es similar a la de `ngClass`, pero como aquí estamos agregando estilos de CSS de forma directa tenemos una salvedad:

**No podemos usar `kebab-case`, sino que debemos sustituirlo por `camelCase`**

```
background-color => backgroundColor
text-align => textAlign
...
```

Los valores siguen las reglas de evaluación normal. Los números no necesitan comillas, pero las palabras, al ser `strings`  van entre comillas:

## Directivas propias o Custom

Las directivas propias nos sirven para marcar los elementos HTML y poderles asignar diferentes propiedades e interactividades.

Hay dos formas de crear directivas propias:

* Mediante creación directa
* Mediante uso de Angular CLI

### Creación directa de Directiva:

1. Creamos el archivo y carpeta dentro de la carpeta src/app/ 
con el siguiente nombre: tu-nombre-de-directiva.directive.ts

2. Dentro de ese archivo debemos crear la clase exportable con el nombre de nuestra directiva y precedida del decorador `@Directive`, que debemos importar del `@angular\core`

```typescript
import { Directive } from '@angular/core';

@Directive({
  selector: '[tuNombreDeDirectiva]'
});
export class TuNombreDeDirectiva {
  
}
```
*Los selectores de las directivas siguen las mismas estructuras que las directivas angular directamente aplicadas a los elementos HTML. Cuando son directivas tipo estructural irán precedidas de un asterisco (`*`) y las directivas de tipo atributo van dentro de corchetes, como en el ejemplo ocurre con `[tuNombreDeDirectiva]`*

3.  Para poder inyectar estilos u otra propiedad de atributo debemos usar una referencia al elemento HTML,  lo que vamos a hacer en la función constructora de la clase de nuestra directiva:

```typescript

import { Directive, ElementRef } from '@angular/core';

@Directive ({
  selector: '[tuNombreDeDirectiva]'
})
export class TuNombreDeDirectiva {

  constructor(private elementRef: ElementRef) {}

}
```

Ahora ya podemos referirnos dentro de la clase de nuestra directiva al elemento HTML que queremos modificar al agregar la directiva. Al haber precedido `elementRef` con el modificador `private` hemos conseguido dos cosas:

  1. Lo hemos convertido en una propiedad de esta clase.
  2. Le hemos asignado directamente la referencia HTML del elemento al que le hemos agregado la directiva.


4.  Para poder usar esta referencia dentro de la clase, la podemos ejecutar con el evento de ciclo de vida `ngOnInit` y podemos asignarle el valor que queramos a su estilo u otro atributo, por ejemplo:

```typescript

export class TuNombreDeDirectiva implements OnInit{

  constructor(private elementRef: ElementRef){}

  ngOnInit() {
    this.elementRef.nativeElement.style.backgroundColor = 'red';  
  }
  
}
```
*Así hemos conseguido cambiar el color de fondo de cualquier elemento HTML al que le hemos agregado nuestra directiva*

5. Por último, aunque no menos importante, para poder utilizar nuestra directiva debemos registrarla en el módulo, incluyéndola dentro del array de declaraciones, como tenemos a nuestro a nuestro componentes:

```typescript
\\app.module.ts

import { tuNombreDeDirectiva } from './tu/url/relativa/a/tu/tuNombreDeDirectiva.directive.ts';

@NgModule({
  declarations: [
    ...,
    tuNombreDeDirectiva,
  ],
  imports: [
    ...,
  ],
  providers: [],
  bootstrap: [AppComponent];
})
export class AppModule {}
```


### Creación mediante el uso del CLI de Angular

Mediante el CLI va a ser similar todo al uso, salvo por la creación de los archivos en sí y su registro en AppModule.

1. Usamos el siguiente comando de consola para crear la Directiva:
```shell
ng create directive tuNombreDeDirectiva
```

Con estos realizado dos cosas, la creación de archivo y carpetas donde estará nuestra directiva. Su registro en `AppModule` y la generación básica del código de la directiva:

```typescript
import { Directive } from '@angular/core';

@Directive({
  selector: '[tuNombreDeDirectiva]'
});
export class TuNombreDeDirectiva {
  
}
```

A partir de aquí ya somos nosotros los que debemos preparar la inyección de la referencia en la directiva (paso 3) y su uso en el evento de ciclo de vida `OnInit` (paso 4)



### Uso interactivo de Directivas propias

El uso que hemos visto en los ejemplos era muy estático, ya que habíamos establecido propiedades fijas. Podemos ampliar este uso apoyándonos de algunos decoradores extra y otros mecanismos que nos van a permitir acciones dinámicas.

##### `@HostListener` 

Nos permite escuchar determinados eventos del elemento que contiene la directiva. Permite que podamos asignar propiedades y ejecutar métodos al producirse dicho evento.

```typescript

import { Directive, HostListener, OnInit, ElementRef } from '@angular/core';

@Directive({
  selector: 'tuNombreDeDirectiva'
})
export class TuNombreDeDirectiva implements OnInit {


  constructor(private elementRef: ElementRef,private renderer: RendererV2){}

  ngOnInit(){
    @HostListener('mouseenter') mouseover(eventData: Event) {
      this.renderer.setStyle(this.elementRef.nativeElement, 'background-color', 'blue', false, false);
    }
    
    @HostListener('mouseleave') mouseleave(eventData: Event) {
      this.renderer.setStyle(this.elementRef.nativeElement, 'background-color', 'red', false, )
    }

  }


}
```



##### `@HostBinding`

Nos permite enlazar directamente una propiedad de la clase de la directiva con el valor de cualquier atributo del HTML de nuestra directiva y reaccionar a dichos valores así como modificarlos.

En sí puede funcionar como un repuesto del método anterior de usar la combinación de `@HostListener` con `Renderer` para modificar los valores de los atributos de un elemento HTML. En este caso sólo tenemos que modificar el valor la propia propiedad de la Directiva y, como  está enlazada con el atributo del elemento HTML, al modificar la propiedad modificamos el valor del atributo

Un ejemplo similar al anterior pero usando `@HostBinding` sin usar `Renderer`:
```typescript
@Directive({
  selector: '[appChangeBackgroundOnHover]'
})
export class ChangeBackgroundOnHover implements OnInit {

  @HostBinding('style.backgroundColor') bckGrndColor: string = 'transparent';

  @HostListener('mouseenter') mouseover(eventData: Event) {
    this.bckGrndColor = 'blue';
  }

  @HostListener('mouseleave') mouseleave(eventData: Event) {
    this.bckGrndColor = 'transparent';
  }

}
```


##### Combinar `@HostBindin` y `@HostListener` con `Property Binding`

Podemos utilizar la técnica anterior para hacer que se puedan fijar los valores del elemento HTML desde el propio elemento HTML:

```html
<p appChangeBackgroundOnHover [defaultColor]='transparent' [highlightColor]='yellow' > This is a test text</p>
```

```typescript
@Directive({
  selector: '[appChangeBackgroundOnHover]'
})
export class ChangeBackgroundOnHover {

  @Input() defaultColor: string = 'transparent';
  @Input() highlightColor: string = 'blue';
  @HostBinding('style.backgroundColor') bckGrndColor: string;

  ngOnInit(){
    this.bckGrndColor = this.defaultColor
  }

  @HostListener('mouseenter') mouseover(eventData: Event){
    // Lo fijamos aquí porque antes no tenemos el valor de highligthColor disponible.
    this.bckGrndColor = this.highlightColor;
  }
  
  @HostListener('mouseleave') mouseleave(eventData: Event){
    this.bckGrndColor = this.defaultColor;
  }
  

}
```


