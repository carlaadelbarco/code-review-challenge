# Code review Carla Alvarez

Buenas a todos!
Muchas gracias por dejarme participar en este reto, me ha parecido super interesante y entretenido :)

# Arquitectura Hexagonal

Estamos utilizando una arquitectura hexagonal, que se caracteriza por la disposición de los paquetes en: infrastructure, application y domain. Esta estructura es adecuada para aislar la lógica central de las implementaciones de capas exteriores.

Este enfoque se basa en el Domain-Driven Design (DDD) y también se conoce como arquitectura de puertos y adaptadores. Las capas exteriores dependen de las capas interiores, pero no viceversa.

Esto garantiza que se cumplan los principios SOLID si se sigue correctamente la arquitectura hexagonal. Los principios SOLID que se deben de cumplir son:

- **Responsabilidad Única**: Cada clase tiene una única responsabilidad, facilitando los cambios.
- **Open-Closed**: Entidades y casos de uso están abiertos a extensión pero cerrados a modificación. Se añade funcionalidad extendiendo clases y escribiendo nuevo código, no modificando el existente.
- **Sustitución de Liskov**: Los adaptadores y puertos deben ser sustituibles sin afectar al comportamiento del sistema, permitiendo cambiar implementaciones de infraestructura o servicios externos fácilmente.
- **Segregación de la Interfaz**: Usar clases pequeñas y especializadas que extienden de una clase padre privada, evitando la dependencia de interfaces innecesariamente grandes.
- **Inversión de la Dependencia**: Las capas interiores no dependen de las exteriores. Las dependencias se invierten mediante la inyección de dependencias.

Sin embargo, algunas clases en el repositorio no están ubicadas correctamente según los principios de la arquitectura hexagonal.

## Propuesta de Organización de Clases

```plaintext
com.idealista
    ├── application
    │   └── usecases
    │       └── AdsServiceImpl.java
    ├── domain
    │   ├── ports
    │   │   ├── inputs
    │   │   │   └── AdsService.java
    │   │   └── outputs
    │   │       └── AdRepository.java
    │   └── models
    │       ├── Ad.java
    │       ├── Constants.java
    │       ├── Picture.java
    │       ├── Quality.java
    │       └── Typology.java
    ├── infrastructure
    │   ├── controllers
    │   │   └── AdsController.java
    │   ├── response
    │   │   ├── PublicAd.java
    │   │   └── QualityAd.java
    │   ├── entities
    │   │   ├── AdVo.java
    │   │   └── PictureVo.java
    │   ├── mappers
    │   │   ├── PictureMapper.java (nueva clase propuesta)
    │   │   └── AdMapper.java (nueva clase propuesta)
    │   ├── persistence
    │   │   └── InMemoryPersistence.java
    │   ├── handler
    │   │   └── GlobalExceptionHandler.java (posible clase añadida para el manejo de errores)
    │   └── config
    │       (Aquí irían posibles clases de configuración necesarias, como la conexión con BBDD por ejemplo)
    └── Main.java
```


Se han reorganizado diversas clases para ajustarse mejor a una arquitectura hexagonal.
Mi propuesta sería la siguiente:

## Capa de Infraestructura:

### API
    - Uso de @Data: En las clases #AdVO y #PictureVO se podría usar @Data de Lombok para ahorrar líneas de código en getters y setters.

    - AdsController: Añadir anotaciones de Swagger para que los usuarios sepan qué tipo de datos deben utilizar.

    - La clase #QualityAd está utilizando para el parámetro irrelevantSince un tipo Date, y este tipo de dato no esta mal, pero era mas utilizado en versiones anteriores. A partir de Java 8, se utiliza mejor el tipo LocalDate de la biblioteca de "java.time" siendo así mas robusta y menos propensa a errores. 

    - En #Ad y #Picture podría ser recomendable, en función de las casuísticas especificas, añadir a todos los campos "final", si no se esperan cambios una vez establecidos. Los objetos inmutables pueden ser mas eficientes en términos de memoria. Siguiendo la misma linea y teniendo en cuenta que esto seria solo en algunas casuísticas, todos los campos que sean Integer, si son sustituidos por "int" en algunas ocasiones puede reducir el consumo de memoria. Si se aplicara este cambio, no se podría utilizar colecciones tipo List<Integer> ni utilizar métodos adicionales como toString(), compareTo(), etc

### persistence
Personalmente esta capa la dividiria en dos: 
    - persistency: para implementar los repositorios de JPA de nuestra aplicación, este sera el punto de acceso de nuestra bbdd. En esta carpeta lo correcto seria poner #InMemoryPersistence. Las clases de repositorios jpas deben ir en las capas mas externas de una arquitectura hexagonal, para asegurar que cumplimos con los patrones solid. Asi de esta manera, se puede cambiar el tipo de bbdd que utilizamos sin que afecte a los demás componentes.
    - entities: aquí irían las clases PictureVo y AdVo, para que ambas estuvieran correctamente separadas y de esta manera se hace un código mas mantenible en el futuro


## Capa de Aplicación

En esta capa se encuentra la implementación de los casos de uso
    - La clase AdsServiceImpl contiene en su código lógica de mappeo de datos que debería estar en una clase independiente que se dedique únicamente a mappear los datos. De esta manera cumplimos el principio SOLID de responsabilidad única. Este mappeo seria mejor implementarlo en la capa de infraestructura, en una carpeta especifica para los mappers.

## Capa de Domain:
    - Aqui he optado por implementar los puertos (inputs y outputs) y los models.


# Buenas Prácticas, Principios SOLID y Clean Code

- En la clase #inMemoryPersistence se podrían sustituir los parámetros List<AdVO> ads y List<PictureVO> pictures por Map<Integer, AdVO> y Map<Integer, PictureVO>, ya que los mapas proporcionan una búsqueda, inserción y eliminación más eficientes en comparación con las listas. Además, si las casuísticas del proyecto lo permiten y fueran inmutables, haría estas variables “final”, mejorando la seguridad del código y mejorando la eficiencia en términos de memoria. 

- En las clases #QualityAd y #PublicAd podrían heredar de una clase padre, ya que comparten muchos campos en común. De esta manera, cumpliríamos con el principio solid de "Segregación de la interfaz", que indica que se deben utilizar clases pequeñas y especializadas que extiendan de una clase padre privada. No se deben conocer los métodos que no se van a utilizar. 

- Textos hardcodeados: En las clases de #AdsServiceImpl y #InMemoryPersistence, en el constructor, se están poniendo textos hardcodeados en el código. Esto no es una buena practica, ya que si se desea modificar alguno de estos valores habría que modificar el código de la aplicación y redesplegarlo. Una buena practica que se puede implementar es crear un fichero de properties común para todo el proyecto, en el que se vayan poniendo las propiedades con los textos correspondientes, siendo este fichero independiente del código. La manera de recuperar dichos valores del fichero de propiedades seria a través de una anotación del framework de spring: por ejemplo: "@Value("${propertie.value}")" 

- En la clase Constants, podríamos modificar el nombre de cada una de las variables para que no dependieran del valor, ya que si se desea cambiar el valor de una de ellas, deberíamos cambiar también el nombre de la constante, y eso haría que tuviéramos que cambiarla en el resto del código donde se estuviera utilizando. Es una buena practica utilizar nombres genéricos.

- Echo en falta la utilización de anotaciones de Lombok, para facilitar la lectura/escritura de nuestro código. Ademas de lo comentado anteriormente en el punto YYYYY(cambiar), se podría utilizar @Data en clases como #AdVO o #PictureVO.

# Swagger API:
- No hay fichero swagger.yaml para la integración de terceros. A partir de este fichero, cualquier tercero que lo importe en su proyecto y utilizando API First, podría generar los dtos necesarios.
- No hay uso de swagger para poder hacer uso de OpenAPI para la visualización del endpoint. Esto se haría de una manera muy sencilla

Recomiendo en este caso, la dependencia de OpenAPI o Swagger para la generacion de la documentación.

Tambien podemos incluir la dependencia de "Springfox" para generar la interfaz web para que terceros puedan integrarse de manera sencilla con estos enpoints.


# Errores
Manejo de errores:
    - No hay manejo de errores y excepciones. De hecho, probando el servicio, cuando se produce algún error, siempre devuelve un error 500. 
    Seria mas recomendable utilizar una clase genérica de manejo de errores o excepciones, de tal manera, que estuvieran categorizados cada uno de los errores que pudieran darse. Errores 500, 400, etc. 
    Propongo la utilización de una clase que sea "GlobalExceptionHandler" con la anotación @ControllerAdvice del spring de framework y categorizar cada uno de los errores en un json que devuelva el código de error y un mensaje explicando el tipo de error que se ha dado.

    Esta clase se podria añadir en la capa de infrastructure - handler

# Performance:

- En cuanto al almacenamiento y persistencia en memoria, he podido observar que esta almacenando los datos en tiempo de ejecución, ya que no existe ninguna BBDD ni ningún tipo de almacenamiento en general. Podríamos utilizar una bbdd como postgres por su robustez y alta disponibilidad para almacenar todos los datos que necesitemos. También podríamos utilizar una bbdd no relacional como MongoDB o incluso un elasticSearch si en un futuro se quieren incorporar mecanismos de búsqueda y predicción de la búsqueda mas avanzados, ya que para este tipo de buscadores, elasticsearch es una gran opción.
De esta manera, la clase "InMemoryPersistence" podría ser modificada para que recuperara sus datos en la bbdd que elijamos, y no tendría que estar haciendo las consultas en memoria consumiendo muchos mas recursos de los necesarios.

- Los streams en java consumen mucha memoria, hay que tenerlo en cuenta. En la clase AdsServiceImpl se utilizan.

- Se podría utilizar algo como redis para cachear los pisos y no tener que estar recurriendo a la bbdd constantemente

- Aunque no es el motivo de esta code review, de cara a un posible despliegue, se podrían incluir balanceadores de carga para no sobrecargar un único servidor

# Testing

- Puedo observar que existe un test unitario para cubrir una funcionalidad, pero esto provoca una baja cobertura. Existen muy pocos test unitarios, por lo que yo realizaría test unitarios con JUnit y Mockito para cubrir cada uno de los casos de uso que existen en la aplicación. 
Además, se podria añadir la dependencia mvn en el pom de jacoco, para que cada vez que se ejecute mvn test genere un archivo de cobertura de los test de la aplicacion y podamos tener una vision general.

- Añadiria mas tipos de test, como por ejemplo: 
    - Integración: 
    Se utilizan para verificar la interacción entre diferentes módulos o componentes de una aplicación. Aseguran que los módulos funcionen juntos como se espera cuando se integran. (mockito)

    - Contraect test / Contrato:
    Aseguran que diferentes servicios que interactúan entre sí respeten un contrato común, es decir, que las interfaces y las expectativas de respuesta se mantengan consistentes. En java se usa Pact

    - Pruebas de Carga:
    Se utilizan para evaluar el rendimiento de una aplicación bajo condiciones de carga esperadas o máximas. JMeter

Todas estas pruebas se pueden incluir en una pipeline de integración continua para automatizar las pruebas, usando un servidor de CI/CD (Jenkins, Azure…)

# Integracion continua y despliegue continuo - CI/CD
Echo en falta algún archivo por ejemplo docker-compose.yaml para hacer mas sencillo el despliegue y arranque en nuestro local. Se podría incorporar un docker para que cuando levantáramos la aplicación nos desplegara una bbdd por ejemplo.
Además de, como hemos comentado en el punto anterior, seria interesante incluir un paquete de pruebas integrado en CI/CD para que se ejecuten los test antes de hacer un despliegue, de tal manera, que si no supera los test o cierta cobertura que configuremos, no sea posible realizar el despliegue.

# Configuracion de entornos
En este punto, quiero comentar por encima, que es una buena practica tener ficheros de configuracion separados por entornos, en los que cada uno de los ficheros hace referencia a la configuración especifica del entorno en el que se encuentra.






