
## Pipes y para qué son útiles.

Son una herramienta disponible para modificar y filtrar los datos que recibimos. Tenemos Pipes para procesos síncronos y procesos asíncronos. 

Un ejemplo básico de un uso de Pipes sería por ejemplo modificar una variable para mostrarla en mayúsculas cuando el dato nos viene en minúsculas, salvo la primera letra.

```typescript
username = "Manolito" => "MANOLITO" //

```

```html
<p>{{username | uppercase}}</p>
```

## Utilizando Pipes

Para poder usar un `pipe`, lo primero es saber dónde vamos a utilizarlo, y como los `pipes` nos van a servir para mostrar un resultado modificado, su  lugar natural de uso va a ser donde mostramos nuestra información, es decir **en nuestra plantilla HTML**

```html
<strong>{{serverInstance | uppercase}}</stong>
```


## Parametrizando Pipes


Los parámetros para configurar un `pipe` se agregan al propio `pipe` agregándole dos puntos y después una cadena de texto

```html
<li>{{server.created | date:'fullDate'}}</li>
```

Cada pipe tendrá sus propios parámetros permitido, y se pueden encadenar distintos tipos de parámetros
```html
<li>{{server.created | date:'fullDate':'otroParametro'}}</li>
```

## Encadenando Pipes

Podemos encadenar Pipes para conseguir dos modificaciones encadenadas, pero el orden puede ser importante. Los `pipes` van a seguir una lógica de izquierda a derecha . El siguiente ejemplo sólo funciona si mantenemos el orden que hemos establecido. Si invertimos el orden de los pipes el ejemplo no funcionaría:

```html
<li>{{server.created | date:'fullDate' | uppercase }}</li>
```


## Creando un Pipe personalizado

Tenemos la posibilidad de crear nuestros propios `pipes` creando un nuevo archivo con un nombre descriptivo: `shorten.pipe.ts` por ejemplo, donde nuestro archivo contendrá:
```typescript
import { Pipe, PiperTransform } from '@angular/core';

@Pipe({
  name: 'shorten'
})
export class ShortenPipe implements PipeTransform {

 // Método necesario
  transform(value: any){
    return value.substr(0,5);
  }

}
```

Para poder usarlo, necesitamos declararlo como un componente más en nuestro `AppModule`

```typescript
import { ShortenPipe } from './shorten.pipe';

@NgModule({
declarations: [
  ...
  ShortenPipe
],
...
})
```

Y ya lo tendríamos disponible en nuestro componente

```html
<li>{{server.name | shorten}}</li>
```


## Parametrizando un Pipe personalizado

Para parametrizar un Pipe personalizado debemos agregar a nuestra función `transform` argumentos que van a ser nuestros parámetros

```typescript
import { Pipe, PipeTransform} from '@angular/core'

@Pipe({
  name: 'shorten'
})
export class ShortenPipe implements PipeTransform {

  transform(value: any, limit: number){
    if (value.lenght > limit) {
      return value.substring(0, limit) + '...'
    }
    return value;
  }

}

```

```html
  <p>{{server.name | shorten:5}}</p>  // Nos devuelva 5 caracteres y '...', p.e. 'Produ...'
```

## Uso de Pipes como buscadores

Si ponemos un input de tipo text y enlazamos con `NgModel` su contenido a una propiedad de la clase del componente prodemos usar dicho input como una especie de buscador

```html
<input type="text" [ngModel]="textofiltrado">
```

```typescript
//En el componente
textofiltrado: string = '';
```

```typescript
// Mi Pipe personalizado
import { Pipe, PipeTransform} from '@angular/core'

@Pipe({
  name: 'filter'
})
export class FilterPipe implements PipeTransform {

  transform(value: any, filterString: string){
    if (value.length === 0) {
      return value;
    }

    let arrayResultado = [];
    
	for (const item of value){
	  if (item.name.indexOf(filterString) !== -1 || filterString === ''){
	    arrayResultado.push(item);
	  }
	}

    return arrayResultado;
  }

}
```

Ahora vamos a utilizar el Pipe en nuestro `*ngFor` para que nos filtre los resultados que contienen ese `string`

```html
<li *ngFor="let server of server | filter:textoFiltrado">
  {{server.name}}
</li>
```


## Pipes puros e impuros

Un Pipe `puro` es un filtro que no se recalcula cuando se modifican los datos. Los `Pipes` nativos de Angular son todos `puros` ya que un `Pipe` impuro tiene un alto coste en el rendimiento de la aplicación.

Aún así, nosotros podemos tener nuestros propios `Pipes` impuros. Para esto debemos usar un `Pipe` personalizado y agregar en el decorador la propiedad `pure: false`

```typescript
import { Pipe, PipeTransform} from '@angular/core'

@Pipe({
  name: 'filter',
  pure: false
})
```

Estos `Pipes` se recalculan cada vez que se modifican los datos que se están filtrando.

## Entendiendo Pipes Asíncronos

Hay un `Pipe` que realiza una función algo diferente a la de los demás `Pipes`. Nos permite mostrar datos asíncronos tales como una petición `http`.

Si en nuestro HTML tenemos una propiedad que aún no ha recibido los datos de una petición, se nos van a mostrar como un `[Object object]` de una promesa sin resolver, a pesar de que el dato que esperamos sea `string`.

Podemos usar un `Pipe` llamado **`async`** para que filtre el resultado sólo cuando se resuelva esa promesa:

```html
<h2>{{serverName | async}}</h2>
```

De esta forma no se mostrará el resultado `[Object object]` antes de recibir el dato de la promesa.