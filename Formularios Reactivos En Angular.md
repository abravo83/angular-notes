
## ¿ Qué son Formularios Reactivos?

Mientras que con los Formulario dirigidos por plantilla o `Template Driven` la forma del formulario y el `objeto formulario`  era capturado por Angular al interpretar el HTML, en los **Formularios Reactivos** estos objetos se crean a través de la lógica (Nuestro archivo `.ts`) .


## ¿Cómo se crea un Formulario Reactivo?

Veamos un ejemplo en la lógica de un componente (un componente modular, no `standalone`):

```typescript
import { Component } from '@angular/core';
import { FormGroup } from '@angular/forms';


@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.componente.css']
})
export class AppComponent {

  tipos: string[] = ['usuario', 'administrador'];
  //Donde se van a agrupar nuestras respuestas del formulario.
  registroFormGroup!: FormGroup;

  // Además para poder usarlo debemos importar ReactiveFormsModule de '@angular/forms' en nuestro AppModule, el el array Imports.
  // Si usas un componente standalone, lo debes entonces importar en el array imports del decorador @Component, y asegurarte que arriba se agrega el import a '@angular/forms'.
}
```

Con estos elementos: Una importación de `ReactiveFormsModule`, y un `FormGroup`ya estaríamos listos para poder crear nuestro primer formulario reactivo.

Vamos ahora a definir nuestro `FormGroup` que hemos llamado `registroFromGroup`, y lo vamos a hacer en el ciclo de vida `OnInit`:

```typescript
import { Component, OnInit } from '@angular/core';
import { FormGroup } from '@angular/forms';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.componente.css']
})
export class AppComponent implements OnInit {
  tipos: string[] = ['usuario', 'administrador'];
  registroFormGroup!: FormGroup;

  ngOnInit(): void {
    this.registroFormGroup = new FormGroup({
      //Aquí añadiremos nuestros controles (pares campo-valor)  
    })
  }
}
```

Veamos ahora, antes de añadir nuestros controles al `FormGroup`, cómo es la estructura de nuestro formulario

```html
<form [formGroup]="registroFormGroup" (ngSubmit)="onSubmit()">
  <div>
    <label>Usuario</label>
    <input type="text" [formControl]="username" />
  </div>
  <div>
    <label>Email</label>
    <input type="text" [formControl]="email" />
    <!-- type="email" también nos sirve para aprovechar la validación HTML5-->
  </div>
  <div *ngFor="let tipo of tipos">
    <label>
      <input type="radio" [formControl]="tipoUsuario"/>
    </label>
  </div>
  <button type="Submit">Enviar</button>
</form>
```

Como ves, se han sustituido los `name` por `formControl` y hemos agregado `[formGroup]='registroFormGroup'` y `(ngSubmit)="onSubmit()"`  a nuestra etiqueta `<form>`.

Si recordamos, en el formulario TD también agregamos `(ngSubmit)` pero **hay una diferencia con respecto al formulario TD**. En este caso **no hemos pasado una referencia local** como argumento de `onSubmit(aform) #aform`. Tampoco hemos creado dicha referencia local.

Ahora que ya tenemos nuestro html podemos saber como vamos a configurar nuestro `FormGroup`. Pero necesitamos especificar que nuestros controles son del tipo `FormControl`, así que se agrega a nuestras importaciones desde `'@angular/forms'`

```typescript
...
import { FormGroup, FormControl } from '@angular/forms';
...
ngOnInit(): void {
  this.registroFormGroup = new FormGroup({
    'username': new FormControl(null), // También he visto usar '' para dejar un campo vacío
    'email': new FormControl(null),
    'tipoUsuario': new FormControl('usuario') //Agregamos un valor por defecto.
  })
}

onSubmit(){
  console.log(this.registroFormGroup);
}
```


## Agregando validaciones a nuestro formulario.



