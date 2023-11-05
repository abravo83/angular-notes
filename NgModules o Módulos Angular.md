Los módulos de Angular no son iguales a los módulos de JavaScript. Un módulo de Angular nos define un *contexto de compilación* para un grupo de componentes dentro de un dominio de una aplicación. Por ejemplo, un módulo puede contener todos los servicios para formar una unidad funcional.

Toda aplicación de Angular tiene un módulo raíz, convencionalmente llamado `AppModule` , que nos provee del mecanismo de `bootstrapping` o lanzamiento de la aplicación. Una aplicación a su vez contiene muchos módulos funcionales.

Igual que ocurre con JavaScript, un módulo puede importar partes de otros módulos, y permitir que su propia funcionalidad sea exportada y usada por otros módulos. Por ejemplo: Para usar el servicio de *routing* en nuestra aplicación importamos el módulo `Router` 

Si organizamos nuestro código en diferentes módulos funcionales ayudamos al proceso de mantenimiento de dicho código y además podemos aprovechar la ventaja de usar [[lazy-loading]], o cargar los módulos bajo demanda para reducir la cantidad de código que necesitamos cargar al inicio de la aplicación.


