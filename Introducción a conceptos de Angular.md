Angular es tanto una plataforma como un framework para construir SPAs (Single Page Client Applications) usando HTML y JavaScript.

### Fundamentos de la #Arquitectura de una aplicación Angular

Los componentes básicos, los ladrillos, son los **[[Componentes de Angular]]** que están agrupados u organizados en [[NgModules o Módulos Angular]].

Los Módulos de Angular a su vez recogen código en *grupos funcionales*. Se puede definir por tanto a una aplicación de Angular como *un conjunto de NgModules*

Hay como mínimo un *módulo raíz*, pero normalmente hay varios módulos más que hacen la función de *módulos de características*, y este módulo raíz es el lanzado en el arranque (bootstrapping).

Los componentes definen *vistas*, que son grupos de elementos de la pantalla que Angular puede seleccionar y modificar de acuerdo con la lógica de tu programa.

Los componentes usan #servicios, que permiten a los *proveedores de servicios* ser **inyectados** en los componentes.

Los **Módulos, Componentes y Servicios** son clases que usan #decoradores . Estos decoradores indican a cual de estos tipos pertenece la clase y agregan metadatos que le indican a Angular cómo usarlos.

* Los metadatos de un componente asocian una #plantilla a dicho componente, que define su vista. En una plantilla se combinan HTML ordinario con #directivas de Angular y con #bindings o enlaces de variables que nos permiten modificar el HTML antes de renderizarlo.
* Los metadatos de un *servicio* le dan información a la aplicación para hacerlos disponibles a los componentes mediante [[inyección de dependencias (DI)]]

Los componentes de una aplicación típicamente definen muchas vistas organizadas de forma jerárquica. Angular nos permite usar el servicio [[Router]] para facilitar la definir la navegación entre vistas para el navegador.

