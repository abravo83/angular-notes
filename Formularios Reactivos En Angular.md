
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


En los formularios reactivos podemos especificar la función de validación. Esto lo hacemos directamente en la inicialización del grupo de control, donde utilizamos el segundo argumento del constructor  del `FormControl`. Para usar los `Validators` necesitamos importarlos de `'@angular/forms'`;

```typescript
import { FormGroup, FormControl, Validators} from '@angular/forms';
...

ngOnInit(): void {
  this.registroFormGroup = new FormGroup({
    'username': new FormControl(null, Validators.required),
    'email': new FormControl(null, [Validators.required, Validators.email]),
    'tipoUsuario': new FormControl('usuario', Validators.required)
  })
}
```

Podemos usar una única función `Validators` o varias usando un array de funciones. En el argumento usamos sólo la referencia a la función, pero no la ejecutamos, por eso las funciones de validación no llevan el `()` detrás.

## Accediendo a los valores de los controles.

Si por ejemplo queremos mostrar un mensaje debajo de alguno de nuestros inputs cuando no pasan la validación y tenemos que usar el valor resultado le la validación no seguimos el mismo procedimiento que seguíamos con un formulario TD, sino que usamos un método de nuestro grupo de control: `get('nombreDelControl')` y a partir de ese método ya podemos acceder al resultado de la validación

```html
<form [formGroup]="registroFormGroup" (ngSubmit)="onSubmit()">
  <div>
    <label>Usuario</label>
    <input type="text" [formControl]="username" />
    <span *ngIf="!registroFormGroup.get('username').valid && registroFormGroup.get('username').touched">Por favor, introduce un nombre de usuario</span>
    
  </div>
  <div>
    <label>Email</label>
    <input type="text" [formControl]="email" />
    <span *ngIf="!registroFormGroup.get('email').valid && registroFormGroup.get('email').touched">Por favor, introduce un email correcto</span>
  </div>
  <div *ngFor="let tipo of tipos">
    <label>
      <input type="radio" [formControl]="tipoUsuario"/>
    </label>
  </div>
  <button type="Submit">Enviar</button>
</form>
```

Otra forma de acceder a los valores de los controles, además del `get()` es usando una ruta. Esto es especialmente útil cuando hemos agrupado los valores en un GroupData

## Agrupando los controles y accediendo a ellos

Podemos agrupar algunos controles dentro de un nuevo `FormGroup` anidado, al que llamaremos `userData` 

```typescript

import { FormGroup, FormControl, Validators} from '@angular/forms';
...

ngOnInit(): void {
  this.registroFormGroup = new FormGroup({
	'userData' = new FormGroup({
	  'username': new FormControl(null, Validators.required),
      'email': new FormControl(null, [Validators.required, Validators.email]),
	}),  
    'tipoUsuario': new FormControl('usuario', Validators.required)
  })
}
```

Ahora, fíjate en que hemos agrupado un par de `inputs` dentro de un nuevo `<div>` al que le hemos asignado otro `FormGroup` llamado `userData`. Ahora los valores de esos campos están anidados, y para poder acceder a sus datos el HTML la forma va a cambiar de `registroFormGroup.get('username')` a `registroFormGroup.userData.get('username')`

```html
<form [formGroup]="registroFormGroup" (ngSubmit)="onSubmit()">
  <div [formGroupName]="userData">
	  <div>
	    <label>Usuario</label>
	    <input type="text" [formControl]="username" />
	    <span
	      *ngIf="!registroFormGroup.userData.get('username').valid && 
	             registroFormGroup.userData.get('username').touched"
             >	
		 Por favor, introduce un nombre de usuario
		 </span>
	  </div>
	  <div>
	    <label>Email</label>
	    <input type="text" [formControl]="email" />
	    <span *ngIf="!registroFormGroup.userData.get('email').valid && 
				    registroFormGroup.userData.get('email').touched"
				    >
			Por favor, introduce un email correcto
			</span>
	  </div>
  </div>
  <div *ngFor="let tipo of tipos">
    <label>
      <input type="radio" [formControl]="tipoUsuario"/>
    </label>
  </div>
  <button type="Submit">Enviar</button>
</form>
```

## Usando agrupación en Array

Supongamos que queremos crear en nuestro formulario un apartado donde el usuario va a poder agregar diferentes controles, como que el usuario diga sus aficiones y un botón para agregar nuevos campos para cada afición.

```html
<div formArrayName="aficiones">
  <label>Tus aficiones</label>
  <button type="button" (click)="onAgregarAficion()">Agregar una afición</button>
  <div *ngFor="let aficion of aficiones; let i = index;">
    <input type="text" [formControlName]="i"/>
  </div>
</div>
```


```typescript
import { FormArray, FormGroup, FormControl } from '@angular/forms';

...

ngOnInit(): void {
  this.registroFormGroup = new FormGroup({
	'userData' = new FormGroup({
	  'username': new FormControl(null, Validators.required),
      'email': new FormControl(null, [Validators.required, Validators.email]),
	}),  
    'tipoUsuario': new FormControl('usuario', Validators.required),
    'aficiones': new FormArray
  })
}

onAgregarAficion(){
  const control = new FormControl(null, Validators.required);
  (<FormArray>this.registroFormGroup.get('aficiones')).push(control)
}

get aficiones(){
  return (this.registroFormGroup.get('aficiones') as FormArray).aficiones;
}
```

## Creando Funciones Validadoras Propias

Ya hemos visto donde se incluyen las validaciones ya construidas, pero, por lo que sea, las validaciones disponibles no son suficientes y necesitamos una validación propia.

Para conseguir esto vamos a tener que incluir una función en nuestra clase que nos sirva para la validación e incluirla en el array de validaciones de la función de construcción del `FormGroup`.

```typescript

nombresProhibidos = ['Pili', 'Maki'];

ngOnInit(): void {
  this.registroFormGroup = new FormGroup({
	'userData' = new FormGroup({
	  'username': new FormControl(null, [Validators.required, this.nombresProhibidos.bind(this)] ),
	  // Como accedemos a esta función desde la plantilla, debemos referenciarla para que sepa a que 'this' nos referimos (¿?)
      'email': new FormControl(null, [Validators.required, Validators.email]),
	}),  
    'tipoUsuario': new FormControl('usuario', Validators.required),
    'aficiones': new FormArray
  })
}


nombresProhibidos(control: FormControl): {[s: string]: boolean} {
  // Cuando pasamos la validación la respuesta es NULL, cuando no la pasamos la respuesta es {'unstring': true};
  
  if (this.nombresProhibidos.indexOf(control.value)){
    return {'nombreProhibido': true};
  }
  return null;
}
```

Podemos añadir un mensaje propio a este error de validación en la plantilla

```html
<div>
	<label>Usuario</label>
	<input type="text" [formControl]="username" />
	<span
	  *ngIf="registroFormGroup.get('userData.username').errors['required']"
	  >	
	   Por favor, introduce un nombre de usuario
	 </span>
	 <span *ngIf="registroFormGroup.get('userData.username').errors['nombreProhibido']">Este nombre de usuario no está permitido</span>
</div>
```

## Creando un validador asíncrono

Cuando tenemos que hacer la validación tras hacer una consulta http o cualquier otro proceso asíncrono (Que tiene una respuesta posterior y la web sigue su proceso normal bloquearse mientras espera la respuesta), entonces necesitamos una validación asíncrona.

```typescript
import { Observable } from 'rxjs/Observable';
...

ngOnInit(): void {
  this.registroFormGroup = new FormGroup({
	'userData' = new FormGroup({
	  'username': new FormControl(null, [Validators.required, this.nombresProhibidos.bind(this)] ),
	  // Como accedemos a esta función desde la plantilla, debemos referenciarla para que sepa a que 'this' nos referimos (¿?)
      'email': new FormControl(null, [Validators.required, Validators.email, this.emailsProhibidosAsincronos.bind(this)]),  //En realidad aquí no es necesario porque la función no usa this.
	}),  
    'tipoUsuario': new FormControl('usuario', Validators.required),
    'aficiones': new FormArray
  })
}

emailsProhibidosAsincronos(control: FormControl): Promise<any> | Observable<any> {
  const promise = new Promise((resolve, reject)=>{
    setTimeout(()=>{
      if (control.value === 'test@test.com'){
        resolve({'emailProhibido': true})
      } else {
      resolve null
      }
    } ,1500);
    
	return promise;
  })
}

```

Si inspeccionamos el campo del input en las herramientas de desarrollador, veremos que durante los 1500ms se añade la clase `ng-pendig` y posteriormente `ng-valid` o `ng-invalid` según el email que hayamos introducido.

Así podremos realizar una validación asíncrona.

## Reaccionando a cambios de estado o cambios de valor 

Nos podemos subscribir a los cambios de estado o de valor de un campo, ya que nuestro `FormGroup` principal tiene dos `Observables` a los que nos podemos subscribir para ejecutar algún código cuando se produzca el evento de cambio:

```typescript
ngOnInit(): void {
  this.registroFormGroup = new FormGroup({
	'userData' = new FormGroup({
	  'username': new FormControl(null, [Validators.required, this.nombresProhibidos.bind(this)] ),
	 
      'email': new FormControl(null, [Validators.required, Validators.email, this.emailsProhibidosAsincronos.bind(this)]), 
	}),  
    'tipoUsuario': new FormControl('usuario', Validators.required),
    'aficiones': new FormArray
  });

  //Subscripciones a cambios de estado y de valor.
  this.registroFormGroup.valueChanges.subscribe((value)=>{
    console.log(value);
  });
  
  this.registroFormGroup.statusChanges.subscribe((satus)=>{
    console.log(status);
  });
  
}
```

