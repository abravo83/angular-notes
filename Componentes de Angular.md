Toda aplicación Angular contiene al menos un componente, el **componente raíz**, que se conecta a una jerarquía de componentes con el DOM o *document object model*. Cada componente define un clase que contiene la lógica y los datos de la aplicación, y asociada a esta clase hay un plantilla HTML que define la vista que se mostrará en el destino.

El decorador `@Component()` identifica a la clase inmediatamente siguiente como `componente` y provee la plantilla y meta-datos específicos de ese componente.

Además de la clase y el decorador los componentes consisten en:

* Un plantilla HTML que define la vista
* Una clase de Typescript que define el comportamiento lógico del componente
* Un selector que define el nombre que nuestro componente tendrá dentro de otra plantilla. (A este selector se le denomina también **SELECTOR CSS**)
* Opcionalmente el estilo o estilos CSS a aplicar a la plantilla

## Formas de crear componentes en Angular

Los componentes en Angular se pueden crear de dos formas distintas, pero para poder crearlos necesitamos un paso previo: **[[Crear o inicializar un proyecto Angular]]**

### Crear un componente usando Angular CLI

Usaremos el siguiente componente desde la terminal situados en la carpeta de nuestro proyecto:
```shell
ng generate component <nombre-del-componente>
```
Alternativamente tenemos una versión reducida:
```shell
ng g c <nombre-del-componente>
```

### Crear componentes manualmente

Para crear un componente de manera manual, nos situaremos de nuevo en la carpeta de nuestro proyecto y a partir de entonces:
1.  Creamos un nuevo archivo con en nombre `<nombre-del-componente.component.ts>`
2.  En la parte superior del archivo creado incluimos el comando de importación:
```typescript
import { Component } from '@angular/core';
```
3. Tras la importación, en línea aparte, añadimos el decorador `@Component`:
```typescript
@Component({
})
```
4. Añadimos el selector de nuestro componente:
```typescript
@Component({
  selector: 'componente-recien-creado'
})
```
5. Definimos la plantilla usando en este caso un archivo externo:
```typescript
@Component({
  selector: 'componente-recien-creado'.
  templayeUrl: './nombre-del-componente.component.html'
})
```
6. Seleccionamos los estilos de la plantilla del componente. En este caso también usaremos un archivo externo:
```typescript
@Component({
  selector: 'componente-recien-creado',
  templateUrl: './nombre-del-componente.component.html',
  styleUrls: ['./nombre-del-componente.component.css']
})
```
  hay que fijarse que `styleUrls` tiene una *s* al final, y que usamos un *Array* para definir el archivo de estilos **o archivos**, separados por coma

7. Por último, añadimos la clase **exportable** en una línea inferior, pero sin nada más entre medias salvo espacios en blanco
```typescript
export class ComponentOverviewComponent {

}
```
## Ciclo de vida de un componente Angular

Todo componente de Angular sigue un ciclo de vida que se inicia cuando la clase es instanciada y se renderiza el componente junto con sus componentes hijos. Posteriormente el ciclo continúa con la detección de cambios, ya que Angular comprueba cuando han cambiado las propiedades que están asociadas a la lógica de la clase del componente y actualiza tanto la vista como la lógica del componente. Finalmente, el ciclo acaba cuando se destruye el componente y se elimina su renderizado del DOM.

Nuestra aplicación puede usar métodos enlazados al ciclo de vida para que dichos métodos sean ejecutados cuando se llega a un determinado punto del ciclo de vida.

La **[[directrices de Angular]]**  siguen un ciclo de vida similar.

Vamos a ver un ejemplo de uso de un evento de ciclo de vida `OnInit` en un componente:

```typescript
imports { Component, OnInit } from '@angular/core';

@Component({
  selector: 'my-component',
  templateURL: './my-component.component.html',
  styleUrls: ['./my-component.component.css'] 
})
export class MyComponent implements OnInit {

  constructor (private log: Array<string>) {}

  //Implementamos el método OnInit para que se registre el primer log en el ciclo de vida.
  ngOnInit() {
    this.log.push('Iniciando aplicación');
  }

}
```

Este es el orden de ejecución de los distintos eventos de ciclo de vida disponible:

	El constructor, aunque no sea un método de ciclo de vida, es previo a todos ellos.

* ngOnChanges() | todos los ciclos en los que se activa | cuando detecta cambios
* ngOnInit() | una vez | siempre
* ngDoCheck() | todos los ciclos en que se activa | cuando detecta cambios
* ngAfterContentInit() | una vez | siempre
* ngAfterContentCheck() | todos los ciclos en que se activa | cuando detecta cambios
* ngAfterViewChecked()  | todos los ciclos en que se activa | cuando detecta cambios
* ngOnDestroy() | una vez | cuando se destruye el componente (o directiva)

### Uso del constructor y de ngOnInit()

La documentación de Angular aconseja que la construcción e instanciación de un componente debe ser poco costosa, ligera. Es por ello que se recomienda dejar tareas complejas de inicio de un componente para el método `ngOnInit` , como por ejemplo, hacer peticiones de datos.

El método constructor debe queda relegado a declarar las variables locales y fijarles valores sencillos. Los enlaces de datos con las plantillas no ocurren hasta después de la construcción, así que toda asignación de variables que venga de un enlace de datos de un campo de la plantilla debe quedar emplazada a un `ngOnInit()`

### Uso del método ngOnDestroy()

El método `ngOnDestroy()` es donde debemos colocar la lógica que libere de los recursos utilizados por nuestro componente y que no vamos a necesitar más. Ejemplos serían:

* Darnos de baja de Observables y eventos del DOM
* Parar intervalos
* Dar de baja todos los `callbacks` que una directiva tenga registrada con servicios globales o de la aplicación.


## Interacción entre componentes de Angular

Nos referimos a la capacidad de pasar datos de un componente padre a otro hijo usando `input binding`, y también a la capacidad de interceptar cambios de propiedad mediante un `input property setter`.

### Pasar datos de un componente padre a otro hijo usando input binding

Tenemos un componente padre:
```typescript
import { Component } from '@angular/core';

import { HEROES } from './hero';

@Component({
  selector: 'app-hero-parent',
  template: `
    <h2>{{master}} controls {{heroes.length}} heroes</h2>
    
    <app-hero-child
      *ngFor="let hero of heroes"
      [hero]="hero"
      [master]="master"
    >
    </app-hero-child>
  `
})
export class HeroParentComponent {
  heroes = HEROES;
  master = 'Master';
}
```

Y su componente hijo:

```typescript
import { Component, Input } from '@angular/core'

import { Hero } from './hero';

@Component({
  selector: 'app-hero-child',
  template: `
  <h3>{{hero.name}} says:</h3>
  <p>I, {{hero.name}}, am at your service, {{masterName}}</p>
  `
})
export class HeroChildComponent {
  @Input() hero!: Hero;
  @Input('master') masterName = ''; 
}
```

### Interceptar cambios de propiedad mediante un `input property setter`

Vamos a fijar un `setter` para la propiedad `name` que elimine el espacio en blanco y que si detecta que la propiedad `name` está vacía le ponga algún texto por defecto:

```typescript
import { Component, Input } from '@angular/core'

@Component({
  selector: 'app-name-child',
  template: `
    <h3>"{{name}}"</h3>
  `
})
export class NameChildComponet {
  @Input()
  get name(): string { return this._name; }
  set name(name: string){
    this._name = (name && name.trim()) || '<no name set>';
  }
  private _name = '';
}
```

Y ahora, vamos a crear su componente padre donde pasamos diferentes valores para `name`:

```typescript
import { Component } from '@angular/core'

@Component({
  selector: 'app-name-parent',
  template: `
    <h2>Master controls {{names.length}} names</h2>
	
	<app-name-child *ngFor="let name of names" [name]="name"></app-name-child>
  `
})
export class NameParentComponent {
  // Muestra 'Dr. IQ', '<no name set>', 'Bombasto'
  names = ['Dr. IQ', '', 'Bombasto'];
}
```



### Interceptar cambios de propiedad mediante `ngOnChanges()`

Podemos usar el método de ciclo de vida `ngOnChanges()` para detectar y actuar según los cambios producidos en los valores de la propiedades `input`

Esta forma de interactuar con distintos valores `input` es quizás preferible a usar `setters` y `getters` cuando estamos vigilando varios valores `input`

Vamos a ver un ejemplo de componente hijo en que que detectamos cambios a las propiedades `major` y `minor` y construimos un mensaje de log informando sobre estos cambios:

```typescript
import { Component, Input, OnChanges, SimpleChanges } from '@angular/core'

@Component({
  selector: 'app-version-child',
  template: `
    <h3>Version {{major}}.{{minor}}</h3>
    <h4>Change log:</h4>
	<ul>
	  <li *ngFor="let change of changeLog">{{change}}</li>
	</ul>    
  `
})
export class VersionChildComponent {

  @Input() major = 0;
  @Input() minor = 0;
  changeLog: string[] = [];

  ngOnChanges(changes: SimpleChanges) {
    const log: string[] = [];
    for (const propName in changes) {
      const changedProp = changes[propName];
      const to = JSON.stringify(changedProp.currentValue);
      if (changedProp.isFirstChange()) {
        log.push(`Initial value of ${propName} set to ${to}`);
      } else {
        const from = JSON.stringify(changedProp.previousValue);
        log.push(`${propName} changed from ${from} to ${to}`)
      }
    }
  }
}

```

El componente padre, por su parte, aporta los valores de `major` y `minor`, y enlaza métodos a botones que alteran sus valores:

```typescript
import { Component } from '@angular/core'

@Component({
  selector: 'app-version-parent',
  template: `
    <h2>Source code version</h2>
    <button type="button" (click)="newMinor()">New minor version</button>
    <button type="button" (click)="newMajor()">New major version</button>
    <app-version-child [major]="major" [minor]="minor"></app-version-child>
  `
})
export class VersionParentComponent {
  major = 1;
  minor = 23;

  newMinor(){
    this.minor++;
  }

  newMajor(){
    this.major++;
    this.minor = 0;
  }
}
```





### Un componente padre que escucha eventos de un componente hijo

El componente hijo expone una propiedad `EventEmitter` con la que puede hacer eventos `emits` cuando algo ocurre. El componente padre se enlaza a esa propiedad evento y reacciona a esos eventos.

La propiedad `EventEmitter` del hijo es una *propiedad de salida* , adornada típicamente con el decorador `@Output` , como podemos ver en el componente `VoterComponent`:

```typescript
import { Component } from '@angular/core'

@Component({
  selector: 'app-voter',
  template: `
    <h4>{{name}}</h4>
    <button type="button" (click)="vote(true)" [disabled]="didVote">Agree</button>
    <button type="button" (click)="vote(false)" [disabled]="didVote">Disagree</button>
  `
})
export class VoterComponent {
  @Input() name = '';
  @Output() voted = new EventEmitter<boolean>();

  didVote = false;
  
  vote(agreed: boolean) {
   this.voted.emit(agreed);
   this.didVote = true;
  }
}
```

Al hacer click en el botón `Agree` o `Disagree` disparamos un evento que emite consigo el valor del argumento boleano que le pasamos al ejecutar el método enlazado a los botones.

Por su parte, el componente padre va a enlazar un método manejador de eventos: `onVoted()` que responde a la emisión del evento del componente hijo y actualiza un contador según el valor emitido:

```typescript
import { Component } from '@angular/core'

@Component({
  selector: 'app-vote-taker',
  template: `
    <h2>Should mankind colonize de Universe?</h2>
    <h3>Agree: {{agreed}}, Disagree: {{disagreed}}</h3>

	<app-voter
	  *ngFor="let voter of voters"
	  [name]="voter"
	  (voted)="onVoted($event)"
	  >
	</app-voter>
  `
})
export class VoteTakerComponent {
  agreed = 0;
  disagreed = 0;
  voters = ['Dr. IQ', 'Celeritas', 'Bombasto'];

  onVoted(agreed: boolean) {
    if (agreed) {
      this.agreed++
    } else {
      this.disagreed++
    }
  }
}
```

## Encapsulación de componentes en Angular
#Encapsulacion-de-componentes-en-Angular

Los componentes pueden ser encapsulados dentro del componente anfitrión para que así no afecten al resto de la aplicación.

La opción de encapsulación se configura dentro del decorador `@Component()` 

```typescript
@Component({
  selector: 'app-no-encapsulation',
  template: `
    <h2>None</h2>
    <div class="none-message">No encapsulation</div>
  `,
  styles: ['h2, .none-message { color: red }'],
  encapsulation: ViewEncapsulation.None,
})
```

En el ejemplo hemos fijado la encapsulación como `None`, es decir: *ninguna* . Tenemos las siguientes opciones:

* ` ViewEncapsulation.Emulated `
* ` ViewEncapsulation.None `
* ` ViewEncapsulation.ShadowDom `

### ViewEncapsulation.None

Le decimos a Angular que no aplique ningún tipo de encapsulación, lo que hará que cualquier estilo definido en el componente estará disponible de forma global para todos los componentes de la aplicación. Es el comportamiento normal que seguiría incluir los estilos directamente en el HTML.

### ViewEncapsulation.ShadowDom

Utilizamos la API `Shadow DOM` para contener la vista del componente dentro del `ShadowRoot`, que usamos como anfitrión del componente, aplicándole los estilos de forma aislada.


### ViewEncapsulation.Emulated

Angular modifica los selectores CSS de los componentes para que sólo se apliquen a la vista del componente y no afecten a otros elementos de la aplicación, emulando el comportamiento de `Shadow DOM`.

Cuando vemos nuestra plantilla veremos que los elementos HTML del componente con `ViewEncapsulation.Emulated` tienen atributos adicionales que nosotros no añadimos, tales como `_nghost` y `_ngcontent` según los elementos sean padres o hijos, y algún sufijo para poder realizar la encapsulación entre distintos componentes con encapsulación habilitada.


### Demostraciones de uso

Un ejemplo con encapsulación emulada:

```typescript
@Component({
  selector: 'app-emulated-encapsulation',
  template: `
    <h2>Emulated</h2>
    <div class="emulated-message">Emulated encapsulation</div>
    <app-no-encapsulation></app-no-encapsulation>
  `,
  styles: ['h2, .emulated-message { color: green; }'],
  encapsulation: ViewEncapsulation.Emulated,
})
export class EmulatedEncapsulationComponent { }
```

![[EncapsulationCapture1.png]]

En nuestro ejemplo, le habíamos añadido el componente `<app-no-encapsulation>` que es un componente sin encapsulación. El comportamiento de la encapsulación emulada es que los estilos de nuestro componente no se han propagado al otro, a pesar de que ambos componentes usan `<h2>` , por ejemplo.

Si ambos elementos estuvieran dentro de un elemento padre con `ViewEncapsulation.ShadowDom` , vamos a ver un ejemplo de código y su aspecto final:

```typescript
@Componente({
  selector: 'app-shadow-dom-encapsulation',
  template: `
    <h2>ShadowDom</h2>
    <div class="shadow-message">Shadow DOM encapsulation</div>
    <app-emulated-encapsulation></app-emulated-encapsulation>
  `,
  styles: ['h2, .shadow-message { color: blue; }'],
  encapsulation: ViewEncapsulation.ShadowDom,
})
export class ShadowDomEncapsulationComponent { }
```

![[EncapsulationCapture2.png]]

Como vemos, los `<h2>` del padre son azules, los del hijo con encapsulación emulada son verdes, mientras que los del nieto (duplicado) sin encapsulación han heredado los estilos del componente con encapsulación con `ShadowDom`, pero no los del componente con encapsulación `Emulated`.
## Uso de estilos en los componentes de Angular

Los estilos en los componentes en Angular se pueden definir de varias formas. Una es usando la propiedad `styles` en el decorador, donde podemos definir el código CSS directamente:

```typescript
import { Component } from '@angular/core'

@Component({
  selector: 'app-root',
  template: `
    <h1>Tour of Heroes</h1>
    <app-hero-main [hero]="hero"></app-hero-main>
  `,
  styles: ['h1 { font-weight: normal}']
})
export class HeroAppComponent {
/* */
}
```

### Mejores prácticas al aplicar estilos a los componentes

Debes considerar los estilos de un componente como la implementación privada de los detalles de ese componente. Cuando estemos usando un componente común no debemos sobrescribir los estilos del componente igual que no debemos acceder a la miembros privados de una clase en `Typescript`

Mientras que la encapsulación de estilos que se aplica por defecto previene que los estilos de otros componentes acaben afectando a otros componentes, los estilos globales afectan a todos los componentes de una página. Esto incluye `::ng-deep` que promueve el estilo de un componente a estilo global.

### Crear un componente que permita la personalización

Como autor de un componente, puedes diseñar un componente