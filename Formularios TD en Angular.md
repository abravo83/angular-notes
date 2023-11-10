
## ¿Por qué es necesario utilizar herramientas de Angular para usar formularios?

Los formularios tradicionales, a través de las etiquetas `html` típicas:

```html
<form>
  <label>Nombre</label>
  <input type="text" name="nombre" />
  <label>Email</label>
  <input type="email" name="email" />
  <button type="submit">Enviar</button>
</form>
```

Pero vamos a necesitar Angular para recuperar esos datos, enviarlos a nuestra lógica para validarlos y para enviarlos donde necesitamos, por lo que no nos va a servir este formulario tradicional con un `action` y un método `post`.

Necesitaremos que se cree un `objeto  formulario` donde se aglutinen los valores del formulario. 

Algo así:

```typescript
{
  value:{
    nombre: 'Pedro',
    email: 'pedro.gomizample@esunaprueba.com'
  }
  valid: true
}
```

Y esto es lo que los formularios de Angular nos va a permitir.

## ¿Qué es un Formulario `Template-Driven`?

Es un formulario donde Angular infiere el Objeto Formulario a partir del DOM.

Es una de las dos opciones principales que tenemos para crear formulario usando Angular. La otra opción es un `Reactive Form` o **Formulario Reactivo** en el que el formulario es creado de forma programática y sincronizado con el DOM.

Esta segunda opción nos da más control sobre el formulario, pero resulta un poco más compleja de configurar.


## ¿Cómo es un Formulario TD? ¿Cómo usarlo?

Para poder utilizarlo primero debemos importar el módulo `FormsModule` desde `'@angular/forms'` a nuestro `app.module.ts`

```typescript
import { FormsModule } from '@angular/forms'
//...
@NgModule({
  //...
  imports: [
    //..,
    FormsModule,
    //...
  ],
  //...
})
export class AppModule { }
```

Una vez que tenemos este módulo importado, `FormsModule` creará para nosotros un objeto formulario cuando detecte la etiqueta `<form>` en nuestra plantilla.

Sin embargo, somos nosotros los que debemos indicar la situación de los `inputs` que queremos agregar a ese objeto formulario.
Lo vamos a hacer agregando a nuestra etiqueta `input` la directiva `ngModel` manteniendo las propiedades `name="nombreDeVariable"` que serán las que identifiquen las propiedades de los valores de nuestro *objeto formulario*

```html
<form>
  <label>Nombre<label>
  <input 
    type="text" 
    id="nombre-usuario" 
    name="nombre" 
    ngModel
  />
  <button type="submit">Enviar<button>
</form>
```

Para poder enviar y utilizar los datos necesitamos crear un método para usarlo al enviarlo.

```typescript
  onSubmit(){
  }
```

El mejor lugar para usar el método en la plantilla NO es un botón de tipo `submit` porque este botón va a hacer que se dispare el evento de Javascript `submit`.

Hay una directiva que usa Angular para escuchar este evento, que se coloca en la etiqueta de apertura `<form>` llamada `(onSubmit)`:

```html
<form (ngSubmit)="onSubmit()">
  <label>Nombre<label>
  <input 
    type="text" 
    id="nombre-usuario" 
    name="nombre" 
    ngModel
  />
  <button type="submit">Enviar<button>
</form>
```
 
 Para pasarle los valores se puede usar una referencia local, que vamos a enlazar con la directiva "ngForm" del elemento `<form>` y vamos a pasársela como argumento a nuestro método:
```html
<form (ngSUbmit)="onSubmit(aform)" #aform="ngForm">
```

```typescript
onSubmit(form: NgForm){
  console.log(form.form.value);
}
```

<hr>

También podríamos haber accedido a la referencia local usando `@ViewChild`:

```html
<form (ngSUbmit)="onSubmit()" #aform="ngForm">
```

```typescript
@ViewChild('aform') aform!: Ngform;  //La ! es porque no le vamos a dar un valor inicial, para que Typescript ignore ese error.

onSubmit(){
  console.log(this.aform.form.value);
}
```

*Fíjate que ahora `onSubmit()` no tiene argumentos, estoy usando directamente la referencia local sin pasarla al método como argumento*

**Esta forma de acceder a los datos del formulario nos puede ser muy útil para, por ejemplo, tener habilitado o deshabilitado el botón Enviar, validaciones en tiempo real, etc.**

<hr>

## Formas de validar nuestro Formulario TD


Podemos añadir directivas de validación a nuestro HTML, que aunque sean nativas de HTML, Angular va a usar:

* required
* email
* ...

```html
<form (ngSubmit)="onSubmit()">
  <label>Nombre<label>
  <input 
    type="text" 
    id="nombre-usuario" 
    name="nombre" 
    ngModel
    required
  />
  <input 
    type="email" 
    id="email-usuario" 
    name="email" 
    ngModel
    required
    email
  />
  <button type="submit">Enviar<button>
</form>
```

Estas directivas harán que nuestro elemento `NgForm` devuelva en su atributo `valid=false` si no se cumplen las validaciones nativas que le hemos agregado, como estar vacíos, o no cumplir el formato de email.

Angular, además, va a renderizar el HTML agregándole clases según el estado de validación. Si observamos con el inspector el elemento input podremos ver clases como:

```html
<input class="... ng-dirty ng-touched ng-valid"
```

Que nos pueden permitir adaptar estilos cuando un input "tiene contenido", ha sido "tocado", o "es válido" según las directivas de control que le hayamos agregado. 

## Ejemplos de uso de la validación

Podemos aprovechar las validaciones para evaluar si el formulario está listo para ser enviado o no usando la propiedad `disabled` enlazada con la cualidad `valid` del formulario

```html
<form (ngSubmit)="onSubmit()" #aform="NgForm">
  <label>Nombre<label>
  <input 
    type="text" 
    id="nombre-usuario" 
    name="nombre" 
    ngModel
    required
  />
  <input 
    type="email" 
    id="email-usuario" 
    name="email" 
    ngModel
    required
    email
  />
  <button type="submit" [disabled]="!aform.valid">Enviar<button>
</form>
```

También podemos usar las clases que Angular genera para nosotros (`ng-dirty`, `ng-invalid`, etc)

```css
input.ng-invalid.ng-touched {
  border: 1px solid red;
}
```


O incluso hacer que se muestren mensajes de error cuando no se han agregado los datos en formato correcto.

```html
<form (ngSubmit)="onSubmit()" #aform="NgForm">
  <label>Nombre<label>
  <input 
    type="text" 
    id="nombre-usuario" 
    name="nombre" 
    ngModel
    required
  />
  <input 
    type="email" 
    id="email-usuario" 
    name="email" 
    ngModel
    required
    email
    #aemail="ngModel"
  />
  <p *ngIf="!aemail.valid && aemail.touched">Por favor introduce un email correcto</p>
  <button type="submit" [disabled]="!aform.valid">Enviar<button>
</form>
```


## Fijar valores por defecto en el formulario

Si en vez de utilizar `ngModel` como una directiva más lo usamos con un `property binding` y le damos un valor estaremos definiendo el valor por defecto que queremos mostrar en nuestro campo del input o del `<select>`

```html

<form (ngSubmit)="onSubmit()" #aform="NgForm">
  <label>Nombre<label>
  <input 
    type="text" 
    id="nombre-usuario" 
    name="nombre" 
    [ngModel]="Tu nombre aquí"
    required
  />
  <label>Email</label>
  <input 
    type="email" 
    id="email-usuario" 
    name="email" 
    [ngModel]="tucorreo@elquesea.com"
    required
    email
    #aemail="ngModel"
  />
  <label>Elige una pregunta secreta</label>
  <select
    id="secret"
    [ngModel]="pet"
    name="secret"
    >
    <option value="mascota">¿Cuál era el nombre de tu primera mascota?</option>
    <option value="coche">¿Cuál era la marca de tu primer coche?</option>
  </select>
  <p *ngIf="!aemail.valid && aemail.touched">Por favor introduce un email correcto</p>
  <button type="submit" [disabled]="!aform.valid">Enviar<button>
</form>

```

## Uso de `two way binding`

Por supuesto podemos usar también `two way binding` (`([ngModel])`) para mostrar la variable de forma interpolada en la plantilla, aunque entonces deberemos también definir la propiedad en nuestra clase del componente.


## Agrupación de controles del formulario en un `Form Group`

Podemos agrupar una serie de controles para poder hacer una validación general de todo el grupo, por ejemplo, o por mantener todos los valores agrupados. Para eso podemos usar la directiva `ngModelGroup="stringDefinidaPorMi"` en un elemento padre de los subelementos donde se encuentran los `<inputs>` que queremos agrupar

```html

<form (ngSubmit)="onSubmit()" #aform="NgForm">
  <div id="datos-usuario" ngModelGroup="datosUsuario">
   <label>Nombre<label>
    <input type="text" id="nombre-usuario" name="nombre" [ngModel]="Tu nombre aquí" required
  />
    <label>Email</label>
    <input type="email" id="email-usuario" name="email" [ngModel]="tucorreo@elquesea.com" required email #aemail="ngModel"/>
  </div>
  <label>Elige una pregunta secreta</label>
  <select id="secret"[ngModel]="pet"name="secret">
    <option value="mascota">¿Cuál era el nombre de tu primera mascota?</option>
    <option value="coche">¿Cuál era la marca de tu primer coche?</option>
  </select>
  <p *ngIf="!aemail.valid && aemail.touched">Por favor introduce un email correcto</p>
  <button type="submit" [disabled]="!aform.valid">Enviar<button>
</form>

```

Esto hará que los datos en nuestro objeto formulario se encuentren ahora en diferentes propiedades agrupados bajo el nombre `datosUsuario`

```javascript
NgForm = {
  ...
  controls: {
    ...
    datosUsuario: {
      ...
      dirty: true,
      valid: true,
      
    }
  }
  ...
  value: {
    ...
    ...
    datosUsuario: {
      nombre: "...",
      email: "..."
    }
  }
}
```

## Usando los valores del Formulario TD


### Modificando los valores del formulario

Podemos usar los datos de distintas formas. O bien, usamos los datos haciendo uso de los enlaces de doble vía ( `two ways data-binding` ) con el ya conocido `([ngModel])` o podemos usar la referencia local que hemos creado anteriormente en nuestra etiqueta `<form>` y usarla ( a través de `@ViewChild()`) .

```html
<form (ngSubmit)="onSubmit()" #aform="NgForm">
  <div id="datos-usuario" ngModelGroup="datosUsuario">
  </div>
  <button (click)="proponerNombre()">Sugerir datos</button>
...

...
</form>
```

```typescript
export class AppComponent{
 @ViewChild('aform') formulario: NgForm;
  ...
  proponerNombre(){
    const nombreSugerido: 'MrRoot';
    this.formulario.setValue({
      datosUsuario: {
        nombre: nombreSugerido,
        email: '',
        secret: 'coche',
      }
    })
  }
}
```

En el ejemplo hemos usado la referencia para modificar los datos del formulario al hacer click en el botón `Sugerir Datos` que ejecuta el método `proponerNombre()`

Este método hará que se eliminen los datos previos del formulario. El mejor caso de uso puede ser quizás un botón para resetear el formulario.

SI queremos modificar exclusivamente un único valor hay que modificar el método  proponer nombre. En él debemos acceder al subelemento `form` dentro de `formulario` y esta vez usar el método `.patchValue()`:

```typescript
 proponerNombre(){
    const nombreSugerido: 'MrRoot';
    this.formulario.form.patchValue({
      datosUsuario: {
        nombre: nombreSugerido,
      }
    })
  }
```

Así podremos modificar exclusivamente un único valor del formulario, en vez de al usar `setValue({})` que necesita definir todos los controles del formulario.

### Presentando los valores del formulario en la plantilla.

Para presentar los valores vamos a definir primero en la clase de nuestro componente un objeto que pueda contener estos datos, una variable booleana que contenga el estado del envío del formulario (es decir, si se ha hecho `click` o no en el botón), y por último en el método que se va a ejecutar al hacer `click` en el botón Enviar, vamos a hacer que ese método asigne los valores a nuestra propiedad objeto.

```typescript
export class AppComponent{

  usuario = {
    nombre: '',
    email: '',
    secret: ''
  }
  enviado: boolean = false;
  
  onSubmit(){
	this.enviado = true;
    this.usuario.nombre = this.formulario.value.datosUsuario.nombre;
    this.usuario.email = this.formulario.value.datosUsuario.email;
    this.usuario.secret = this.formulario.value.datosUsuario.secret;
  }
}

```

Podemos presentar esto valores a continuación de nuestro formulario en este mismo componente así:

```html
<form (ngSubmit)="onSubmit()" #aform="NgForm">
...
</form>
<div *ngIf="enviado">
  <p>Tu nombre es: {{usuario.nombre}}</p>
  <p>Tu email es: {{usuario.email}}</p>
  <p>Tu pregunta secreta es: {{usuario.secret}}</p>
</div

```

Anteriormente hemos dicho que podemos modificar todos los valores del formulario de una sola vez usando `this.referencialocal.setValue({objetoConLasMismasPropiedadesQueElValueDeNuestroFormulario})` y que un posible uso podría ser el de resetear el formulario, pero la verdad es que tenemos disponible un método mucho mas sencillo: `this.referencialocalformulario.reset()` y que además resetea las clases que Angular va agregando para guardar los estados del formulario.

Vamos a agregar este método para que se ejecute cuando hacemos click en enviar:

```typescript
onSubmit(){
	this.enviado = true;
    this.usuario.nombre = this.formulario.value.datosUsuario.nombre;
    this.usuario.email = this.formulario.value.datosUsuario.email;
    this.usuario.secret = this.formulario.value.datosUsuario.secret;
    this.formulario.reset();  //MUY IMPORTANTE, esperar a volcar los datos en la variable local antes de resetear el formulario ;)
  }
```

Podríamos usar de forma parecida `reset(`) a como usamos anteriormente `setValue()`, pasándole valores específicos a los que resetearse y funcionaría exactamente igual que `setValue()`, salvo por lo que quitar las clases agregadas de `ng-touched`, etc.