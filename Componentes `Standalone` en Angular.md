
Son componentes que funcionan de forma totalmente independiente. Se incluyó a partir de la versión 14 y se usa por defecto en la versión 17.

En vez de usar `appModule`, cada componente contiene su propio `módulo` dentro de si mismo.

Entonces, su decorador `@component` va a contener un array de `imports` para importar los módulos que se usen en el componente, los componentes hijos, y luego exportamos el componente y aquello que debemos exportar.

De esta forma nos libramos totalmente del uso de @NgModules

Un componente `standalone` tiene un decorador `@Component` algo distinto:

```typescript
@Component({
  standalone: true,
  selector: 'app-componente-ejemplo',
  templateUrl: './componente-ejemplo.component.html',
  styleUrls: ['./componente-ejemplo.component.css'],
})
```

Con esa primera propiedad, `standalone: true`, ya hemos definido a nuestro componente como `standalone`, y a partir de aquí su comportamiento va a ser diferente.

Ahora no va a permitir que este componente se declare en ningún otro módulo (recordemos que sólo se puede declarar un componente en un módulo, para usarlo en otros módulos debemos importar el módulo que lo declara). 

Porque ahora este componente ya se declara *en sí mismo*.

Además ya no va a tener disponibles todos los módulos que hemos importado en `AppModule` sino que necesita sus propias importaciones:

```typescript
@Component({
  standalone: true,
  imports: [CommonModule, BrowserModule, Router],
  provider: [],
  selector: 'app-componente-ejemplo',
  templateUrl: './componente-ejemplo.component.html',
  styleUrls: ['./componente-ejemplo.component.css'],
})
export class ComponenteEjemplo {}
```

Dentro del array de importaciones también podemos importar módulos propios (como `AppModule`).
