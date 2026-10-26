# Sílabo · Pruebas de APIs con REST Assured + Cucumber

> Versión para imprimir: [PDF con formato Canvia](pdf/silabo-restassured-cucumber.pdf).

> Parte 1 del curso de QA Automation de Canvia. 100 % práctico: cada sesión termina con código tuyo en verde y un Pull Request.

| | |
|---|---|
| **Horario** | martes 27 a viernes 30 de octubre de 2026 · 15:00 a 17:00 (4 sesiones de 2 h) |
| **Requisito** | haber completado la instalación (`mvn test` en verde) — [paso a paso](01-instalacion-windows-paso-a-paso.md) |
| **API de práctica** | [Restful Booker](https://restful-booker.herokuapp.com/apidoc/index.html) (reservas de hotel) |
| **API del examen** | Portal Canvia Market (lo levanta el instructor; también corre en tu PC y en la CI) |
| **Entregas** | Pull Request a `develop` por cada sesión, con la CI en verde ([cómo](04-git-y-github.md#6-el-flujo-de-cada-ejercicio)) |

## Al terminar la semana sabrás

1. Leer la documentación de una API y convertirla en escenarios Gherkin en español.
2. Automatizar consultas, altas, cambios y bajas con autenticación por token.
3. Validar códigos, cuerpos, tiempos y el **esquema JSON** de las respuestas.
4. Organizar el proyecto para que crezca: especificaciones reutilizables, configuración por ambiente y etiquetas.
5. Generar **reportes** que entiende cualquiera y correr todo en un **pipeline de CI** en cada Pull Request.

## Plan de sesiones

### Sesión 1 · martes 27 — Del HTTP al primer escenario

| Tiempo | Qué hacemos |
|---|---|
| 15 min | Entorno verificado: `mvn test` en verde en todas las PC |
| 20 min | Qué es una API: petición, respuesta, verbos, códigos de estado y JSON |
| 25 min | Gherkin y el proyecto por dentro: `.feature`, steps, runner, hooks y contexto |
| 30 min | En vivo: consultar reservas con filtros y pedir una que no existe |
| 30 min | **Ejercicios 1 y 2** |

- **Ejercicio 1** · Buscar reservas por nombre y pedir un id inexistente (404).
- **Ejercicio 2** · Crear una reserva con una tabla de datos y verificar cada campo.
- **Entrega:** PR `feature/s1-<tu-nombre>`.

### Sesión 2 · miércoles 28 — Autenticación y ciclo de vida completo

| Tiempo | Qué hacemos |
|---|---|
| 10 min | Revisión de los PR de ayer: lo que más se repitió |
| 30 min | Token con `POST /auth`, cabeceras, `PUT`, `PATCH` y `DELETE` |
| 20 min | Objetos Java en lugar de mapas: serialización con Jackson |
| 20 min | Validar la forma de la respuesta con **JSON Schema** |
| 40 min | **Ejercicios 3 y 4** |

- **Ejercicio 3** · Autenticarse y actualizar una reserva con token (y fallar sin él).
- **Ejercicio 4** · Ciclo de vida: crear → consultar → actualizar → eliminar → comprobar que ya no existe.
- **Entrega:** PR `feature/s2-<tu-nombre>`.

### Sesión 3 · jueves 29 — Código que escala y caza de bugs

| Tiempo | Qué hacemos |
|---|---|
| 35 min | `RequestSpecification`, cliente de API por recurso, URL por ambiente, log solo cuando falla |
| 15 min | Esquema del escenario con casos negativos y etiquetas (`@smoke`, `@regresion`) |
| 20 min | Reportes de Cucumber: HTML, JSON y JUnit; cómo leer un fallo |
| 50 min | **Ejercicio 5 · Caza de bugs**: la API esconde al menos cinco |

- **Entrega:** PR `feature/s3-<tu-nombre>` con los escenarios y un `BUGS.md` (qué hiciste, qué esperabas, qué pasó).

### Sesión 4 · viernes 30 — Reportes, pipeline y examen final

| Tiempo | Qué hacemos |
|---|---|
| 40 min | GitHub Actions paso a paso: disparadores, caché de Maven, `mvn test`, reporte como artefacto, resumen de pruebas en el PR y publicación en GitHub Pages |
| 10 min | `@smoke` en cada PR y regresión completa programada |
| 60 min | **Examen final** sobre la API del Portal Canvia Market |
| 10 min | Cierre y siguientes pasos |

## Examen final (60 min)

Automatiza contra la API del Portal Canvia Market, en tu rama `feature/examen-<tu-nombre>`:

| Criterio | Peso |
|---|---|
| Escenarios en Gherkin claros, en español y declarativos | 20 % |
| Autenticación, CRUD y casos negativos funcionando | 30 % |
| Validaciones correctas (código, cuerpo y esquema) | 20 % |
| Pipeline en verde con el reporte publicado | 20 % |
| Bugs encontrados y documentados | 10 % |

Aprobación: 70 %. El enunciado completo se publica en el portal al iniciar el examen.

## Cómo se evalúa cada sesión

| | Cumple | Falta |
|---|---|---|
| Ejecuta | `mvn test` en verde en tu PC y en la CI | Falla o no compila |
| Se entiende | Gherkin en español de negocio, sin URLs ni JSON en el `.feature` | Pasos técnicos en el escenario |
| Valida de verdad | Comprueba código **y** contenido | Solo mira el código 200 |
| Es independiente | Cada escenario crea sus propios datos | Depende de ids fijos o del orden |

---

[Índice](README.md) · Material de clase: [presentaciones en PDF](pdf/)
