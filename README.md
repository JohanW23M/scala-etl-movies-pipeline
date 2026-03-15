
#  ETL Data Pipeline & Análisis Funcional de Películas

![Scala](https://img.shields.io/badge/Scala-DC322F?style=for-the-badge&logo=scala&logoColor=white)
![Data Engineering](https://img.shields.io/badge/Data_Engineering-ETL-blue?style=for-the-badge)
![MySQL](https://img.shields.io/badge/MySQL-005C84?style=for-the-badge&logo=mysql&logoColor=white)
![JSON](https://img.shields.io/badge/JSON-000000?style=for-the-badge&logo=json&logoColor=white)

Este repositorio contiene la arquitectura y el desarrollo de un **Pipeline ETL (Extract, Transform, Load)** diseñado para procesar, sanitizar y analizar un dataset masivo de películas. 

El proyecto está construido íntegramente utilizando el paradigma de **Programación Funcional en Scala**, garantizando la inmutabilidad de los datos, el manejo seguro de errores y la ejecución transaccional hacia una base de datos relacional.

##  Stack Tecnológico

* **Lenguaje:** Scala 3
* **Streaming & I/O:** `FS2` (Lectura eficiente y procesamiento de flujos de datos).
* **Procesamiento JSON:** `Circe` (Parseo funcional y decodificación fuertemente tipada de objetos complejos).
* **Persistencia de Datos:** `Doobie` (Capa JDBC puramente funcional) + `Cats-Effect`.
* **Motor de Base de Datos:** MySQL.

---

##  Arquitectura del Pipeline ETL

El flujo de datos está orquestado en tres fases modulares, evitando estados mutables y asegurando la reproducibilidad:

### 1. Extracción y Análisis Exploratorio (FS2)
La lectura del dataset se realiza mediante streams, permitiendo un procesamiento eficiente sin saturar la memoria RAM.
* **Limpieza de Anomalías:** Implementación de estrategias de imputación funcional. Por ejemplo, la corrección de presupuestos en `0` mediante el cálculo dinámico de promedios reales utilizando funciones de orden superior (`map`, `filter`, `foldLeft`).
* **Análisis de Frecuencia:** Agrupación y conteo de variables textuales (idiomas, estados de producción) para identificar patrones de distribución sin efectos colaterales.

### 2. Transformación y Parsing Complejo (Circe)
Uno de los mayores desafíos técnicos resueltos fue la normalización de columnas que contenían **arreglos JSON anidados como texto plano** (ej. `crew`, `cast`, `genres`, `production_companies`).
* **Decodificación Segura:** Mapeo de strings JSON hacia *Case Classes* (ej. `CrewMember`) utilizando los decodificadores de Circe.
* **Manejo Funcional de Errores:** Uso extensivo de tipos seguros como `Option` y `Either` para prevenir caídas del sistema ante JSONs corruptos o valores nulos.
* **Filtrado Lógico:** Extracción de entidades específicas anidadas (ej. filtrar únicamente a los miembros con el rol de `"Director"` dentro del arreglo `crew`).

### 3. Carga y Persistencia Relacional (Doobie)
Los datos estructurados son inyectados en una base de datos MySQL rigurosamente normalizada.
* **Diseño Transaccional:** Ejecución de sentencias SQL fuertemente tipadas y desacopladas por entidad (`loader/ProyectoIntegradorLoader.scala`).
* **Procesamiento por Lotes (Batching):** Implementación de inserciones masivas (`Update.updateMany`) para optimizar el rendimiento de red y reducir el número de conexiones.
* **Integridad Referencial:** Inserción controlada de tablas maestras, entidades principales (`pelicula`) e intermedias (relaciones N:M), previniendo duplicidad mediante cláusulas `ON DUPLICATE KEY UPDATE`.

---

##  Estructura del Proyecto

El código fuente sigue los principios de separación de responsabilidades:

```text
Proyecto_Integrador/
├── build.sbt                    <-- Dependencias (fs2, circe, doobie, cats-effect)
├── src/
│   ├── main/
│   │   ├── resources/           <-- Archivos de entrada (CSV y configuración)
│   │   └── scala/
│   │       ├── data/            <-- Lógica de extracción (CSVParser)
│   │       ├── models/          <-- Modelos del dominio y DTOs (Case Classes)
│   │       ├── loader/          <-- Capa de persistencia JDBC (Doobie)
│   │       ├── utilities/       <-- Reglas de negocio y sanitización
│   │       └── Main.scala       <-- Orquestador del pipeline (Punto de entrada)
└── README.md                    <-- Documentación principal

```

##  Características Técnicas Destacadas

1. **Ausencia de Estado Mutable:** Todas las transformaciones retornan nuevas estructuras, previniendo condiciones de carrera.
2. **Escalabilidad y Rendimiento:** El uso de streaming y operaciones *batch* permite escalar la solución a datasets de millones de registros de manera fluida.
3. **Atomicidad:** El uso de `ConnectionIO` asegura que la carga de una película completa (incluyendo todas sus tablas intermedias) se ejecute como una única transacción segura.
4. **Explotación de Datos:** Ejecución nativa de consultas analíticas (top ingresos, frecuencia de idiomas, directores) directamente desde Scala validando la carga.

```

