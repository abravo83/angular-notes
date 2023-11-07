
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


## ¿Cómo es un Formulario TD?

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
Lo vamos a hacer agregando a nuestra etiqueta `input` la directiva `ngModel`

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

