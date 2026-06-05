# Suite estructurada con keywords reutilizables y archivo Resource

## Metadatos

| Campo | Detalle |
|---|---|
| **Duración estimada** | 72 minutos |
| **Complejidad** | Media |
| **Nivel Bloom** | Aplicar (Apply) |
| **Módulo** | 2 — Test Cases, Keywords y Bibliotecas |
| **Laboratorio** | 02-00-01 |

---

## Descripción General

En este laboratorio construirás un proyecto Robot Framework con separación de responsabilidades: los test cases residirán en la carpeta `tests/` y las keywords reutilizables, junto con las variables compartidas, en un archivo `.resource` dentro de la carpeta `resources/`. El escenario de negocio simula operaciones básicas de un sistema de gestión de clientes de una empresa ficticia de telecomunicaciones llamada **TelecomPlus**. Aprenderás a definir los tres tipos de variables del framework, a construir keywords compuestas con parámetros y a importar el archivo Resource con ruta relativa para ejecutar la suite desde la raíz del proyecto.

---

## Objetivos de Aprendizaje

Al completar este laboratorio serás capaz de:

- [ ] Crear un archivo `.resource` que centralice keywords reutilizables y variables compartidas, separadas de los test cases
- [ ] Definir y utilizar los tres tipos de variables de Robot Framework: escalares (`${var}`), listas (`@{list}`) y diccionarios (`&{dict}`) en contextos apropiados
- [ ] Construir keywords compuestas con parámetros que encapsulen lógica de negocio reutilizable en múltiples test cases
- [ ] Importar el archivo Resource desde la suite principal usando ruta relativa y verificar que keywords y variables se resuelven correctamente

---

## Prerrequisitos

### Conocimientos

- Haber completado los laboratorios 01-00-01 y 01-00-02 del Módulo 1
- Comprensión de las cuatro secciones de un archivo `.robot` (`Settings`, `Variables`, `Test Cases`, `Keywords`)
- Conocimiento básico de Python sobre listas y diccionarios (analogía con `@{list}` y `&{dict}`)
- Familiaridad con los conceptos de BuiltIn y Collections vistos en la Lección 2.1

### Acceso y Herramientas

- Entorno virtual Python (`venv`) activado con Robot Framework 7.x instalado
- Visual Studio Code con la extensión Robot Framework Language Server
- Terminal (PowerShell/cmd en Windows, bash/zsh en macOS/Linux)
- Permisos de escritura en el directorio de trabajo del proyecto

---

## Entorno de Laboratorio

### Requisitos de Hardware

| Componente | Mínimo |
|---|---|
| Procesador | Intel Core i5 8ª gen / AMD Ryzen 5 (4 núcleos) |
| RAM | 8 GB |
| Almacenamiento libre | 500 MB para este laboratorio |
| Pantalla | 1280 × 768 (para visualizar reportes HTML) |

### Requisitos de Software

| Software | Versión |
|---|---|
| Python | 3.10 o superior |
| Robot Framework | 7.x (última estable) |
| robotframework-collections | Incluida en RF (estándar) |
| Visual Studio Code | 1.85 o superior |
| RF Language Server (extensión) | 1.12 o superior |

### Preparación del Entorno

Sigue estos pasos **antes** de comenzar las actividades del laboratorio.

**Paso 0.1 — Verificar el entorno virtual activo**

> ⚠️ **OBLIGATORIO**: Todos los laboratorios deben ejecutarse dentro de un entorno virtual Python. Nunca instales paquetes en el Python global del sistema.

```bash
# Windows (PowerShell)
# Si el venv aún no existe, créalo primero:
python -m venv venv
# Activar:
.\venv\Scripts\Activate.ps1

# Windows (cmd)
venv\Scripts\activate.bat

# macOS / Linux (bash/zsh)
python3 -m venv venv
source venv/bin/activate
```

Verifica que el prompt muestra `(venv)` al inicio. Luego comprueba las versiones:

```bash
python --version
# Esperado: Python 3.10.x o superior

robot --version
# Esperado: Robot Framework 7.x.x (Python 3.x.x ...)
```

**Paso 0.2 — Instalar dependencias necesarias**

La biblioteca Collections es estándar de Robot Framework y no requiere instalación adicional. Solo necesitas confirmar que RF está instalado:

```bash
pip install robotframework
pip show robotframework
# Verificar que la versión sea 7.x
```

**Paso 0.3 — Crear la estructura base del proyecto**

```bash
# Crear el directorio raíz del proyecto (si no existe desde laboratorios anteriores)
mkdir telecomplus_suite
cd telecomplus_suite

# Crear la estructura de carpetas
# Windows (PowerShell / cmd)
mkdir tests
mkdir resources
mkdir reports

# macOS / Linux
mkdir -p tests resources reports
```

La estructura resultante debe ser:

```
telecomplus_suite/
├── tests/
├── resources/
└── reports/
```

---

## Actividades del Laboratorio

---

### Paso 1 — Comprender la arquitectura del proyecto antes de codificar

**Objetivo:** Establecer mentalmente el modelo de separación de responsabilidades que guiará todo el laboratorio.

#### Instrucciones

1. Abre VS Code en la carpeta `telecomplus_suite`:

   ```bash
   # Desde la raíz del proyecto
   code .
   ```

2. Observa la estructura de carpetas en el explorador de VS Code. El principio de diseño es:
   - `resources/gestion_clientes.resource` → contiene **variables** y **keywords** reutilizables
   - `tests/suite_clientes.robot` → contiene únicamente **test cases** que importan el resource

3. Revisa el siguiente diagrama conceptual antes de escribir código:

   ```
   tests/suite_clientes.robot
   ┌─────────────────────────────────┐
   │ *** Settings ***                │
   │   Resource  ../resources/...   │──┐
   │                                 │  │ importa
   │ *** Test Cases ***              │  │
   │   TC1: Crear Cliente Básico     │  ▼
   │   TC2: Validar Plan Tarifario   │  resources/gestion_clientes.resource
   │   TC3: Calcular Factura         │  ┌──────────────────────────────────┐
   │   TC4: Gestionar Lista Planes   │  │ *** Variables ***                │
   │   TC5: Actualizar Datos Cliente │  │   ${EMPRESA}  TelecomPlus        │
   └─────────────────────────────────┘  │   @{PLANES_DISPONIBLES}  ...     │
                                         │   &{CLIENTE_TEMPLATE}  ...       │
                                         │                                  │
                                         │ *** Keywords ***                 │
                                         │   Crear Cliente                  │
                                         │   Validar Plan Tarifario         │
                                         │   Calcular Factura Mensual       │
                                         │   Agregar Plan A Lista           │
                                         │   Actualizar Datos Cliente       │
                                         └──────────────────────────────────┘
   ```

#### Salida Esperada

No hay archivo creado aún. Este paso es de análisis. El estudiante debe poder responder: *¿Por qué separamos keywords en un archivo `.resource` en lugar de escribirlas directamente en el `.robot`?*

> **Respuesta esperada:** Para reutilizar las mismas keywords en múltiples suites sin duplicar código, y para mantener los test cases enfocados en el *qué* (comportamiento de negocio) en lugar del *cómo* (implementación técnica).

#### Verificación

- [ ] La carpeta `telecomplus_suite/` existe con las subcarpetas `tests/`, `resources/` y `reports/`
- [ ] VS Code muestra la estructura en el explorador lateral

---

### Paso 2 — Crear el archivo Resource con variables de los tres tipos

**Objetivo:** Definir variables escalares, de lista y de diccionario que representen datos del dominio de telecomunicaciones.

#### Instrucciones

1. En VS Code, crea el archivo `resources/gestion_clientes.resource`.

2. Escribe la sección `*** Variables ***` con los tres tipos de variables:

   ```robot
   *** Variables ***
   # ─── Variables Escalares (${var}) ──────────────────────────────────────────
   # Representan un único valor: texto, número o booleano
   ${EMPRESA}              TelecomPlus
   ${VERSION_API}          v2.1
   ${IMPUESTO_PORCENTAJE}  ${0.19}
   ${LIMITE_PLANES}        ${5}

   # ─── Variables de Lista (@{list}) ───────────────────────────────────────────
   # Representan colecciones ordenadas de valores
   @{PLANES_DISPONIBLES}   Básico    Estándar    Premium    Empresarial
   @{REGIONES_COBERTURA}   Norte     Sur         Centro     Oriente     Occidente

   # ─── Variables de Diccionario (&{dict}) ─────────────────────────────────────
   # Representan pares clave-valor (como objetos/registros)
   &{CLIENTE_TEMPLATE}
   ...    nombre=Sin Asignar
   ...    documento=000000000
   ...    plan=Básico
   ...    activo=${TRUE}
   ...    deuda=${0.0}

   &{TARIFAS_PLANES}
   ...    Básico=${29900}
   ...    Estándar=${49900}
   ...    Premium=${79900}
   ...    Empresarial=${149900}
   ```

3. Guarda el archivo con `Ctrl+S` (Windows/Linux) o `Cmd+S` (macOS).

#### Salida Esperada

El archivo `resources/gestion_clientes.resource` existe y VS Code no muestra errores de sintaxis en el panel de problemas. La extensión RF Language Server debe mostrar resaltado de sintaxis para las variables.

#### Verificación

- [ ] Las variables escalares usan el prefijo `${...}` y contienen un único valor
- [ ] La variable de lista `@{PLANES_DISPONIBLES}` contiene exactamente 4 elementos
- [ ] Los diccionarios `&{CLIENTE_TEMPLATE}` y `&{TARIFAS_PLANES}` usan la sintaxis `clave=valor` con `...` para continuación de línea
- [ ] No hay errores de sintaxis reportados por VS Code

> **Nota conceptual:** En Robot Framework, `${IMPUESTO_PORCENTAJE}` con valor `${0.19}` crea una variable con tipo numérico flotante en Python. Sin las llaves internas (`0.19` sin `${}`), se crearía una cadena de texto `"0.19"`. Esta distinción es crítica para operaciones aritméticas.

---

### Paso 3 — Construir las keywords reutilizables en el archivo Resource

**Objetivo:** Definir cinco keywords con parámetros que encapsulen lógica de negocio del sistema TelecomPlus.

#### Instrucciones

1. Continúa editando `resources/gestion_clientes.resource`. Agrega la sección `*** Settings ***` al inicio (antes de `*** Variables ***`) y luego la sección `*** Keywords ***` al final:

   ```robot
   *** Settings ***
   Library    Collections
   ```

2. Agrega la sección `*** Keywords ***` al final del archivo con las cinco keywords:

   ```robot
   *** Keywords ***
   Crear Cliente
       [Documentation]    Crea un nuevo registro de cliente copiando el template
       ...                y actualizando nombre, documento y plan indicados.
       [Arguments]    ${nombre}    ${documento}    ${plan}=Básico
       # Copiar el diccionario template para no modificar el original
       &{nuevo_cliente}=    Copy Dictionary    ${CLIENTE_TEMPLATE}
       Set To Dictionary    ${nuevo_cliente}
       ...    nombre=${nombre}
       ...    documento=${documento}
       ...    plan=${plan}
       Log    Cliente creado: ${nombre} | Doc: ${documento} | Plan: ${plan}
       RETURN    &{nuevo_cliente}

   Validar Plan Tarifario
       [Documentation]    Verifica que el plan indicado existe en la lista
       ...                de planes disponibles de TelecomPlus.
       [Arguments]    ${plan}
       List Should Contain Value    ${PLANES_DISPONIBLES}    ${plan}
       Log    Plan '${plan}' es válido en ${EMPRESA} ${VERSION_API}

   Calcular Factura Mensual
       [Documentation]    Calcula el valor total de la factura aplicando impuesto.
       ...                Retorna el valor total como número.
       [Arguments]    ${plan}    ${meses}=${1}
       ${tarifa_base}=    Get From Dictionary    ${TARIFAS_PLANES}    ${plan}
       ${subtotal}=       Evaluate    ${tarifa_base} * ${meses}
       ${impuesto}=       Evaluate    ${subtotal} * ${IMPUESTO_PORCENTAJE}
       ${total}=          Evaluate    ${subtotal} + ${impuesto}
       Log    Factura ${plan} x${meses} mes(es): subtotal=${subtotal} | IVA=${impuesto} | TOTAL=${total}
       RETURN    ${total}

   Agregar Plan A Lista Activa
       [Documentation]    Agrega un plan a una lista dinámica de planes activos
       ...                y verifica que no supere el límite configurado.
       [Arguments]    ${lista_planes}    ${nuevo_plan}
       ${cantidad_actual}=    Get Length    ${lista_planes}
       Should Be True
       ...    ${cantidad_actual} < ${LIMITE_PLANES}
       ...    msg=No se pueden agregar más planes. Límite es ${LIMITE_PLANES}, actual: ${cantidad_actual}
       Append To List    ${lista_planes}    ${nuevo_plan}
       ${nueva_cantidad}=    Get Length    ${lista_planes}
       Log    Plan '${nuevo_plan}' agregado. Total planes activos: ${nueva_cantidad}
       RETURN    ${lista_planes}

   Actualizar Datos Cliente
       [Documentation]    Actualiza uno o más campos de un diccionario de cliente.
       ...                Retorna el diccionario actualizado.
       [Arguments]    ${cliente}    ${campo}    ${valor}
       Dictionary Should Contain Key    ${cliente}    ${campo}
       ...    msg=El campo '${campo}' no existe en el registro de cliente
       Set To Dictionary    ${cliente}    ${campo}=${valor}
       Log    Campo '${campo}' actualizado a '${valor}' para cliente: ${cliente}[nombre]
       RETURN    ${cliente}
   ```

3. Guarda el archivo.

#### Salida Esperada

El archivo `resources/gestion_clientes.resource` completo debe verse así en el explorador de VS Code, sin errores de sintaxis:

```
resources/
└── gestion_clientes.resource   ← ~70 líneas, sin marcadores de error
```

#### Verificación

- [ ] La keyword `Crear Cliente` tiene el parámetro `${plan}` con valor por defecto `Básico`
- [ ] `Calcular Factura Mensual` usa `Evaluate` para operaciones aritméticas
- [ ] `Agregar Plan A Lista Activa` valida el límite antes de agregar
- [ ] `Actualizar Datos Cliente` verifica que el campo existe antes de modificarlo
- [ ] Todas las keywords usan `RETURN` (sintaxis RF 4+, no `[Return]`)
- [ ] La sección `*** Settings ***` importa `Collections` en el resource

> **Nota sobre sintaxis RF 7.x:** La palabra clave `RETURN` (en mayúsculas, como bloque nativo) es la forma estándar desde Robot Framework 4+. La sintaxis antigua `[Return]` sigue funcionando pero está deprecada. En este curso usamos siempre `RETURN`.

---

### Paso 4 — Crear la suite de test cases que importa el Resource

**Objetivo:** Escribir la suite principal con cinco test cases que usen las keywords y variables del archivo Resource.

#### Instrucciones

1. Crea el archivo `tests/suite_clientes.robot`.

2. Escribe la sección `*** Settings ***` con la importación del resource usando ruta relativa:

   ```robot
   *** Settings ***
   Documentation    Suite de pruebas del sistema de gestión de clientes TelecomPlus.
   ...              Valida operaciones de creación, consulta y actualización de clientes.
   Resource         ../resources/gestion_clientes.resource
   ```

   > **Importante:** La ruta `../resources/gestion_clientes.resource` es **relativa al archivo `.robot`**, no al directorio desde donde se ejecuta `robot`. El archivo está en `tests/`, por lo tanto sube un nivel (`../`) para llegar a `resources/`.

3. Agrega la sección `*** Variables ***` con variables locales de la suite (que complementan las del resource):

   ```robot
   *** Variables ***
   # Variables locales de la suite (complementan las del resource)
   ${DOC_CLIENTE_1}    CC-10234567
   ${DOC_CLIENTE_2}    NIT-900123456
   ${DOC_CLIENTE_3}    CC-55678901
   ```

4. Escribe los cinco test cases:

   ```robot
   *** Test Cases ***
   TC-01: Crear un cliente con plan por defecto
       [Documentation]    Verifica que se puede crear un cliente con el plan
       ...                Básico (valor por defecto del parámetro).
       [Tags]    clientes    creacion
       &{cliente}=    Crear Cliente    nombre=María López    documento=${DOC_CLIENTE_1}
       Should Be Equal    ${cliente}[nombre]      María López
       Should Be Equal    ${cliente}[plan]        Básico
       Should Be Equal    ${cliente}[activo]      ${TRUE}
       Log    TC-01 completado. Cliente creado: ${cliente}

   TC-02: Crear un cliente con plan Premium y validar el plan
       [Documentation]    Crea un cliente con plan Premium y verifica que ese
       ...                plan existe en la lista de planes disponibles.
       [Tags]    clientes    creacion    planes
       &{cliente}=    Crear Cliente
       ...    nombre=Carlos Mendoza
       ...    documento=${DOC_CLIENTE_2}
       ...    plan=Premium
       Should Be Equal    ${cliente}[plan]    Premium
       # Validar que el plan asignado es un plan oficial de TelecomPlus
       Validar Plan Tarifario    ${cliente}[plan]
       Log    TC-02 completado. Plan Premium validado para: ${cliente}[nombre]

   TC-03: Calcular factura mensual y trimestral de un plan
       [Documentation]    Verifica el cálculo de factura para 1 y 3 meses
       ...                del plan Estándar, incluyendo IVA del 19%.
       [Tags]    facturacion    calculos
       # Factura de 1 mes
       ${total_mensual}=    Calcular Factura Mensual    plan=Estándar    meses=${1}
       # Estándar = 49900, con IVA 19% = 49900 * 1.19 = 59381.0
       Should Be Equal As Numbers    ${total_mensual}    ${59381.0}
       # Factura de 3 meses
       ${total_trimestral}=    Calcular Factura Mensual    plan=Estándar    meses=${3}
       # 49900 * 3 = 149700, con IVA = 149700 * 1.19 = 178143.0
       Should Be Equal As Numbers    ${total_trimestral}    ${178143.0}
       Log    TC-03 completado. Mensual: ${total_mensual} | Trimestral: ${total_trimestral}

   TC-04: Gestionar lista dinámica de planes activos de un cliente
       [Documentation]    Verifica que se pueden agregar planes a una lista
       ...                dinámica sin superar el límite configurado (${LIMITE_PLANES}).
       [Tags]    planes    listas
       # Crear lista inicial con un plan base
       @{planes_activos}=    Create List    Básico
       Log    Lista inicial: ${planes_activos}
       # Agregar planes adicionales
       ${planes_activos}=    Agregar Plan A Lista Activa    ${planes_activos}    Estándar
       ${planes_activos}=    Agregar Plan A Lista Activa    ${planes_activos}    Premium
       # Verificar que la lista contiene los planes esperados
       ${cantidad}=    Get Length    ${planes_activos}
       Should Be Equal As Integers    ${cantidad}    ${3}
       List Should Contain Value    ${planes_activos}    Estándar
       List Should Contain Value    ${planes_activos}    Premium
       Log    TC-04 completado. Planes activos: ${planes_activos}

   TC-05: Actualizar datos de un cliente existente
       [Documentation]    Crea un cliente y luego actualiza su plan y estado
       ...                usando la keyword Actualizar Datos Cliente.
       [Tags]    clientes    actualizacion
       # Crear cliente inicial
       &{cliente}=    Crear Cliente
       ...    nombre=Ana Torres
       ...    documento=${DOC_CLIENTE_3}
       ...    plan=Básico
       Should Be Equal    ${cliente}[plan]    Básico
       # Actualizar el plan del cliente
       ${cliente}=    Actualizar Datos Cliente
       ...    cliente=${cliente}
       ...    campo=plan
       ...    valor=Empresarial
       Should Be Equal    ${cliente}[plan]    Empresarial
       # Validar que el nuevo plan es oficial
       Validar Plan Tarifario    ${cliente}[plan]
       # Actualizar deuda del cliente
       ${cliente}=    Actualizar Datos Cliente
       ...    cliente=${cliente}
       ...    campo=deuda
       ...    valor=${149900}
       Should Be Equal As Numbers    ${cliente}[deuda]    ${149900}
       Log    TC-05 completado. Cliente actualizado: ${cliente}
   ```

5. Guarda el archivo.

#### Salida Esperada

```
tests/
└── suite_clientes.robot   ← ~80 líneas, sin errores de sintaxis
```

La extensión RF Language Server debe mostrar las keywords del resource con autocompletado al escribir en VS Code.

#### Verificación

- [ ] La importación `Resource ../resources/gestion_clientes.resource` usa ruta relativa correcta
- [ ] TC-01 usa la keyword `Crear Cliente` sin especificar `plan` (usa el valor por defecto)
- [ ] TC-03 usa `Should Be Equal As Numbers` para comparar valores flotantes
- [ ] TC-04 crea una lista con `Create List` (BuiltIn) antes de pasarla a la keyword del resource
- [ ] TC-05 encadena dos llamadas a `Actualizar Datos Cliente` sobre el mismo objeto
- [ ] Todos los test cases tienen `[Documentation]` y `[Tags]`

---

### Paso 5 — Ejecutar la suite desde la raíz del proyecto

**Objetivo:** Lanzar la ejecución completa de la suite y verificar que todos los test cases pasan.

#### Instrucciones

1. Abre la terminal integrada de VS Code (`Ctrl+ñ` en Windows/Linux, `Ctrl+\`` en macOS) o una terminal externa.

2. Asegúrate de estar en la **raíz del proyecto** (`telecomplus_suite/`) y de que el venv está activo:

   ```bash
   # Verificar directorio actual
   # Windows
   cd
   # macOS/Linux
   pwd
   # Debe mostrar: .../telecomplus_suite

   # Verificar que el venv está activo (debe aparecer (venv) en el prompt)
   ```

3. Ejecuta la suite con el directorio de salida apuntando a `reports/`:

   ```bash
   robot --outputdir reports tests/suite_clientes.robot
   ```

4. Observa la salida en la terminal. Debes ver algo similar a:

   ```
   ==============================================================================
   Suite Clientes :: Suite de pruebas del sistema de gestión de clientes TelecomPlus.
   ==============================================================================
   TC-01: Crear un cliente con plan por defecto                          | PASS |
   ------------------------------------------------------------------------------
   TC-02: Crear un cliente con plan Premium y validar el plan            | PASS |
   ------------------------------------------------------------------------------
   TC-03: Calcular factura mensual y trimestral de un plan               | PASS |
   ------------------------------------------------------------------------------
   TC-04: Gestionar lista dinámica de planes activos de un cliente       | PASS |
   ------------------------------------------------------------------------------
   TC-05: Actualizar datos de un cliente existente                       | PASS |
   ------------------------------------------------------------------------------
   Suite Clientes :: Suite de pruebas...                                 | PASS |
   5 tests, 5 passed, 0 failed
   ==============================================================================
   Output:  .../telecomplus_suite/reports/output.xml
   Log:     .../telecomplus_suite/reports/log.html
   Report:  .../telecomplus_suite/reports/report.html
   ```

5. Abre el reporte HTML en el navegador:

   ```bash
   # Windows
   start reports\report.html

   # macOS
   open reports/report.html

   # Linux
   xdg-open reports/report.html
   ```

#### Salida Esperada

- **5 tests, 5 passed, 0 failed** en la terminal
- El archivo `reports/report.html` se abre en el navegador mostrando todos los tests en verde
- El archivo `reports/log.html` muestra el detalle de cada keyword ejecutada con sus mensajes de `Log`

#### Verificación

- [ ] La terminal muestra `5 tests, 5 passed, 0 failed`
- [ ] Los archivos `reports/output.xml`, `reports/log.html` y `reports/report.html` fueron generados
- [ ] El reporte HTML muestra el nombre de la suite y los cinco test cases
- [ ] En `log.html`, al expandir TC-03, se ven los valores calculados: `59381.0` y `178143.0`

---

### Paso 6 — Explorar la ejecución por etiquetas y validar el Resource de forma aislada

**Objetivo:** Usar filtros de ejecución por tags para ejecutar subconjuntos de la suite y confirmar la flexibilidad del sistema de keywords.

#### Instrucciones

1. Ejecuta solo los test cases relacionados con creación de clientes (tag `creacion`):

   ```bash
   robot --outputdir reports --include creacion tests/suite_clientes.robot
   ```

   Salida esperada:

   ```
   2 tests, 2 passed, 0 failed
   ```

2. Ejecuta solo los test cases de facturación:

   ```bash
   robot --outputdir reports --include facturacion tests/suite_clientes.robot
   ```

   Salida esperada:

   ```
   1 test, 1 passed, 0 failed
   ```

3. Ejecuta todos los tests **excepto** los de cálculos:

   ```bash
   robot --outputdir reports --exclude calculos tests/suite_clientes.robot
   ```

   Salida esperada:

   ```
   4 tests, 4 passed, 0 failed
   ```

4. Ejecuta con nombre de test específico usando `--test`:

   ```bash
   robot --outputdir reports --test "TC-05*" tests/suite_clientes.robot
   ```

   Salida esperada:

   ```
   1 test, 1 passed, 0 failed
   ```

#### Salida Esperada

Cada ejecución filtra correctamente los test cases según el criterio indicado y genera un nuevo reporte en `reports/`.

#### Verificación

- [ ] El filtro `--include creacion` ejecuta exactamente TC-01 y TC-02
- [ ] El filtro `--include facturacion` ejecuta exactamente TC-03
- [ ] El filtro `--exclude calculos` ejecuta TC-01, TC-02, TC-04 y TC-05
- [ ] Todos los subconjuntos pasan sin errores

> **Concepto clave:** Los tags permiten organizar la ejecución sin modificar los archivos de prueba. Esto es esencial en pipelines CI/CD donde se ejecutan subconjuntos de pruebas según el contexto (smoke tests, regression, etc.).

---

## Validación y Pruebas Finales

Una vez completados todos los pasos, realiza esta verificación integral del proyecto.

### Lista de Verificación Final

```bash
# Desde telecomplus_suite/, ejecutar la suite completa con verbose
robot --outputdir reports --loglevel DEBUG tests/suite_clientes.robot
```

Confirma cada punto:

| Verificación | Comando / Acción | Resultado Esperado |
|---|---|---|
| Estructura de archivos | `ls -R` (Linux/Mac) o `tree` (Windows) | Ver `tests/`, `resources/`, `reports/` con sus archivos |
| Suite completa | `robot --outputdir reports tests/suite_clientes.robot` | `5 tests, 5 passed, 0 failed` |
| Reporte HTML | Abrir `reports/report.html` | Todos los tests en verde |
| Log detallado | Abrir `reports/log.html` | Mensajes de Log visibles en cada keyword |
| Variables en log | Expandir TC-01 en log.html | Ver `Cliente creado: {'nombre': 'María López', ...}` |
| Cálculo TC-03 | Expandir TC-03 en log.html | Ver `subtotal=149700 | IVA=28443.0 | TOTAL=178143.0` |

### Estructura Final del Proyecto

Verifica que la estructura de archivos sea exactamente:

```
telecomplus_suite/
├── resources/
│   └── gestion_clientes.resource
├── tests/
│   └── suite_clientes.robot
└── reports/
    ├── log.html
    ├── output.xml
    └── report.html
```

```bash
# Verificación rápida en terminal
# Windows (PowerShell)
Get-ChildItem -Recurse -File | Select-Object FullName

# macOS / Linux
find . -type f | sort
```

---

## Solución de Problemas

### Problema 1: `Resource file 'gestion_clientes.resource' does not exist`

**Síntoma:**

Al ejecutar `robot tests/suite_clientes.robot`, la terminal muestra:

```
[ ERROR ] Error in file '.../tests/suite_clientes.robot' on line 3:
Resource file '../resources/gestion_clientes.resource' does not exist.
```

**Causa:**

La ruta relativa `../resources/gestion_clientes.resource` en la sección `*** Settings ***` es relativa **al archivo `.robot`**, no al directorio de ejecución. Sin embargo, el error más común es ejecutar el comando `robot` desde dentro de la carpeta `tests/` en lugar de desde la raíz del proyecto `telecomplus_suite/`, o que el archivo `.resource` tenga un nombre diferente (mayúsculas/minúsculas incorrectas).

**Solución:**

1. Verifica que estás ejecutando desde la raíz del proyecto:

   ```bash
   # Confirmar directorio actual
   # Windows: cd    |    macOS/Linux: pwd
   # Debe terminar en: telecomplus_suite

   # Si estás en tests/, sube un nivel:
   cd ..
   ```

2. Verifica que el archivo existe con el nombre exacto:

   ```bash
   # Windows
   dir resources\

   # macOS/Linux
   ls resources/
   # Debe mostrar: gestion_clientes.resource
   ```

3. Si el nombre es correcto y el directorio también, verifica la línea de importación en `suite_clientes.robot`:

   ```robot
   # ✅ Correcto (ruta relativa al archivo .robot en tests/)
   Resource    ../resources/gestion_clientes.resource

   # ❌ Incorrecto
   Resource    resources/gestion_clientes.resource
   ```

---

### Problema 2: `ValueError` o `TypeError` en TC-03 al calcular la factura

**Síntoma:**

TC-03 falla con un error similar a:

```
TypeError: unsupported operand type(s) for *: 'str' and 'int'
```

o el test falla con:

```
49900 != 59381.0
```

**Causa:**

Las claves del diccionario `&{TARIFAS_PLANES}` en el archivo Resource fueron definidas sin el delimitador `${}` para los valores numéricos. Por ejemplo:

```robot
# ❌ Incorrecto: el valor '29900' es una cadena de texto
&{TARIFAS_PLANES}
...    Básico=29900

# ✅ Correcto: el valor ${29900} es un entero Python
&{TARIFAS_PLANES}
...    Básico=${29900}
```

Cuando Robot Framework lee `Básico=29900` (sin `${}`), almacena el string `"29900"`, y Python no puede multiplicar una cadena por un número en la keyword `Calcular Factura Mensual`.

**Solución:**

1. Abre `resources/gestion_clientes.resource`

2. Localiza la variable `&{TARIFAS_PLANES}` y asegúrate de que **todos los valores numéricos** usan `${número}`:

   ```robot
   &{TARIFAS_PLANES}
   ...    Básico=${29900}
   ...    Estándar=${49900}
   ...    Premium=${79900}
   ...    Empresarial=${149900}
   ```

3. Guarda el archivo y vuelve a ejecutar:

   ```bash
   robot --outputdir reports tests/suite_clientes.robot
   ```

> **Regla general:** En Robot Framework, cualquier valor que deba ser tratado como número (entero o flotante) en operaciones aritméticas **debe** estar envuelto en `${}`. Sin ese delimitador, todos los valores en variables son cadenas de texto.

---

## Limpieza del Entorno

Una vez finalizado el laboratorio, sigue estos pasos para dejar el entorno ordenado.

### Archivar el proyecto (recomendado)

Antes de limpiar, crea una copia de respaldo del proyecto completo. Los laboratorios posteriores del módulo construyen sobre este trabajo:

```bash
# Desde el directorio padre de telecomplus_suite/
# Windows (PowerShell)
Compress-Archive -Path telecomplus_suite -DestinationPath telecomplus_suite_lab02-00-01_backup.zip

# macOS / Linux
zip -r telecomplus_suite_lab02-00-01_backup.zip telecomplus_suite/
```

### Limpiar reportes generados (opcional)

Si deseas limpiar los reportes de las ejecuciones de prueba sin eliminar el código:

```bash
# Windows
del /Q reports\*.html reports\*.xml

# macOS / Linux
rm reports/*.html reports/*.xml
```

### Desactivar el entorno virtual

```bash
# Windows y macOS/Linux (mismo comando)
deactivate
```

> ⚠️ **No elimines** los archivos `.resource` ni `.robot`. El próximo laboratorio (02-00-02) extenderá este mismo proyecto agregando más keywords y el sistema de variables avanzado.

---

## Resumen

En este laboratorio aplicaste el principio de **separación de responsabilidades** en Robot Framework creando una arquitectura de proyecto de dos capas:

| Capa | Archivo | Contenido |
|---|---|---|
| **Recursos** | `resources/gestion_clientes.resource` | Variables de los 3 tipos + 5 keywords reutilizables |
| **Suite** | `tests/suite_clientes.robot` | 5 test cases que consumen el resource |

### Conceptos Clave Consolidados

- **Archivo `.resource`**: Contiene `*** Settings ***`, `*** Variables ***` y `*** Keywords ***`, pero **no** `*** Test Cases ***`. Es la unidad de reutilización en Robot Framework.
- **Tres tipos de variables**:
  - `${escalar}` → un valor (texto, número, booleano)
  - `@{lista}` → colección ordenada, acceso por índice `${lista}[0]`
  - `&{diccionario}` → pares clave-valor, acceso por clave `${dict}[clave]`
- **Keywords con parámetros opcionales**: `[Arguments]    ${param}=valor_defecto` permite llamar la keyword con o sin ese argumento.
- **`RETURN` nativo (RF 4+)**: Reemplaza la sintaxis antigua `[Return]` y es la forma estándar en RF 7.x.
- **Importación con ruta relativa**: La ruta en `Resource` es relativa al archivo `.robot`, no al directorio de ejecución.
- **Filtros de ejecución**: `--include`, `--exclude` y `--test` permiten ejecutar subconjuntos de la suite sin modificar el código.

### Próximos Pasos

En el laboratorio **02-00-02** profundizarás en el sistema de variables de Robot Framework, explorarás el scope de variables (local, suite, global), aprenderás a pasar variables entre keywords usando `Set Suite Variable` y extenderás el archivo Resource con keywords más complejas que usan estructuras de control (`IF`, `FOR`) nativas de RF 7.x.

### Referencias

- [Guía de usuario de Robot Framework — Archivos Resource](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#resource-and-variable-files)
- [Guía de usuario de Robot Framework — Variables](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#variables)
- [Referencia biblioteca Collections](https://robotframework.org/robotframework/latest/libraries/Collections.html)
- [Referencia biblioteca BuiltIn](https://robotframework.org/robotframework/latest/libraries/BuiltIn.html)
- [Robot Framework — Sintaxis RETURN (RF 5+)](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#returning-values-from-keywords)

---

---

# Parametrización con Setup/Teardown y filtrado por tags

## 1. Metadatos

| Campo            | Detalle                                      |
|------------------|----------------------------------------------|
| **Duración**     | 72 minutos                                   |
| **Complejidad**  | Media                                        |
| **Nivel Bloom**  | Aplicar (Apply)                              |
| **Módulo**       | 02 — Test Cases, Keywords y Bibliotecas      |
| **Laboratorio**  | 02-00-02 (Práctica 4)                        |

---

## 2. Descripción General

En este laboratorio el estudiante extiende el proyecto construido en la Práctica 3 (Lab 02-00-01) añadiendo mecanismos de control de ejecución profesionales. Se implementarán **Suite Setup**, **Suite Teardown**, **Test Setup** y **Test Teardown** para gestionar el ciclo de vida completo de la suite, usando la biblioteca **OperatingSystem** para inicializar y limpiar archivos temporales. Adicionalmente, se asignarán **tags** a cada test case para habilitar la ejecución selectiva mediante los argumentos `--include` y `--exclude`, se creará una segunda suite en un subdirectorio para demostrar la jerarquía de suites, y se implementará un **Test Template** con tabla de datos para validar múltiples entradas de forma compacta.

---

## 3. Objetivos de Aprendizaje

- [ ] Implementar `Suite Setup`, `Suite Teardown`, `Test Setup` y `Test Teardown` para gestionar precondiciones y postcondiciones de ejecución usando `OperatingSystem`
- [ ] Asignar tags significativos a los test cases y ejecutar subconjuntos de la suite usando `--include`, `--exclude` y los operadores `AND` / `OR`
- [ ] Organizar una suite jerárquica con subdirectorios que Robot Framework trate como sub-suites anidadas
- [ ] Aplicar la directiva `Test Template` para parametrizar un test case con múltiples conjuntos de datos en formato tabla
- [ ] Verificar el comportamiento de `Run Keywords` y `No Operation` dentro de las fases de setup y teardown

---

## 4. Prerequisitos

### Conocimiento previo
- Haber completado el **Lab 02-00-01** con el proyecto de suite estructurada funcional
- Comprensión de keywords reutilizables, archivos `Resource` y variables de distintos tipos (`${scalar}`, `@{list}`, `&{dict}`)
- Familiaridad básica con la terminal para crear subdirectorios y ejecutar comandos `robot`

### Acceso requerido
- Entorno virtual Python (`venv`) activado con Robot Framework 7.x instalado
- Proyecto del Lab 02-00-01 disponible en disco (o la copia de respaldo)
- Conexión a internet no requerida para este laboratorio (todo es local)

---

## 5. Entorno de Laboratorio

### Hardware mínimo recomendado

| Componente       | Mínimo                                      |
|------------------|---------------------------------------------|
| Procesador       | Intel Core i5 8ª gen / AMD Ryzen 5 (4 núcleos) |
| RAM              | 8 GB                                        |
| Almacenamiento   | 5 GB libres                                 |
| Pantalla         | 1280 × 768 para reportes HTML               |

### Software requerido

| Paquete                  | Versión mínima |
|--------------------------|----------------|
| Python                   | 3.10+          |
| Robot Framework          | 7.x            |
| Visual Studio Code       | 1.85+          |
| RF Language Server (ext) | 1.12+          |

### Verificación del entorno y activación del venv

Antes de comenzar, abre una terminal en la raíz del proyecto del Lab 02-00-01 y ejecuta:

```bash
# ── Windows (cmd) ──────────────────────────────────────────────────────────
.venv\Scripts\activate

# ── Windows (PowerShell) ───────────────────────────────────────────────────
.venv\Scripts\Activate.ps1

# ── macOS / Linux (bash/zsh) ───────────────────────────────────────────────
source .venv/bin/activate
```

Confirma que el entorno está activo y que Robot Framework está disponible:

```bash
python --version
robot --version
```

**Salida esperada (ejemplo):**
```
Python 3.11.7
Robot Framework 7.1.1 (Python 3.11.7 on win32)
```

> ⚠️ **IMPORTANTE:** Si el prompt no muestra el prefijo `(.venv)`, el entorno virtual NO está activo. No continúes hasta resolverlo.

---

## 6. Pasos del Laboratorio

---

### Paso 1 — Revisar la estructura del proyecto y planificar las extensiones

**Objetivo:** Confirmar que el proyecto del Lab 02-00-01 está completo y trazar el mapa de los nuevos archivos que se crearán.

#### Instrucciones

1. Abre una terminal en la raíz del proyecto (la carpeta que contiene `tests/`, `resources/`, `variables/`, etc.) y ejecuta:

```bash
# ── Windows ────────────────────────────────────────────────────────────────
tree /F

# ── macOS / Linux ──────────────────────────────────────────────────────────
find . -not -path './.venv/*' | sort
```

2. Confirma que existe al menos la siguiente estructura mínima del lab anterior:

```
proyecto-telecom/
├── tests/
│   └── facturacion_suite.robot
├── resources/
│   └── telecom_keywords.resource
├── variables/
│   └── config_vars.robot
└── results/
```

3. Crea los nuevos directorios que se necesitarán en este laboratorio:

```bash
# ── Windows (cmd) ──────────────────────────────────────────────────────────
mkdir tests\clientes
mkdir temp_data

# ── macOS / Linux ──────────────────────────────────────────────────────────
mkdir -p tests/clientes
mkdir -p temp_data
```

4. La estructura objetivo al finalizar este laboratorio será:

```
proyecto-telecom/
├── tests/
│   ├── facturacion_suite.robot       ← modificado en este lab
│   └── clientes/                     ← nuevo subdirectorio (sub-suite)
│       └── clientes_suite.robot      ← nuevo archivo
├── resources/
│   └── telecom_keywords.resource     ← modificado en este lab
├── variables/
│   └── config_vars.robot
├── temp_data/                        ← creado por Suite Setup
└── results/
```

#### Salida esperada
Los directorios `tests/clientes/` y `temp_data/` aparecen en el árbol del proyecto.

#### Verificación
```bash
# ── Windows ────────────────────────────────────────────────────────────────
if exist tests\clientes echo OK
if exist temp_data echo OK

# ── macOS / Linux ──────────────────────────────────────────────────────────
test -d tests/clientes && echo OK
test -d temp_data && echo OK
```

---

### Paso 2 — Actualizar el archivo de variables con rutas de archivos temporales

**Objetivo:** Centralizar en `variables/config_vars.robot` las rutas que usarán los setup/teardown para no repetir strings literales.

#### Instrucciones

1. Abre `variables/config_vars.robot` en VS Code.

2. Añade las siguientes variables al final del bloque `*** Variables ***` existente (conserva todo lo que ya tenías del lab anterior):

```robot
# ── Rutas para Setup/Teardown ──────────────────────────────────────────────
${TEMP_DIR}           ${CURDIR}/../temp_data
${LOG_INIT_FILE}      ${TEMP_DIR}/suite_init.log
${LOG_BILLING_FILE}   ${TEMP_DIR}/billing_test.log
${LOG_CUSTOMER_FILE}  ${TEMP_DIR}/customer_test.log

# ── Datos de prueba para Test Template ────────────────────────────────────
${PLAN_BASICO}        Plan Básico 50MB
${PLAN_ESTANDAR}      Plan Estándar 200MB
${PLAN_PREMIUM}       Plan Premium 1GB
```

3. Guarda el archivo (`Ctrl+S` / `Cmd+S`).

> **Nota:** `${CURDIR}` es una variable automática de Robot Framework que apunta al directorio del archivo `.robot` o `.resource` que la usa. Cuando se use desde `tests/`, resolverá correctamente hacia `temp_data/` en la raíz del proyecto.

#### Salida esperada
El archivo `config_vars.robot` contiene las nuevas variables sin errores de sintaxis (VS Code no muestra subrayados rojos con el Language Server activo).

#### Verificación
Ejecuta una comprobación rápida de sintaxis:
```bash
python -m robot --dryrun --nostatusrc variables/config_vars.robot 2>&1 | head -5
```
El resultado debe indicar `0 tests, 0 passed` sin errores de parseo.

---

### Paso 3 — Implementar keywords de Setup/Teardown en el archivo Resource

**Objetivo:** Añadir al archivo `resources/telecom_keywords.resource` las keywords que gestionarán el ciclo de vida de la suite y de cada test case.

#### Instrucciones

1. Abre `resources/telecom_keywords.resource` en VS Code.

2. Asegúrate de que la sección `*** Settings ***` importa `OperatingSystem` y las variables:

```robot
*** Settings ***
Library     OperatingSystem
Library     Collections
Resource    ../variables/config_vars.robot
```

3. Al final de la sección `*** Keywords ***` existente, añade las siguientes keywords nuevas:

```robot
# ══════════════════════════════════════════════════════════════════════════
# KEYWORDS DE SUITE SETUP / TEARDOWN
# ══════════════════════════════════════════════════════════════════════════

Inicializar Entorno De Suite
    [Documentation]    Crea el directorio temporal y el archivo de log de
    ...                inicialización. Se invoca como Suite Setup.
    Log    Iniciando Suite Setup de Telecom    console=True
    Create Directory    ${TEMP_DIR}
    Directory Should Exist    ${TEMP_DIR}
    ${timestamp}=    Get Time    format=%Y-%m-%d %H:%M:%S
    Create File    ${LOG_INIT_FILE}
    ...    Suite iniciada: ${timestamp}\nEntorno: TEST\nProyecto: Telecom-Lab\n
    Log    Directorio temporal creado: ${TEMP_DIR}
    Log    Archivo de inicialización creado: ${LOG_INIT_FILE}

Limpiar Entorno De Suite
    [Documentation]    Elimina los archivos temporales generados durante la
    ...                suite. Se invoca como Suite Teardown.
    Log    Iniciando Suite Teardown de Telecom    console=True
    Run Keyword And Ignore Error    Remove File    ${LOG_INIT_FILE}
    Run Keyword And Ignore Error    Remove File    ${LOG_BILLING_FILE}
    Run Keyword And Ignore Error    Remove File    ${LOG_CUSTOMER_FILE}
    Log    Archivos temporales eliminados correctamente

# ══════════════════════════════════════════════════════════════════════════
# KEYWORDS DE TEST SETUP / TEARDOWN
# ══════════════════════════════════════════════════════════════════════════

Preparar Test De Facturación
    [Documentation]    Crea el archivo de log para el test de facturación
    ...                y registra el inicio del test.
    [Arguments]    ${nombre_test}=Test de Facturación
    ${timestamp}=    Get Time    format=%Y-%m-%d %H:%M:%S
    Create File    ${LOG_BILLING_FILE}
    ...    Test iniciado: ${nombre_test}\nTimestamp: ${timestamp}\n
    Log    Test Setup ejecutado para: ${nombre_test}

Finalizar Test De Facturación
    [Documentation]    Registra el resultado en el log y realiza limpieza
    ...                post-test. Usa No Operation si no hay nada que limpiar.
    Log    Test Teardown ejecutado — registrando resultado
    ${existe}=    Run Keyword And Return Status
    ...    File Should Exist    ${LOG_BILLING_FILE}
    IF    ${existe}
        ${contenido}=    Get File    ${LOG_BILLING_FILE}
        Log    Contenido del log de billing:\n${contenido}
    ELSE
        No Operation
    END

Preparar Test De Clientes
    [Documentation]    Inicializa el contexto para tests de clientes.
    Log    Preparando contexto de test de clientes

Finalizar Test De Clientes
    [Documentation]    Limpieza post-test para tests de clientes.
    Run Keyword And Ignore Error    Remove File    ${LOG_CUSTOMER_FILE}
    Log    Contexto de clientes limpiado

# ══════════════════════════════════════════════════════════════════════════
# KEYWORDS DE NEGOCIO (para Test Template)
# ══════════════════════════════════════════════════════════════════════════

Validar Plan De Telecomunicaciones
    [Documentation]    Verifica que el nombre del plan contiene la palabra
    ...                clave esperada y que la velocidad es válida.
    [Arguments]    ${nombre_plan}    ${velocidad_mb}    ${precio_usd}
    Log    Validando plan: ${nombre_plan} | ${velocidad_mb} MB | $${precio_usd}
    Should Contain    ${nombre_plan}    Plan
    ...    msg=El nombre del plan debe comenzar con 'Plan'
    Should Be True    ${velocidad_mb} > 0
    ...    msg=La velocidad debe ser mayor que cero
    Should Be True    ${precio_usd} > 0
    ...    msg=El precio debe ser mayor que cero
    Log    Plan '${nombre_plan}' validado exitosamente
```

4. Guarda el archivo.

#### Salida esperada
El archivo `telecom_keywords.resource` contiene las 6 nuevas keywords sin errores de sintaxis.

#### Verificación
```bash
python -m robot --dryrun --nostatusrc resources/telecom_keywords.resource 2>&1 | tail -3
```
Debe aparecer `0 tests, 0 passed` sin mensajes de error.

---

### Paso 4 — Modificar `facturacion_suite.robot` con Setup/Teardown y Tags

**Objetivo:** Actualizar la suite principal para incorporar `Suite Setup`, `Suite Teardown`, `Test Setup`, `Test Teardown` y asignar tags a cada test case.

#### Instrucciones

1. Abre `tests/facturacion_suite.robot` en VS Code.

2. Reemplaza (o actualiza) el contenido completo del archivo con lo siguiente. Conserva los test cases que ya tenías del lab anterior; el ejemplo muestra la estructura completa con test cases representativos:

```robot
*** Settings ***
Documentation       Suite de pruebas de Facturación — Telecom Lab
...                 Módulo 02 | Lab 02-00-02
...                 Demuestra Suite Setup/Teardown, Test Setup/Teardown y Tags.
Library             OperatingSystem
Library             Collections
Resource            ../resources/telecom_keywords.resource
Resource            ../variables/config_vars.robot

# ── Control de ejecución a nivel de suite ─────────────────────────────────
Suite Setup         Inicializar Entorno De Suite
Suite Teardown      Limpiar Entorno De Suite

# ── Setup/Teardown por defecto para TODOS los tests de esta suite ──────────
Test Setup          Preparar Test De Facturación
Test Teardown       Finalizar Test De Facturación


*** Variables ***
${DESCUENTO_PROMO}      15
${UMBRAL_CREDITO}       100


*** Test Cases ***

# ──────────────────────────────────────────────────────────────────────────
# TESTS DE HUMO (smoke) — Validaciones rápidas de sanidad
# ──────────────────────────────────────────────────────────────────────────

El sistema de facturación responde correctamente
    [Documentation]    Verifica que las variables de configuración básica
    ...                están disponibles y tienen valores válidos.
    [Tags]    smoke    critical
    Log    Verificando configuración básica del sistema de facturación
    Should Not Be Empty    ${DESCUENTO_PROMO}
    Should Be True    ${DESCUENTO_PROMO} > 0
    Log    Configuración básica validada — descuento: ${DESCUENTO_PROMO}%

El directorio temporal fue creado por Suite Setup
    [Documentation]    Confirma que el Suite Setup creó correctamente
    ...                el directorio de trabajo temporal.
    [Tags]    smoke    infrastructure
    Directory Should Exist    ${TEMP_DIR}
    File Should Exist         ${LOG_INIT_FILE}
    ${contenido}=    Get File    ${LOG_INIT_FILE}
    Should Contain    ${contenido}    Suite iniciada
    Log    Directorio temporal verificado: ${TEMP_DIR}

# ──────────────────────────────────────────────────────────────────────────
# TESTS DE FACTURACIÓN (billing) — Lógica de negocio de facturación
# ──────────────────────────────────────────────────────────────────────────

Calcular factura con descuento promocional
    [Documentation]    Aplica un descuento del ${DESCUENTO_PROMO}% y verifica
    ...                que el monto final es correcto.
    [Tags]    smoke    billing    regression
    ${monto_original}=    Set Variable    ${200.00}
    ${descuento}=         Evaluate    ${monto_original} * ${DESCUENTO_PROMO} / 100
    ${monto_final}=       Evaluate    ${monto_original} - ${descuento}
    Should Be True    ${monto_final} < ${monto_original}
    ...    msg=El monto con descuento debe ser menor al original
    Should Be True    ${monto_final} == 170.0
    ...    msg=Esperado 170.0, obtenido: ${monto_final}
    Log    Factura calculada: original=$${monto_original} | final=$${monto_final}

Verificar umbral de crédito para cliente moroso
    [Documentation]    Confirma que un cliente con deuda superior al umbral
    ...                queda marcado como bloqueado.
    [Tags]    billing    regression    critical
    ${deuda_cliente}=    Set Variable    ${150}
    Should Be True    ${deuda_cliente} > ${UMBRAL_CREDITO}
    ...    msg=La deuda ${deuda_cliente} debería superar el umbral ${UMBRAL_CREDITO}
    Log    Cliente con deuda ${deuda_cliente} supera umbral — acción: bloquear

Generar resumen de facturación mensual
    [Documentation]    Construye un diccionario de resumen y valida sus campos.
    [Tags]    billing    regression    low
    &{resumen}=    Create Dictionary
    ...    mes=Enero
    ...    total_facturas=${42}
    ...    monto_total=${85000.00}
    ...    moneda=USD
    Dictionary Should Contain Key    ${resumen}    mes
    Dictionary Should Contain Key    ${resumen}    total_facturas
    Dictionary Should Contain Key    ${resumen}    monto_total
    ${total}=    Get From Dictionary    ${resumen}    total_facturas
    Should Be True    ${total} > 0
    Log    Resumen de ${resumen}[mes]: ${total} facturas por $${resumen}[monto_total]

# ──────────────────────────────────────────────────────────────────────────
# TEST CON TEMPLATE — Validación parametrizada de planes
# ──────────────────────────────────────────────────────────────────────────

Validar catálogo de planes de telecomunicaciones
    [Documentation]    Utiliza Test Template para validar múltiples planes
    ...                con distintos valores de velocidad y precio.
    [Tags]    billing    smoke    regression
    [Template]    Validar Plan De Telecomunicaciones
    # nombre_plan              velocidad_mb    precio_usd
    Plan Básico 50MB           50              19.99
    Plan Estándar 200MB        200             39.99
    Plan Premium 1GB           1024            79.99

# ──────────────────────────────────────────────────────────────────────────
# TEST DE BAJA PRIORIDAD — Verificación auxiliar
# ──────────────────────────────────────────────────────────────────────────

Verificar lista de regiones de cobertura
    [Documentation]    Confirma que la lista de regiones contiene los valores
    ...                esperados para el sistema de facturación regional.
    [Tags]    billing    low
    @{regiones}=    Create List    Norte    Sur    Este    Oeste    Centro
    List Should Contain Value    ${regiones}    Norte
    List Should Contain Value    ${regiones}    Sur
    ${total_regiones}=    Get Length    ${regiones}
    Should Be True    ${total_regiones} == 5
    Log    Regiones validadas: ${regiones}
```

3. Guarda el archivo.

> **Nota sobre `Test Setup` / `Test Teardown` a nivel de suite vs. test case individual:**
> Las directivas `Test Setup` y `Test Teardown` en `*** Settings ***` aplican a **todos** los tests de la suite. Si un test individual necesita un setup diferente, puede sobreescribirlo con `[Setup]` y `[Teardown]` dentro del propio test case. En este laboratorio todos los tests usan el mismo setup/teardown de facturación para simplificar.

#### Salida esperada
VS Code no muestra errores de sintaxis y el archivo tiene exactamente 6 test cases.

#### Verificación
```bash
python -m robot --dryrun --nostatusrc tests/facturacion_suite.robot 2>&1 | tail -5
```
Salida esperada:
```
6 tests, 6 passed, 0 failed
```

---

### Paso 5 — Crear la sub-suite de clientes en el subdirectorio

**Objetivo:** Demostrar la jerarquía de suites creando `tests/clientes/clientes_suite.robot` con sus propios tags y setup/teardown independientes.

#### Instrucciones

1. Crea el archivo `tests/clientes/clientes_suite.robot` con el siguiente contenido:

```robot
*** Settings ***
Documentation       Sub-suite de pruebas de Clientes — Telecom Lab
...                 Módulo 02 | Lab 02-00-02
...                 Demuestra jerarquía de suites mediante subdirectorios.
Library             OperatingSystem
Library             Collections
Resource            ../../resources/telecom_keywords.resource
Resource            ../../variables/config_vars.robot

# ── Control de ejecución independiente para esta sub-suite ────────────────
Suite Setup         Preparar Test De Clientes
Suite Teardown      Finalizar Test De Clientes


*** Variables ***
${MAX_CLIENTES_ACTIVOS}     1000
${SEGMENTO_RESIDENCIAL}     residencial
${SEGMENTO_EMPRESARIAL}     empresarial


*** Test Cases ***

Verificar registro de cliente residencial nuevo
    [Documentation]    Simula el registro de un cliente residencial y valida
    ...                que los datos obligatorios están presentes.
    [Tags]    customer    smoke    critical
    &{cliente}=    Create Dictionary
    ...    id=CLI-00123
    ...    nombre=María López
    ...    segmento=${SEGMENTO_RESIDENCIAL}
    ...    plan=Plan Básico 50MB
    ...    activo=${TRUE}
    Dictionary Should Contain Key    ${cliente}    id
    Dictionary Should Contain Key    ${cliente}    nombre
    Dictionary Should Contain Key    ${cliente}    segmento
    ${segmento}=    Get From Dictionary    ${cliente}    segmento
    Should Be Equal    ${segmento}    ${SEGMENTO_RESIDENCIAL}
    Log    Cliente registrado: ${cliente}[nombre] | Segmento: ${segmento}

Verificar límite de clientes activos por segmento
    [Documentation]    Confirma que el contador de clientes no supera el
    ...                máximo permitido para el segmento empresarial.
    [Tags]    customer    regression
    ${clientes_empresariales}=    Set Variable    ${248}
    Should Be True    ${clientes_empresariales} <= ${MAX_CLIENTES_ACTIVOS}
    ...    msg=Se superó el límite de ${MAX_CLIENTES_ACTIVOS} clientes activos
    Log    Clientes empresariales activos: ${clientes_empresariales} (límite: ${MAX_CLIENTES_ACTIVOS})

Validar campos obligatorios de alta de cliente
    [Documentation]    Usa una lista de campos requeridos para verificar
    ...                que ninguno está vacío en el formulario de alta.
    [Tags]    customer    regression    low
    @{campos_requeridos}=    Create List
    ...    nombre    apellido    documento    email    telefono    plan
    @{datos_cliente}=    Create List
    ...    Carlos    Mendoza    DNI-45678    cmendoza@email.com    555-0198    Plan Estándar 200MB
    ${longitud_campos}=    Get Length    ${campos_requeridos}
    ${longitud_datos}=     Get Length    ${datos_cliente}
    Should Be Equal As Integers    ${longitud_campos}    ${longitud_datos}
    ...    msg=El número de campos requeridos debe coincidir con los datos provistos
    Log    Validación de ${longitud_campos} campos obligatorios completada

Buscar cliente por ID en lista de activos
    [Documentation]    Simula una búsqueda de cliente en la lista de IDs
    ...                activos del sistema.
    [Tags]    customer    smoke
    @{ids_activos}=    Create List
    ...    CLI-00120    CLI-00121    CLI-00122    CLI-00123    CLI-00124
    ${id_buscado}=    Set Variable    CLI-00123
    List Should Contain Value    ${ids_activos}    ${id_buscado}
    ...    msg=El cliente ${id_buscado} no fue encontrado en el sistema
    Log    Cliente ${id_buscado} encontrado en el sistema de clientes activos
```

2. Guarda el archivo.

> **¿Cómo trata Robot Framework los subdirectorios?**
> Cuando se ejecuta `robot tests/`, Robot Framework recorre recursivamente todos los subdirectorios y trata cada directorio como una **sub-suite** cuyo nombre es el del directorio. Así, `tests/clientes/` se convierte en la sub-suite `Clientes` dentro de la suite raíz `Tests`. No se requiere ninguna configuración adicional.

#### Salida esperada
El archivo `tests/clientes/clientes_suite.robot` existe y contiene 4 test cases.

#### Verificación
```bash
python -m robot --dryrun --nostatusrc tests/clientes/clientes_suite.robot 2>&1 | tail -5
```
Salida esperada:
```
4 tests, 4 passed, 0 failed
```

---

### Paso 6 — Ejecutar la suite completa y verificar la jerarquía

**Objetivo:** Ejecutar ambas suites juntas desde el directorio `tests/` para observar cómo Robot Framework construye la jerarquía automáticamente.

#### Instrucciones

1. Desde la raíz del proyecto, ejecuta la suite completa:

```bash
robot --outputdir results tests/
```

2. Observa la salida en consola. Deberías ver la jerarquía de suites:

```
==============================================================================
Tests
==============================================================================
Tests.Facturacion Suite
==============================================================================
...
Tests.Clientes
==============================================================================
Tests.Clientes.Clientes Suite
==============================================================================
...
```

3. Una vez finalizada la ejecución, abre el reporte HTML:

```bash
# ── Windows ────────────────────────────────────────────────────────────────
start results\report.html

# ── macOS ──────────────────────────────────────────────────────────────────
open results/report.html

# ── Linux ──────────────────────────────────────────────────────────────────
xdg-open results/report.html
```

4. En el reporte, navega a la vista de **Statistics by Tag** y confirma que aparecen los tags: `smoke`, `billing`, `customer`, `regression`, `critical`, `low`, `infrastructure`.

#### Salida esperada
```
==============================================================================
Tests                                                                         
==============================================================================
Tests.Facturacion Suite                                                       
==============================================================================
El sistema de facturación responde correctamente              | PASS |
El directorio temporal fue creado por Suite Setup             | PASS |
Calcular factura con descuento promocional                    | PASS |
Verificar umbral de crédito para cliente moroso               | PASS |
Generar resumen de facturación mensual                        | PASS |
Validar catálogo de planes de telecomunicaciones              | PASS |
Verificar lista de regiones de cobertura                      | PASS |
Tests.Facturacion Suite                                       | PASS |
7 tests, 7 passed, 0 failed
==============================================================================
Tests.Clientes                                                                
==============================================================================
Tests.Clientes.Clientes Suite                                                 
==============================================================================
Verificar registro de cliente residencial nuevo               | PASS |
Verificar límite de clientes activos por segmento             | PASS |
Validar campos obligatorios de alta de cliente                | PASS |
Buscar cliente por ID en lista de activos                     | PASS |
Tests.Clientes.Clientes Suite                                 | PASS |
4 tests, 4 passed, 0 failed
==============================================================================
Tests                                                         | PASS |
11 tests, 11 passed, 0 failed
==============================================================================
```

#### Verificación
```bash
# Verificar que el archivo de output existe y tiene el resultado correcto
python -c "
import xml.etree.ElementTree as ET
tree = ET.parse('results/output.xml')
root = tree.getroot()
stats = root.find('.//total/stat')
print('Total tests:', stats.get('pass'), 'passed,', stats.get('fail'), 'failed')
"
```

---

### Paso 7 — Ejecutar subconjuntos de tests con `--include` y `--exclude`

**Objetivo:** Demostrar el filtrado de ejecución usando tags simples, combinaciones con `AND` / `OR` y exclusiones con `--exclude`.

#### Instrucciones

Ejecuta cada uno de los siguientes comandos y observa cuántos tests se ejecutan en cada caso:

**7a. Solo tests con tag `smoke`:**
```bash
robot --include smoke --outputdir results/smoke tests/
```
Resultado esperado: **5 tests** (los que tienen el tag `smoke` en ambas suites).

**7b. Solo tests con tag `billing`:**
```bash
robot --include billing --outputdir results/billing tests/
```
Resultado esperado: **5 tests** de la suite de facturación.

**7c. Tests con tag `smoke` AND `billing` (deben tener AMBOS tags):**
```bash
robot --include smokeANDbilling --outputdir results/smoke_billing tests/
```
Resultado esperado: **3 tests** (los que tienen simultáneamente `smoke` y `billing`).

**7d. Tests con tag `smoke` OR `customer` (cualquiera de los dos):**
```bash
robot --include smoke --include customer --outputdir results/smoke_or_customer tests/
```
Resultado esperado: **8 tests** (unión de ambos conjuntos sin duplicados).

**7e. Todos los tests EXCEPTO los de baja prioridad (`low`):**
```bash
robot --exclude low --outputdir results/no_low tests/
```
Resultado esperado: **9 tests** (excluye los 2 tests con tag `low`).

**7f. Tests `critical` de cualquier suite:**
```bash
robot --include critical --outputdir results/critical tests/
```
Resultado esperado: **3 tests** marcados como `critical`.

> **Referencia rápida de sintaxis de filtros:**
>
> | Comando | Semántica |
> |---------|-----------|
> | `--include tagA` | Tests que tienen `tagA` |
> | `--include tagANDtagB` | Tests que tienen `tagA` Y `tagB` |
> | `--include tagA --include tagB` | Tests que tienen `tagA` O `tagB` |
> | `--exclude tagA` | Tests que NO tienen `tagA` |
> | `--include tagA --exclude tagB` | Tests con `tagA` pero sin `tagB` |

#### Salida esperada (ejemplo para comando 7c)
```
==============================================================================
Tests
==============================================================================
...
3 tests, 3 passed, 0 failed
==============================================================================
Output:  .../results/smoke_billing/output.xml
```

#### Verificación
Confirma que el directorio `results/` contiene los subdirectorios de cada ejecución:
```bash
# ── Windows ────────────────────────────────────────────────────────────────
dir results /B

# ── macOS / Linux ──────────────────────────────────────────────────────────
ls results/
```

---

### Paso 8 — Verificar el comportamiento del Test Template

**Objetivo:** Inspeccionar en detalle el log del test con `Test Template` para entender cómo Robot Framework itera sobre las filas de datos.

#### Instrucciones

1. Ejecuta únicamente el test de template usando su nombre exacto:

```bash
robot --test "Validar catálogo de planes de telecomunicaciones" --outputdir results/template tests/facturacion_suite.robot
```

2. Abre `results/template/log.html` en el navegador.

3. Expande el test case `Validar catálogo de planes de telecomunicaciones` en el log. Deberías ver **3 iteraciones** del template, cada una con sus propios argumentos:

```
Validar catálogo de planes de telecomunicaciones
  ├── Validar Plan De Telecomunicaciones  Plan Básico 50MB  50  19.99     → PASS
  ├── Validar Plan De Telecomunicaciones  Plan Estándar 200MB  200  39.99 → PASS
  └── Validar Plan De Telecomunicaciones  Plan Premium 1GB  1024  79.99   → PASS
```

4. Ahora añade una **cuarta fila** al template con un valor intencionalmente incorrecto para ver cómo falla una iteración individual sin detener las demás. Edita el test case en `facturacion_suite.robot`:

```robot
Validar catálogo de planes de telecomunicaciones
    [Documentation]    ...
    [Tags]    billing    smoke    regression
    [Template]    Validar Plan De Telecomunicaciones
    # nombre_plan              velocidad_mb    precio_usd
    Plan Básico 50MB           50              19.99
    Plan Estándar 200MB        200             39.99
    Plan Premium 1GB           1024            79.99
    Tarifa Especial            -10             0.00      # ← valor inválido
```

5. Ejecuta nuevamente:
```bash
robot --test "Validar catálogo de planes de telecomunicaciones" --outputdir results/template_fail tests/facturacion_suite.robot
```

6. Observa que **las 3 primeras iteraciones pasan** y solo la cuarta falla. El test case completo se marca como FAIL, pero el log muestra exactamente cuál iteración falló y por qué.

7. **Restaura** el test case a su versión original (sin la cuarta fila) antes de continuar.

#### Salida esperada (con la fila inválida)
```
Validar catálogo de planes de telecomunicaciones             | FAIL |
Several template iterations failed:
  Round 4: 'La velocidad debe ser mayor que cero'
```

#### Verificación
Después de restaurar el archivo, ejecuta:
```bash
robot --test "Validar catálogo de planes de telecomunicaciones" --outputdir results/template_ok tests/facturacion_suite.robot
```
Resultado: `1 test, 1 passed, 0 failed`.

---

### Paso 9 — Verificar el ciclo completo de Setup/Teardown con evidencias

**Objetivo:** Confirmar que los archivos temporales son creados por el Suite Setup y eliminados por el Suite Teardown, validando el ciclo completo.

#### Instrucciones

1. Antes de ejecutar, confirma que `temp_data/` está vacío (o no existe):

```bash
# ── Windows ────────────────────────────────────────────────────────────────
if exist temp_data\suite_init.log (echo "EXISTE - limpiar antes de probar") else (echo "OK - directorio limpio")

# ── macOS / Linux ──────────────────────────────────────────────────────────
[ -f temp_data/suite_init.log ] && echo "EXISTE - limpiar antes de probar" || echo "OK - directorio limpio"
```

2. Ejecuta **solo** el Suite Setup y los tests de smoke para observar la creación de archivos. Usa `--include smoke` para ejecutar rápido:

```bash
robot --include smoke --outputdir results/setup_teardown_test tests/facturacion_suite.robot
```

3. **Inmediatamente después** de que termine la ejecución (antes de que el Suite Teardown elimine los archivos), el log habrá registrado su creación. Revisa el log HTML:

```bash
# ── Windows ────────────────────────────────────────────────────────────────
start results\setup_teardown_test\log.html

# ── macOS ──────────────────────────────────────────────────────────────────
open results/setup_teardown_test/log.html
```

4. En el log, busca la sección **Suite Setup** y confirma que aparece:
   - `Iniciando Suite Setup de Telecom`
   - `Directorio temporal creado: .../temp_data`
   - `Archivo de inicialización creado: .../temp_data/suite_init.log`

5. Busca la sección **Suite Teardown** y confirma que aparece:
   - `Iniciando Suite Teardown de Telecom`
   - `Archivos temporales eliminados correctamente`

6. Verifica en el sistema de archivos que `temp_data/suite_init.log` ya **no existe** (fue eliminado por el Teardown):

```bash
# ── Windows ────────────────────────────────────────────────────────────────
if exist temp_data\suite_init.log (echo "ERROR: el teardown no eliminó el archivo") else (echo "OK: teardown funcionó correctamente")

# ── macOS / Linux ──────────────────────────────────────────────────────────
[ -f temp_data/suite_init.log ] && echo "ERROR: el teardown no eliminó el archivo" || echo "OK: teardown funcionó correctamente"
```

#### Salida esperada
```
OK: teardown funcionó correctamente
```

#### Verificación
Abre `results/setup_teardown_test/log.html` y confirma que las secciones de Setup y Teardown están marcadas en verde (PASS).

---

## 7. Validación y Pruebas Finales

Ejecuta la batería de validación completa para confirmar que todo el laboratorio funciona correctamente:

```bash
# ── 1. Ejecución completa con todos los tests ──────────────────────────────
robot --outputdir results/final_validation tests/

# ── 2. Filtro smoke AND critical ──────────────────────────────────────────
robot --include smokeANDcritical --outputdir results/final_smoke_critical tests/

# ── 3. Solo suite de clientes ─────────────────────────────────────────────
robot --outputdir results/final_customers tests/clientes/

# ── 4. Excluir tests de baja prioridad ────────────────────────────────────
robot --exclude low --outputdir results/final_no_low tests/
```

### Tabla de resultados esperados

| Comando                             | Tests esperados | Resultado esperado |
|-------------------------------------|-----------------|--------------------|
| `robot tests/`                      | 11              | 11 passed, 0 failed |
| `--include smokeANDcritical`        | 3               | 3 passed, 0 failed  |
| `robot tests/clientes/`             | 4               | 4 passed, 0 failed  |
| `--exclude low`                     | 9               | 9 passed, 0 failed  |

### Checklist de validación

- [ ] El directorio `results/final_validation/` contiene `output.xml`, `log.html` y `report.html`
- [ ] En `report.html`, la sección **Statistics by Tag** muestra todos los tags definidos
- [ ] El test `Validar catálogo de planes de telecomunicaciones` muestra 3 iteraciones en el log
- [ ] El Suite Setup crea `temp_data/` y el Suite Teardown lo limpia (verificado en el log)
- [ ] La sub-suite `Tests.Clientes` aparece anidada bajo `Tests` en el reporte
- [ ] Ningún test muestra estado FAIL en la ejecución final

---

## 8. Solución de Problemas

### Problema 1: `Variable '${TEMP_DIR}' not found` al ejecutar la suite

**Síntoma:**
```
Variable '${TEMP_DIR}' not found.
```
El Suite Setup falla en la primera línea con un error de variable no encontrada.

**Causa:**
El archivo `resources/telecom_keywords.resource` no tiene importado `../variables/config_vars.robot` en su sección `*** Settings ***`, o la ruta relativa es incorrecta. Dado que `${TEMP_DIR}` se define en `config_vars.robot`, si ese archivo no se carga en el contexto de la keyword, la variable no está disponible.

**Solución:**
1. Abre `resources/telecom_keywords.resource` y verifica que la sección `*** Settings ***` contiene exactamente:
   ```robot
   *** Settings ***
   Library     OperatingSystem
   Library     Collections
   Resource    ../variables/config_vars.robot
   ```
2. Confirma que la ruta `../variables/config_vars.robot` es correcta relativa a la ubicación de `telecom_keywords.resource`. Si el archivo resource está en `resources/`, entonces `../variables/` apunta correctamente a `variables/`.
3. Ejecuta el dry-run para verificar:
   ```bash
   python -m robot --dryrun --nostatusrc resources/telecom_keywords.resource
   ```

---

### Problema 2: Los tests de la sub-suite `clientes/` no aparecen al ejecutar `robot tests/`

**Síntoma:**
Solo se ejecutan los 7 tests de `facturacion_suite.robot` y los 4 tests de `clientes_suite.robot` no aparecen. El total es 7 en lugar de 11.

**Causa:**
El subdirectorio `tests/clientes/` no fue creado correctamente, o el archivo `clientes_suite.robot` tiene una extensión incorrecta (por ejemplo, `.txt` o `.Robot` con mayúscula en Windows). Robot Framework solo reconoce archivos con extensión `.robot` o `.resource` (en minúsculas en sistemas case-sensitive como Linux).

**Solución:**
1. Verifica que el directorio y el archivo existen con el nombre correcto:
   ```bash
   # ── Windows ──────────────────────────────────────────────────────────────
   dir tests\clientes
   
   # ── macOS / Linux ────────────────────────────────────────────────────────
   ls -la tests/clientes/
   ```
2. Confirma que el archivo se llama exactamente `clientes_suite.robot` (sin mayúsculas, sin espacios, con extensión `.robot`).
3. En Linux/macOS, si el archivo tiene extensión `.Robot`, renómbralo:
   ```bash
   mv tests/clientes/clientes_suite.Robot tests/clientes/clientes_suite.robot
   ```
4. Ejecuta el dry-run apuntando directamente al subdirectorio para confirmar que Robot Framework lo detecta:
   ```bash
   python -m robot --dryrun --nostatusrc tests/clientes/
   ```
   Debe mostrar `4 tests, 4 passed`.

---

## 9. Limpieza

Al finalizar el laboratorio, ejecuta los siguientes pasos para dejar el entorno en estado limpio:

```bash
# ── 1. Verificar que temp_data está limpio (el Teardown debería haberlo hecho)
# ── Windows ────────────────────────────────────────────────────────────────
if exist temp_data\*.log del /Q temp_data\*.log

# ── macOS / Linux ──────────────────────────────────────────────────────────
rm -f temp_data/*.log

# ── 2. Consolidar todos los resultados en una carpeta final ────────────────
robot --outputdir results/lab-02-00-02-final tests/

# ── 3. Opcional: eliminar carpetas de resultados intermedios ───────────────
# ── Windows ────────────────────────────────────────────────────────────────
for /D %i in (results\smoke results\billing results\smoke_billing results\smoke_or_customer results\no_low results\critical results\template results\template_fail results\template_ok results\setup_teardown_test results\final_validation results\final_smoke_critical results\final_customers results\final_no_low) do if exist %i rmdir /S /Q %i

# ── macOS / Linux ──────────────────────────────────────────────────────────
rm -rf results/smoke results/billing results/smoke_billing results/smoke_or_customer \
       results/no_low results/critical results/template results/template_fail \
       results/template_ok results/setup_teardown_test results/final_validation \
       results/final_smoke_critical results/final_customers results/final_no_low

# ── 4. Desactivar el entorno virtual ──────────────────────────────────────
deactivate
```

> 💡 **Recomendación:** Guarda una copia de respaldo del proyecto completo antes de comenzar el siguiente módulo:
> ```bash
> # ── Windows ──────────────────────────────────────────────────────────────
> xcopy /E /I proyecto-telecom proyecto-telecom-backup-lab02-00-02
> 
> # ── macOS / Linux ────────────────────────────────────────────────────────
> cp -r proyecto-telecom proyecto-telecom-backup-lab02-00-02
> ```

---

## 10. Resumen

En este laboratorio implementaste los mecanismos de control de ejecución más importantes de Robot Framework 7.x:

| Concepto | Lo que aprendiste |
|----------|-------------------|
| **Suite Setup / Teardown** | Inicialización y limpieza a nivel de suite usando `OperatingSystem` para gestionar archivos temporales |
| **Test Setup / Teardown** | Precondiciones y postcondiciones aplicadas automáticamente a todos los tests de una suite |
| **Tags y filtrado** | Asignación de tags por categoría y criticidad; uso de `--include`, `--exclude` y operadores `AND` / `OR` |
| **Jerarquía de suites** | Cómo Robot Framework trata subdirectorios como sub-suites anidadas sin configuración adicional |
| **Test Template** | Parametrización compacta de un test case con múltiples filas de datos en formato tabla |
| **Run Keywords / No Operation** | Uso de keywords BuiltIn para control de flujo condicional en setup/teardown |

### Patrones clave para recordar

```robot
# ── Patrón 1: Setup/Teardown a nivel de suite ─────────────────────────────
Suite Setup      Mi Keyword De Inicialización
Suite Teardown   Mi Keyword De Limpieza

# ── Patrón 2: Setup/Teardown a nivel de test ──────────────────────────────
Test Setup       Preparar Contexto De Test
Test Teardown    Limpiar Contexto De Test

# ── Patrón 3: Tags y filtrado ─────────────────────────────────────────────
[Tags]    smoke    billing    critical
# Ejecución: robot --include smokeANDbilling --exclude low tests/

# ── Patrón 4: Test Template con tabla ─────────────────────────────────────
Mi Test Con Template
    [Template]    Mi Keyword
    argumento1    argumento2    argumento3
    valor1a       valor2a       valor3a
    valor1b       valor2b       valor3b
```

### Recursos adicionales

- [Robot Framework User Guide — Suite Setup and Teardown](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#suite-setup-and-teardown)
- [Robot Framework User Guide — Tagging test cases](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#tagging-test-cases)
- [Robot Framework User Guide — Test templates](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#test-templates)
- [OperatingSystem Library — Referencia completa](https://robotframework.org/robotframework/latest/libraries/OperatingSystem.html)
- [Robot Framework User Guide — Organizing test suites](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#organizing-test-suites)

---
