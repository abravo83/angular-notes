
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

