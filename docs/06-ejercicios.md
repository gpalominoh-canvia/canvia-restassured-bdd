# Ejercicios del curso

> Enunciados de los ejercicios 1 a 5, en el orden del [sílabo](05-silabo.md). Láminas con el mismo contenido:
> [presentación del curso (PDF)](pdf/presentacion-restassured-cucumber.pdf).

Todos los ejercicios se hacen sobre este proyecto y contra la API de práctica
[Restful Booker](https://restful-booker.herokuapp.com/apidoc/index.html). Cada uno suma una habilidad al anterior.

| Sesión | Ejercicios | Tiempo en clase | Rama de entrega |
|---|---|---|---|
| 1 · martes 27 | [1](#ejercicio-1--buscar-reservas-y-pedir-lo-que-no-existe) y [2](#ejercicio-2--crear-con-una-tabla-y-verificar-cada-campo) | 30 min | `feature/s1-<tu-nombre>` |
| 2 · miércoles 28 | [3](#ejercicio-3--autenticarse-y-actualizar-con-token) y [4](#ejercicio-4--el-ciclo-de-vida-de-una-reserva) | 40 min | `feature/s2-<tu-nombre>` |
| 3 · jueves 29 | [5 · Caza de bugs](#ejercicio-5--caza-de-bugs) | 50 min | `feature/s3-<tu-nombre>` |
| 4 · viernes 30 | [Examen final](#sesión-4--examen-final) | 60 min | `feature/examen-<tu-nombre>` |

Lo que no termines en clase, termínalo antes de la siguiente sesión: el PR se revisa al inicio de la clase siguiente.

## Reglas para todos los ejercicios

- **Gherkin en español de negocio.** Nada de URLs, cabeceras ni JSON en el `.feature`: eso vive en los steps.
- **Cada escenario crea sus propios datos.** No consultes la reserva 1 ni ningún id fijo: otro alumno puede borrarla.
- **Valida código y contenido.** Un `200` sin revisar el cuerpo no prueba nada.
- **Reutiliza steps.** Antes de crear un step nuevo, busca si ya existe uno con la misma frase.
- **Etiqueta cada escenario** con el número del ejercicio (`@ejercicio1`, `@ejercicio2`...) para poder correrlo solo:

```powershell
mvn test "-Dcucumber.filter.tags=@ejercicio1"
```

### Cómo entregar (todas las sesiones)

1. Actualiza `develop` y crea tu rama: `feature/s1-<tu-nombre>` (en la sesión 2, `s2`; en la 3, `s3`).
2. Trabaja, ejecuta `mvn test` y confirma que termina en `BUILD SUCCESS`.
3. Commits en español: `feat: agrega escenario de búsqueda por apellido`.
4. Sube la rama y abre un Pull Request **a `develop`**. Adjunta en la descripción qué hiciste y qué te costó.
5. Espera la CI en verde. Si está en rojo, el PR no está terminado.

Paso a paso con comandos: [Git y GitHub · el flujo de cada ejercicio](04-git-y-github.md#6-el-flujo-de-cada-ejercicio).

---

## Sesión 1 · Del HTTP al primer escenario

### Ejercicio 1 · Buscar reservas y pedir lo que no existe

Dificultad: 1 de 5.

**Objetivo.** Escribir dos escenarios nuevos reutilizando todo lo que puedas de los steps existentes.

**Criterios de aceptación**

1. Crear una reserva a nombre de un huésped con un apellido único y encontrarla con `GET /booking?lastname=…`:
   la lista no está vacía y contiene su id.
2. Consultar `GET /booking/999999999` responde **404**.
3. Ambos escenarios llevan la etiqueta `@ejercicio1` y corren con el filtro de tags.
4. Ningún escenario depende de datos que no haya creado él mismo.

**Punto de partida**

```gherkin
@ejercicio1
Escenario: Encontrar una reserva por apellido
  Dado que existe una reserva a nombre de "Rosa" "Q-1739"
  Cuando busco reservas con el apellido "Q-1739"
  Entonces el código de respuesta es 200
  Y la lista contiene la reserva creada

@ejercicio1
Escenario: Consultar una reserva inexistente
  Cuando consulto la reserva con id 999999999
  Entonces el código de respuesta es 404
```

**Pistas**

- El filtro va como parámetro de consulta: `given().queryParam("lastname", apellido)`.
- La respuesta de la búsqueda es una lista: `body("bookingid", hasItem(contexto.idReserva))`.
- `que existe una reserva a nombre de {string}` y `... de {string} {string}` son frases distintas: Cucumber no las confunde.
- Si quieres un apellido realmente único, puedes generarlo con `System.currentTimeMillis()`.

**Aprendes:** query params, validar listas con `hasItem`, escenarios negativos.

### Ejercicio 2 · Crear con una tabla y verificar cada campo

Dificultad: 2 de 5.

**Objetivo.** Enviar todos los datos de la reserva desde una **Data Table** y comprobar que la API guardó
exactamente lo que se envió, consultándola después.

**Criterios de aceptación**

1. El paso recibe la tabla como `Map<String, String>` y arma el cuerpo con las fechas anidadas.
2. Tras crearla, un `GET` devuelve los mismos 6 valores (nombre, apellido, precio, depósito, llegada, salida).
3. Si un valor difiere, el mensaje de error dice qué campo y qué se esperaba.
4. Un *Esquema del escenario* repite la prueba con un nombre con tilde y uno con ñ.

**Punto de partida**

```gherkin
@ejercicio2
Escenario: La API guarda exactamente lo enviado
  Cuando creo una reserva con los datos:
    | firstname   | Ñusta      |
    | lastname    | Mamani     |
    | totalprice  | 480        |
    | depositpaid | true       |
    | checkin     | 2026-12-20 |
    | checkout    | 2026-12-27 |
  Y consulto esa reserva
  Entonces la reserva guardada coincide con los datos enviados
```

```java
@Cuando("creo una reserva con los datos:")
public void creoConDatos(Map<String, String> datos) { ... }
```

**Pistas**

- Guarda el `Map` en el `Contexto` para compararlo en el *Entonces*.
- La tabla llega como texto: usa `Integer.parseInt` para el precio y `Boolean.parseBoolean` para el depósito.
- Un `.body("campo", equalTo(valor))` por campo hace que el mensaje de error nombre el campo que falló.
- En un *Esquema* puedes usar `<nombre>` dentro de la tabla de datos: Cucumber lo reemplaza en cada fila de *Ejemplos*.

**Aprendes:** Data Tables, JSON anidado, conversión de tipos (`"150"` → `150`), verificar lo guardado y no solo la respuesta.

**Entrega de la sesión 1:** PR `feature/s1-<tu-nombre>` con los ejercicios 1 y 2.

---

## Sesión 2 · Autenticación y ciclo de vida completo

### Ejercicio 3 · Autenticarse y actualizar con token

Dificultad: 3 de 5.

**Objetivo.** Obtener un token con `POST /auth` (usuario `admin`, clave `password123`) y usarlo para modificar
una reserva con `PUT`. Comprobar también que sin token, o con uno falso, no se puede.

**Criterios de aceptación**

1. Un paso *Dado que estoy autenticado como administrador* guarda el token en el `Contexto`.
2. Con token, `PUT /booking/{id}` cambia el precio y responde **200** con el valor nuevo.
3. Sin token, la misma petición responde **403**. Con un token inventado, también.
4. Usuario y clave no están escritos en el código Java: se leen de `System.getProperty` o de variables de entorno.

**Punto de partida**

```gherkin
@ejercicio3 @seguridad
Escenario: El administrador actualiza el precio
  Dado que estoy autenticado como administrador
  Y que existe una reserva a nombre de "Carlos"
  Cuando actualizo el precio de esa reserva a 999
  Entonces el código de respuesta es 200
  Y el precio de la reserva es 999

@ejercicio3 @seguridad
Esquema del escenario: Nadie más puede actualizar
  Dado que existe una reserva a nombre de "Carlos"
  Cuando actualizo el precio con el token "<token>"
  Entonces el código de respuesta es 403
  Ejemplos:
    | token           |
    |                 |
    | token-inventado |
```

**Pistas**

- Restful Booker recibe el token en una cookie: `.cookie("token", token)`. Si el token está vacío, no envíes la cookie.
- `PUT` reemplaza la reserva completa: envía todos los campos, no solo el precio.
- En lugar de un `Map`, prueba con un objeto Java (`record`) para el cuerpo: Jackson lo convierte a JSON.
- Para leer la clave: `System.getProperty("api.clave")` y pásala con `mvn test "-Dapi.clave=..."`, o desde el `pom.xml`.
- Para pensar: prueba `POST /auth` con una clave incorrecta. ¿Qué código devuelve? ¿Es el que esperarías?
  Anótalo: lo usarás en el ejercicio 5.

**Aprendes:** autenticación por token, cookies, `PUT` frente a `PATCH`, pruebas negativas de seguridad, no versionar credenciales.

### Ejercicio 4 · El ciclo de vida de una reserva

Dificultad: 4 de 5.

**Objetivo.** Automatizar un flujo de punta a punta: crear → modificar parcialmente → comprobar → borrar →
confirmar que ya no existe. Y además, **limpiar** lo que creas.

**Criterios de aceptación**

1. `PATCH` cambia solo `additionalneeds`; el resto de campos queda igual (verifícalo con un `GET`).
2. Tras `DELETE`, un `GET` a ese id responde **404**.
3. Un hook `@After("@limpieza")` borra las reservas que el escenario creó, aunque el escenario falle.
4. Los steps están separados en al menos dos clases (`ReservaSteps`, `AutenticacionSteps`) que comparten el mismo `Contexto`.
5. Al menos una respuesta se valida contra un **JSON Schema** (campos obligatorios y tipos).

**Punto de partida**

```gherkin
@ejercicio4 @limpieza
Escenario: Una reserva nace, cambia y desaparece
  Dado que estoy autenticado como administrador
  Y que existe una reserva a nombre de "Elena"
  Cuando cambio sus necesidades adicionales a "Cuna"
  Entonces la reserva conserva sus demás datos
  Cuando elimino esa reserva
  Entonces el código de respuesta es 201
  Y la reserva ya no existe
```

**Pistas**

- Guarda en el `Contexto` la lista de ids que vas creando; el hook la recorre y borra cada uno.
- Si la reserva ya se borró dentro del escenario, el hook no debe fallar: no valides la respuesta del borrado de limpieza.
- Para comparar "el resto de campos", convierte la respuesta en un objeto (`extract().as(Reserva.class)`) y compáralo
  con el que enviaste: dos `record` con los mismos datos son iguales.
- JSON Schema: agrega la dependencia `io.rest-assured:json-schema-validator` (misma versión que `rest-assured`),
  guarda el esquema en `src/test/resources/schemas/` y valida con `body(matchesJsonSchemaInClasspath("schemas/reserva.json"))`.
- Este escenario tiene dos *Cuando* a propósito: es un flujo de punta a punta. En reglas de negocio, preferimos uno.

**Discusión obligatoria (escríbela en el PR).** Restful Booker responde **201 Created** al borrar. Tu prueba pasa...
¿pero es correcto? ¿Qué código esperarías tú? ¿Lo reportarías como bug o lo aceptarías como contrato?

**Aprendes:** `PATCH`, hooks condicionados por etiqueta, limpieza de datos de prueba, steps en varias clases, JSON Schema.

**Entrega de la sesión 2:** PR `feature/s2-<tu-nombre>` con los ejercicios 3 y 4.

---

## Sesión 3 · Código que escala y caza de bugs

### Ejercicio 5 · Caza de bugs

Dificultad: 5 de 5. Reto: no hay una única respuesta.

**Objetivo.** Restful Booker tiene comportamientos que ninguna API de producción debería tener: **al menos cinco**.
Encuéntralos, escribe para cada uno un escenario que **describa lo correcto** (y que hoy falle) y redacta el reporte del bug.

**Dónde buscar**

1. Reglas de negocio: ¿acepta un precio negativo? ¿Una salida anterior a la llegada?
2. Validación: ¿qué pasa si falta un campo obligatorio en el cuerpo? ¿Y si un campo trae un tipo o formato equivocado?
3. Seguridad: ¿qué código devuelve `/auth` con credenciales malas?
4. Semántica HTTP: ¿qué código devuelve un borrado exitoso? ¿Y borrar algo que no existe?

**Criterios de aceptación**

1. Al menos cinco escenarios, uno por bug, en un archivo `bugs.feature`, cada uno con la etiqueta `@bug`.
2. Cada escenario espera el comportamiento **correcto** (por ejemplo, un `400`), no el que hoy devuelve la API.
3. Los casos que comparten forma se escriben como *Esquema del escenario* (una fila por dato inválido).
4. `mvn test` sigue en verde: los `@bug` no corren por defecto (ver abajo).
5. Un archivo `BUGS.md` en la raíz del proyecto documenta cada bug.
6. El proyecto usa lo visto en la sesión: un `RequestSpecification` reutilizable (ningún step repite `contentType` ni
   `accept`), la URL leída de `-DbaseUrl`, log de petición y respuesta solo cuando una validación falla y las etiquetas
   `@smoke` y `@regresion` en los escenarios existentes.

**Cómo conviven los `@bug` con un `mvn test` en verde**

Un escenario `@bug` falla a propósito hasta que el equipo de la API corrija el bug. Para que no bloquee el pipeline,
se excluye por defecto en `src/test/resources/junit-platform.properties`:

```properties
cucumber.filter.tags=not @bug
```

Una propiedad `-D` gana a ese archivo, así que los bugs se pueden ejecutar a pedido:

```powershell
mvn test                                    # todo menos los @bug: debe quedar en verde
mvn test "-Dcucumber.filter.tags=@bug"      # solo los bugs: deben fallar, uno por bug
```

En el resumen de Maven, los escenarios excluidos aparecen como *Skipped*. Cuando un bug se corrige, su escenario pasa
a verde: se le quita `@bug` y entra a la regresión.

**Formato de `BUGS.md`** (una sección por bug)

```markdown
### BUG-01 · <título corto: qué está mal>

- **Qué hice:** método, ruta y datos enviados.
- **Qué esperaba:** código y respuesta correctos, y por qué.
- **Qué pasó:** código y respuesta reales (cópialos del log).
- **Severidad:** Alta / Media / Baja, con una frase de impacto.
- **Escenario:** nombre del escenario en bugs.feature.
```

**Pistas**

- Parte de una reserva válida y rompe **una sola** regla por caso: así sabes qué provocó el resultado.
- Para enviar un tipo equivocado (texto donde va un número) un `record` no te deja; usa un `Map`.
- Un `5xx` siempre es un bug: la API no supo manejar tu petición.
- No te quedes en el código de estado: mira también qué guardó la API (consulta la reserva después).
- Lee el log que imprime REST Assured cuando el escenario falla: es tu evidencia para el `BUGS.md`.

**Aprendes:** diseño de pruebas negativas, valores límite, redacción de bugs, etiquetas para no bloquear el pipeline.

**Entrega de la sesión 3:** PR `feature/s3-<tu-nombre>` con `bugs.feature` y `BUGS.md`.

---

## Sesión 4 · Examen final

El examen es individual, dura 60 minutos y se hace contra la API del Portal Canvia Market, en la rama
`feature/examen-<tu-nombre>`. El enunciado completo se publica en el portal al iniciar el examen.
Criterios y pesos: [sílabo · Examen final](05-silabo.md#examen-final-60-min).

Para llegar preparado: ten tu proyecto con la CI en verde y el reporte publicado, y repasa los ejercicios 3, 4 y 5.

---

Anterior: [Sílabo](05-silabo.md) · [Índice](README.md)
