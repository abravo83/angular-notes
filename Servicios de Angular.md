
## ¿Qué son servicios?

Son una clase que funciona como repositorio donde centralizar funciones y almacenamiento de datos para los componentes de nuestra aplicación, así como medio de comunicación entre componentes.

## ¿Cómo se crea un servicio?

Debemos crear una clase en un nuevo archivo de nuestra aplicación usando como convención para nombrarla el formato `tunombredelservicio.service.ts`

Dentro de la clase creamos una clase exportable y NO le añadimos decorador ninguno:

```typescript
export class LogginService {
 logStatusChange(status: string) {
   console.log('A server status changed, new status: ' + status);
 }
}
```


## ¿Cómo se consume un servicio?

A pesar de que si simplemente lo importamos en la clase de nuestro componente y lo instanciamos dentro de él esto funcionará, esta no es la manera adecuada de utilizar servicios en Angular. La manera correcta es utilizando **Inyección jerárquica** mediante el `inyector de dependencias`

### ¿Qué es un inyector de dependencias?

Es una clase de la que dependen nuestros componentes y que se instancia de manera automática en nuestros componentes si le informamos a nuestra clase que necesitamos dicha instancia del servicio.

#### ¿Y cómo informamos a Angular que nuestra clase necesita una instancia del servicio?

Añadimos a la clase de nuestro componente una función constructora y enlazamos una propiedad usando la palabra `private` delante del nombre que queramos usar en nuestra clase para el servicio y le asignamos **como tipo** el nombre del servicio:

```typescript
\\ La clase de nuestro componente

export class NewAccountComponent {

  constructor(private logginService: LogginService) {}
}
```

Esto forzará la importación de nuestro servicio en la clase de nuestro componente.
Además, debemos incluir el nombre de nuestro servicio dentro del array `providers` de nuestro componente:

```typescript
import { Component } from '@angular/core';
import { LogginService } from './logging.service'

@Component({
  selector: 'app-new-account',
  templareUrl: './new-account.component.html',
  styleUrls: ['./new-account.component.css'],
  providers: [LogginService]  
})
export class NewAccountComponent {
  constructor(private loggingService: LoggingService) {}

  this.loggingService.logStatusChange(accountStatus);
...
}
```

Como nuestro servicio es un **inyector jerárquico** estará disponible en todos los componentes hijos del padre en el que lo hemos establecido como `provider`

Hay una forma de que nuestro servicio esté disponible en toda nuestra app: Usando el array de `providers` disponible en *AppModule*. Así estará disponible para todos nuestros componentes y **todos nuestros servicios**

Hay otra posibilidad, usar el array `providers` disponible en *AppComponent*. Así estará disponible para todos nuestros componentes pero **NO para nuestros servicios**

Si tenemos nuestro servicio agregado a diferentes arrays `providers` lo que tendremos es *diferentes* instancias del mismo servicio. Mientras que si lo tenemos todo en un nivel superior, como en **AppModule** tendremos una única instancia que se comparte en toda la aplicación.

#### ¿Cómo inyectamos nuestro servicio en otro servicio?

Si tenemos nuestro servicio como proveedor en `AppModules` podemos usarlo dentro de otro servicio de la misma forma que lo inyectamos en un componente, pero necesitamos agregar el decorador `@Injectable()`.

```typescript
@Injectable()
export class MiServicio {
 
 constructor(private nombreDelOtroServicio: NombreDelOtroServicio){}
}
```

#### El decorador `@Inyectable`

El decorador `@Injectable()` no es necesario para todos nuestros servicios, sólo para los servicios que van a aceptar a otros servicios como *inyectables*

Sin embargo, en las versiones más modernas de Angular está recomendado agregar *siempre* el decorador a nuestra clase **servicio**

Además, el decorador nos permite una cosa extra. Si al agregar nuestro decorador le agregamos un objeto de configuración con la propiedad `{providedIn: 'root'}` le indicamos a Angular el nivel donde está actuando como *proveedor* y  <i><b><u>POR TANTO YA NO NECESITA SER AGREGADO A NINGÚN ARRAY `providers`</u></b></i>

```typescript
import { Inyectable } from '@angular/core';
import { NombreDelOtroServicio } from './nombre-del-otro-servicio.service';

@Inyectable({
providedIn: 'root'
})
export class NombreDelServicio {

  constructor(private nombreDelOtroServicio: NombreDelOtroServicio)

}
```

### Comunicación entre componentes usando un servicio (usando `EventEmmiter`)

Queremos usar un evento que emitamos desde un componente y que se pueda escuchar en otro componente. Para esto vamos a seguir los siguientes pasos:


En nuestro servicio creamos el `EventEmmiter`
```typescript
@Inyectable()
export class NuestroServicio {

statusUpdated = new EventEmmiter<any>();
}
```

En nuestro componente emisor:

```typescript
@Component({})
export class ComponenteEmisor {
  constructor(private nuestroServicio: NuestroServicio){}
  onFuncionQueEmite() {
    this.nuestroServicio.statusUpdated.emit(status);
  }
}
```

En nuestro componente receptor

```typescript
@Component({})
export class ComponenteReceptor {
  constructor(private nuestroServicio: NuestroServicio){
  
    this.nuestroServicio.statusUpdated.subscribe(
      (status: string) => alert('Actualización de status: ' + status):
    );
    
  }
}
```
### Comunicación entre componentes usando un servicio

Podemos utilizar diferentes estrategias:
 * Propiedades: Compartiendo una propiedad entre componentes
 * Subjects: Nos permite definir un objeto y emitir un evento para que se actualice en otro componente
 * BehaviourSubject: Nos permite definir un objeto y emitir un evento para que se actualice en otro componente. Nos subscribimos en los componentes a este subject.
 * Signals: Nos permite definir un objeto y emitir un evento para que se actualice en otro componente. Nos subscribimos en los componentes a esta signal.

