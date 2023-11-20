
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
