# mk96-lib-java-common-code

Librería Java compartida con utilidades de uso común y soporte para escribir tests basados en ficheros. Es un proyecto Maven de varios módulos, con Java 21.

## Módulos

Las coordenadas de Maven son `org.mk96.common:<módulo>:1.0.0` y las clases viven en el paquete `org.mk96.common`.

| Módulo | Qué contiene |
|---|---|
| `bom` | Lista de versiones de los demás módulos, para importarla y no repetir la versión en cada dependencia |
| `core` | Utilidades generales (ver abajo). Depende de Lombok, Jackson, Apache Commons Lang y Apache Commons IO |
| `test` | `AbstractMkTest`: clase base con utilidades para tests, como filtrar un resultado con expresiones regulares. Depende de `core` y de JUnit 5 |
| `test-template` | Extensión de JUnit 5 de "plantillas de test" basadas en ficheros JSON. Depende de `test` |

### `core`

| Clase | Para qué sirve |
|---|---|
| `MkLoadFileUtils` | Carga un fichero como bytes, como texto o como objeto, y resuelve rutas absolutas |
| `MkSerializeUtils` | Convierte JSON en objetos y objetos en JSON, en texto o en bytes, con formato normal o plano |
| `MkReflectionUtils` | Utilidades de reflexión: listar campos de una clase o un objeto y leer su tipo o su valor |
| `MkMapUtils` | Utilidades para mapas |
| `MkObject` | Clase base con un `toString()` común |
| `MkException` | Excepción base de la librería |
| `CommonConstant`, `DateFormatConstant`, `RegexConstant` | Constantes comunes: formatos de fecha y expresiones regulares |

### `test-template`

Pensado para tests que cargan un ejemplo de entrada desde un fichero y comparan el resultado con otro fichero.

- La anotación `@MkTestTemplate` marca un método de test dentro de una clase que extiende `AbstractMkTestTemplate`. Permite configurar el tipo de plantilla, el nombre de los ficheros de ejemplo y de resultado, y las expresiones regulares para filtrar campos variables (por ejemplo, un UUID).
- Incluye manejadores para cargar el ejemplo, comparar el resultado, filtrarlo, inicializar los ficheros y actualizar el fichero de resultado.
- Los tests del propio módulo (`src/test`) sirven de ejemplo de uso, con sus ficheros JSON en `src/test/resources`.

## Requisitos

- JDK 21
- Maven 3.9 o superior

## Compilar y probar

```bash
mvn clean verify
```

Compila todos los módulos y ejecuta los tests: 6 en `core` y 30 en `test-template`.

Para instalar la librería en tu repositorio local de Maven (`~/.m2`) y usarla desde otros proyectos de tu máquina:

```bash
mvn clean install
```

## Usarla desde otro proyecto

Añade la dependencia que necesites, con las coordenadas `org.mk96.common`:

```xml
<dependency>
    <groupId>org.mk96.common</groupId>
    <artifactId>core</artifactId>
    <version>1.0.0</version>
</dependency>
```

Si quieres alinear las versiones de todos los módulos, importa el `bom` en `dependencyManagement`:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.mk96.common</groupId>
            <artifactId>bom</artifactId>
            <version>1.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Para que Maven la encuentre, o la instalas en local (`mvn clean install`, ver arriba) o la descargas de GitHub Packages, como se explica a continuación.

## GitHub Packages: dónde poner el token y cómo

La librería se publica en GitHub Packages, en `https://maven.pkg.github.com/mk96-lib/mk96-lib-java-common-code`. GitHub Packages exige un token incluso para descargar.

**El token nunca va en el repositorio ni en el `pom.xml`.** Va en tu fichero personal `~/.m2/settings.xml`, leído desde una variable de entorno.

1. Crea un [token personal de GitHub](https://github.com/settings/tokens) (clásico) con el permiso `read:packages` para descargar, o `write:packages` si también vas a publicar.
2. Guárdalo en una variable de entorno, por ejemplo en tu `~/.bashrc`:
   ```bash
   export GITHUB_TOKEN=tu_token_aquí
   ```
3. Añade el servidor en `~/.m2/settings.xml`. El `id` debe ser `github`, el mismo que usa el `pom.xml`:
   ```xml
   <settings>
     <servers>
       <server>
         <id>github</id>
         <username>TU_USUARIO_DE_GITHUB</username>
         <password>${env.GITHUB_TOKEN}</password>
       </server>
     </servers>
   </settings>
   ```
4. En el proyecto que la use, declara el repositorio con ese mismo `id`:
   ```xml
   <repositories>
     <repository>
       <id>github</id>
       <url>https://maven.pkg.github.com/mk96-lib/mk96-lib-java-common-code</url>
     </repository>
   </repositories>
   ```

### Publicar una versión

Con el token de `write:packages` configurado como arriba:

```bash
mvn clean deploy
```

Publica todos los módulos en GitHub Packages, en la dirección que declara `distributionManagement` en el `pom.xml` raíz.

## Estructura del repositorio

```
.
├── pom.xml                                   Proyecto padre, con la versión de Java y el destino de publicación
├── .gitignore
├── bom/
│   └── pom.xml
└── modules/
    ├── pom.xml
    ├── core/
    │   ├── pom.xml
    │   └── src/
    │       ├── main/java/org/mk96/common/
    │       │   ├── constant/
    │       │   │   ├── CommonConstant.java
    │       │   │   ├── DateFormatConstant.java
    │       │   │   └── RegexConstant.java
    │       │   ├── type/
    │       │   │   ├── MkException.java
    │       │   │   └── MkObject.java
    │       │   └── util/
    │       │       ├── MkLoadFileUtils.java
    │       │       ├── MkMapUtils.java
    │       │       ├── MkReflectionUtils.java
    │       │       └── MkSerializeUtils.java
    │       └── test/
    │           ├── java/org/mk96/common/util/
    │           │   ├── requirements/FileObject.java
    │           │   ├── MkLoadFileUtilsTest.java
    │           │   ├── MkReflectionUtilsTest.java
    │           │   └── MkSerializeUtilsTest.java
    │           └── resources/util/
    │               ├── loadfile/getAbsolutePathExist.json
    │               ├── loadfile/loadFileExample.json
    │               └── serialize/example.json
    ├── test/
    │   ├── pom.xml
    │   └── src/main/java/org/mk96/common/test/
    │       └── AbstractMkTest.java
    └── test-template/
        ├── pom.xml
        └── src/
            ├── main/java/org/mk96/common/test/template/
            │   ├── AbstractMkTestTemplate.java
            │   ├── annotation/
            │   │   ├── MkTestTemplate.java
            │   │   └── MkTestTemplateConfiguration.java
            │   ├── constant/
            │   │   └── MkTestTemplateConstant.java
            │   ├── context/
            │   │   ├── MkTestTemplateInvocationContext.java
            │   │   ├── MkTestTemplateInvocationContextProvider.java
            │   │   └── MkTestTemplateParameterResolver.java
            │   ├── exception/
            │   │   └── MkTestTemplateException.java
            │   ├── handler/
            │   │   ├── compare/MkTestTemplateCompareHandler.java
            │   │   ├── filter/MkTestTemplateFilterHandler.java
            │   │   ├── initialize/
            │   │   │   ├── MkTestTemplateFileInitalizeHandler.java
            │   │   │   ├── example/MkTestTemplateExampleFileInitalizeHandler.java
            │   │   │   └── result/MkTestTemplateResultFileInitalizeHandler.java
            │   │   ├── invoke/MkTestTemplateInvokeHandler.java
            │   │   └── update/MkTestTemplateResultFileUpdateHandler.java
            │   └── type/
            │       ├── MkTestTemplateExecution.java
            │       ├── MkTestTemplateMode.java
            │       ├── MkTestTemplateParameter.java
            │       └── MkTestTemplateType.java
            └── test/
                ├── java/org/mk96/common/test/template/
                │   ├── requirements/ExampleUUID.java
                │   ├── CompareResultTemplateTest.java
                │   ├── FilterResultTemplateTest.java
                │   ├── InitTemplateTest.java
                │   ├── LoadFileTemplateTest.java
                │   └── SimpleTemplateTest.java
                └── resources/test/template/
                    ├── compare/   (template1 y template2, con ResultByte, ResultObject y ResultString)
                    ├── filter/    (template1 y template2, con ResultByte, ResultObject y ResultString)
                    ├── init/      (template1 y template2, con example.json y result.json)
                    ├── load/      (template1 y template2, con ExampleByte, ExampleObject y ExampleString)
                    └── simple/    (template1 y template2, con result.json)
```
