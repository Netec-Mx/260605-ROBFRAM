# Tests con lógica condicional y bucles de datos

## Metadatos

| Campo            | Detalle                                      |
|------------------|----------------------------------------------|
| **Duración**     | 72 minutos                                   |
| **Complejidad**  | Media                                        |
| **Nivel Bloom**  | Aplicar (Apply)                              |
| **Módulo**       | 3 — Control de flujo avanzado                |
| **Versión RF**   | Robot Framework 7.x                          |

---

## Descripción General

En este laboratorio construirás una suite de pruebas que simula la validación de un catálogo de planes de telecomunicaciones para la empresa ficticia **TelecomRF S.A.** Aplicarás bloques `IF / ELSE IF / ELSE` nativos de Robot Framework para clasificar planes según su precio, bucles `FOR` para iterar sobre una lista de clientes y validar su estado de cuenta, y un bucle `WHILE` para simular reintentos de conexión con límite máximo de intentos. Todas las keywords estarán documentadas con `[Documentation]` y usarán aserciones de la librería `BuiltIn` para garantizar resultados esperados.

---

## Objetivos de Aprendizaje

Al completar este laboratorio, serás capaz de:

- [ ] Implementar bloques `IF / ELSE IF / ELSE` dentro de keywords y test cases para ejecutar lógica condicional basada en valores de variables
- [ ] Utilizar bucles `FOR` sobre listas y rangos, y bucles `WHILE` con condición de parada para iterar sobre conjuntos de datos de prueba
- [ ] Aplicar keywords de aserción de la librería `BuiltIn` (`Should Be Equal As Numbers`, `Should Match Regexp`, `Should Contain`) para validar resultados esperados
- [ ] Combinar estructuras de control anidadas para simular flujos de validación de datos representativos de procesos de telecomunicaciones
- [ ] Documentar keywords con `[Documentation]` y `[Arguments]` siguiendo buenas prácticas de mantenibilidad

---

## Prerrequisitos

### Conocimiento previo

- Haber completado los laboratorios **02-00-01** y **02-00-02** del Módulo 2
- Comprensión de variables escalares `${var}`, listas `@{lista}` y diccionarios `&{dicc}` en Robot Framework
- Conocimiento básico de estructuras de control (if/for/while) en cualquier lenguaje de programación
- Familiaridad con la librería `BuiltIn` y keywords de aserción básicas

### Acceso y herramientas

- Entorno virtual Python activo con Robot Framework 7.x instalado
- Visual Studio Code con la extensión **Robot Framework Language Server** instalada
- Terminal (cmd/PowerShell en Windows; bash/zsh en macOS/Linux)
- Proyecto base del Módulo 2 disponible como referencia (no se modifica)

---

## Entorno de Laboratorio

### Tabla de software requerido

| Componente                    | Versión mínima | Propósito                              |
|-------------------------------|----------------|----------------------------------------|
| Python                        | 3.10           | Intérprete base                        |
| Robot Framework               | 7.x            | Framework de automatización            |
| robotframework-collections    | incluida en RF  | Librería Collections para listas/dicts |
| VS Code                       | 1.85           | Editor principal                       |
| RF Language Server (extensión)| 1.12           | Soporte de sintaxis y autocompletado   |

### Configuración del entorno

Sigue los pasos a continuación **antes** de comenzar el laboratorio. Los comandos se presentan con variantes para Windows y Unix.

**Paso A — Activar el entorno virtual existente:**

```bash
# Windows (cmd)
cd C:\proyectos\telecomrf
venv\Scripts\activate

# Windows (PowerShell)
cd C:\proyectos\telecomrf
.\venv\Scripts\Activate.ps1

# macOS / Linux
cd ~/proyectos/telecomrf
source venv/bin/activate
```

**Paso B — Verificar la versión de Robot Framework:**

```bash
robot --version
# Salida esperada: Robot Framework 7.x.x (Python 3.x.x ...)
```

**Paso C — Verificar que la librería Collections está disponible:**

```bash
python -c "import robot.libraries.Collections; print('Collections OK')"
# Salida esperada: Collections OK
```

**Paso D — Crear la estructura de directorios para este laboratorio:**

```bash
# Windows (cmd / PowerShell)
mkdir labs\lab_03_00_01
mkdir labs\lab_03_00_01\resources
mkdir labs\lab_03_00_01\results

# macOS / Linux
mkdir -p labs/lab_03_00_01/resources
mkdir -p labs/lab_03_00_01/results
```

Al finalizar, la estructura del proyecto debe verse así:

```
telecomrf/
├── venv/
├── labs/
│   ├── lab_02_00_01/          ← laboratorio anterior (no modificar)
│   └── lab_03_00_01/
│       ├── resources/
│       │   └── telecom_data.resource   ← crearemos este archivo
│       ├── results/                    ← carpeta para reportes
│       └── suite_control_flujo.robot   ← suite principal
```

---

## Instrucciones Paso a Paso

---

### Paso 1 — Crear el archivo de datos y recursos compartidos

**Objetivo:** Definir las variables de datos (planes, clientes) que serán consumidas por los bucles y condiciones de la suite, y centralizar keywords de utilidad en un archivo Resource.

#### Instrucciones

1. Abre VS Code y navega a la carpeta `labs/lab_03_00_01/resources/`.

2. Crea el archivo `telecom_data.resource` con el siguiente contenido:

```robotframework
*** Settings ***
Library    Collections

*** Variables ***
# ---------------------------------------------------------------------------
# Catálogo de planes de telecomunicaciones (precio en USD/mes)
# Clasificación esperada:
#   básico    → precio < 20
#   estándar  → precio >= 20 y < 50
#   premium   → precio >= 50
# ---------------------------------------------------------------------------
@{PLANES}
...    Plan Básico 10|10.00|básico
...    Plan Básico 15|15.99|básico
...    Plan Estándar 30|30.00|estándar
...    Plan Estándar 45|45.50|estándar
...    Plan Premium 60|60.00|premium
...    Plan Premium 99|99.99|premium

# ---------------------------------------------------------------------------
# Lista de clientes con formato: nombre|telefono|estado_cuenta|saldo_usd
# Estados válidos: activo, suspendido, moroso
# ---------------------------------------------------------------------------
@{CLIENTES}
...    Ana Torres|+52-555-123-4567|activo|0.00
...    Carlos Ruiz|+52-555-987-6543|moroso|150.75
...    María López|+52-555-456-7890|activo|0.00
...    Jorge Pérez|+52-555-321-0987|suspendido|0.00
...    Laura Gómez|+52-555-654-3210|moroso|89.50

# ---------------------------------------------------------------------------
# Parámetros de simulación de reintentos de conexión
# ---------------------------------------------------------------------------
${MAX_REINTENTOS}    5
${INTERVALO_REINTENTO_SEG}    0
${PATRON_TELEFONO}    ^\\+\\d{2}-\\d{3}-\\d{3}-\\d{4}$

*** Keywords ***
Separar Campos De Cadena
    [Documentation]    Divide una cadena con separador "|" y devuelve una lista de campos.
    ...                Argumento: cadena_datos — string con campos separados por "|"
    ...                Retorna: lista de strings con cada campo
    [Arguments]    ${cadena_datos}
    ${campos}=    Split String    ${cadena_datos}    |
    RETURN    ${campos}

Obtener Campo
    [Documentation]    Extrae un campo específico de una lista por índice.
    ...                Argumentos: lista_campos — lista de strings; indice — entero 0-based
    [Arguments]    ${lista_campos}    ${indice}
    ${valor}=    Get From List    ${lista_campos}    ${indice}
    RETURN    ${valor}
```

3. Guarda el archivo (`Ctrl+S` / `Cmd+S`).

#### Salida esperada

VS Code no debe mostrar errores de sintaxis en el archivo. El Language Server subrayará en verde las keywords definidas en `*** Keywords ***`.

#### Verificación

```bash
# Verifica que el archivo existe y tiene contenido
# Windows
type labs\lab_03_00_01\resources\telecom_data.resource

# macOS / Linux
cat labs/lab_03_00_01/resources/telecom_data.resource
```

La salida debe mostrar el contenido del archivo sin errores.

---

### Paso 2 — Crear la suite principal e importar recursos

**Objetivo:** Establecer la estructura base de la suite con la sección `*** Settings ***` correctamente configurada y crear los primeros test cases vacíos como andamiaje.

#### Instrucciones

1. En la carpeta `labs/lab_03_00_01/`, crea el archivo `suite_control_flujo.robot`.

2. Escribe la sección de configuración inicial:

```robotframework
*** Settings ***
Documentation    Suite de pruebas: Validación de catálogo de planes y clientes
...              Empresa ficticia: TelecomRF S.A.
...              Módulo 3 — Laboratorio 03-00-01
...              Cubre: IF/ELSE IF/ELSE, FOR, WHILE, aserciones BuiltIn
Resource         resources/telecom_data.resource
Library          Collections
Library          String

*** Variables ***
# Variable de control para el bucle WHILE de reintentos
${CONEXION_EXITOSA}    ${FALSE}

*** Test Cases ***
TC-01 Clasificar Planes Por Precio
    [Documentation]    Itera sobre el catálogo de planes e invoca la keyword
    ...                de clasificación para cada uno. Verifica que la categoría
    ...                calculada coincide con la categoría esperada en los datos.
    [Tags]    clasificacion    condicional
    Clasificar Todos Los Planes

TC-02 Validar Estado De Cuenta De Clientes
    [Documentation]    Itera sobre la lista de clientes y valida su estado de cuenta.
    ...                Clientes morosos deben tener saldo > 0; activos y suspendidos
    ...                deben tener saldo = 0.
    [Tags]    clientes    bucle-for
    Validar Todos Los Clientes

TC-03 Simular Reintentos De Conexion
    [Documentation]    Ejecuta un bucle WHILE que simula intentos de conexión.
    ...                El bucle se detiene cuando la conexión es exitosa o se
    ...                alcanza el límite máximo de reintentos.
    [Tags]    conexion    bucle-while
    ${intentos_realizados}=    Simular Conexion Con Reintentos    ${MAX_REINTENTOS}
    Should Be True    ${intentos_realizados} <= ${MAX_REINTENTOS}
    ...    El número de intentos no debe superar el máximo configurado

TC-04 Validar Formato De Telefonos
    [Documentation]    Itera sobre los clientes y valida que el número de teléfono
    ...                cumple el patrón internacional: +XX-XXX-XXX-XXXX
    [Tags]    telefonos    regexp
    Validar Formato Telefonos De Clientes

TC-05 Flujo Integrado Clasificacion Y Validacion
    [Documentation]    Test integrador que combina clasificación de planes y
    ...                validación de clientes en un único flujo con lógica
    ...                condicional anidada.
    [Tags]    integrado    avanzado
    Ejecutar Validacion Integrada

*** Keywords ***
```

3. Guarda el archivo.

#### Salida esperada

El Language Server de VS Code debe reconocer los test cases y no mostrar errores de importación (el archivo resource debe resolverse correctamente).

#### Verificación

En la terminal, ejecuta un dry-run para verificar que la suite se parsea sin errores:

```bash
# Desde la raíz del proyecto (con venv activo)
robot --dryrun --outputdir labs/lab_03_00_01/results labs/lab_03_00_01/suite_control_flujo.robot
```

> **Nota:** El dry-run fallará porque las keywords referenciadas aún no están implementadas. Lo importante es que el error sea `No keyword with name '...' found` y **no** un error de sintaxis o importación.

---

### Paso 3 — Implementar la keyword de clasificación de planes (IF / ELSE IF / ELSE)

**Objetivo:** Crear la keyword `Clasificar Plan Por Precio` que usa un bloque `IF / ELSE IF / ELSE` para asignar una categoría según el precio, y la keyword `Clasificar Todos Los Planes` que la invoca desde un bucle `FOR`.

#### Instrucciones

1. En la sección `*** Keywords ***` del archivo `suite_control_flujo.robot`, agrega las siguientes keywords:

```robotframework
Clasificar Plan Por Precio
    [Documentation]    Clasifica un plan de telecomunicaciones según su precio mensual.
    ...                Reglas de clasificación:
    ...                  - básico    → precio < 20.00 USD
    ...                  - estándar  → precio >= 20.00 y < 50.00 USD
    ...                  - premium   → precio >= 50.00 USD
    ...                Argumentos:
    ...                  nombre_plan     — nombre descriptivo del plan
    ...                  precio          — precio mensual como número (float o int)
    ...                  categoria_esp   — categoría esperada para validación
    [Arguments]    ${nombre_plan}    ${precio}    ${categoria_esp}
    # Convertir precio a número flotante para comparaciones numéricas
    ${precio_num}=    Convert To Number    ${precio}
    # Determinar categoría usando bloque IF nativo (RF 4.0+)
    IF    ${precio_num} < 20.0
        ${categoria_calculada}=    Set Variable    básico
        Log    [BÁSICO] ${nombre_plan} → $${precio_num}/mes    console=True
    ELSE IF    ${precio_num} < 50.0
        ${categoria_calculada}=    Set Variable    estándar
        Log    [ESTÁNDAR] ${nombre_plan} → $${precio_num}/mes    console=True
    ELSE
        ${categoria_calculada}=    Set Variable    premium
        Log    [PREMIUM] ${nombre_plan} → $${precio_num}/mes    console=True
    END
    # Validar que la categoría calculada coincide con la esperada
    Should Be Equal    ${categoria_calculada}    ${categoria_esp}
    ...    Plan "${nombre_plan}": categoría calculada "${categoria_calculada}" ≠ esperada "${categoria_esp}"
    RETURN    ${categoria_calculada}

Clasificar Todos Los Planes
    [Documentation]    Itera sobre la lista global PLANES y clasifica cada uno.
    ...                Usa un bucle FOR con separación de campos por "|".
    ...                Llama a "Clasificar Plan Por Precio" para cada elemento.
    ${planes_procesados}=    Set Variable    ${0}
    FOR    ${plan_raw}    IN    @{PLANES}
        # Separar los campos del string: nombre|precio|categoria_esperada
        ${campos}=    Separar Campos De Cadena    ${plan_raw}
        ${nombre}=      Obtener Campo    ${campos}    ${0}
        ${precio}=      Obtener Campo    ${campos}    ${1}
        ${categoria}=   Obtener Campo    ${campos}    ${2}
        # Invocar clasificación con validación incorporada
        Clasificar Plan Por Precio    ${nombre}    ${precio}    ${categoria}
        ${planes_procesados}=    Evaluate    ${planes_procesados} + 1
    END
    Log    Total de planes clasificados correctamente: ${planes_procesados}    console=True
    Should Be Equal As Numbers    ${planes_procesados}    6
    ...    Se esperaban 6 planes en el catálogo, se procesaron ${planes_procesados}
```

2. Guarda el archivo.

#### Salida esperada al ejecutar solo TC-01

```bash
robot --test "TC-01 Clasificar Planes Por Precio" \
      --outputdir labs/lab_03_00_01/results \
      labs/lab_03_00_01/suite_control_flujo.robot
```

La consola debe mostrar:

```
TC-01 Clasificar Planes Por Precio
[BÁSICO] Plan Básico 10 → $10.0/mes
[BÁSICO] Plan Básico 15 → $15.99/mes
[ESTÁNDAR] Plan Estándar 30 → $30.0/mes
[ESTÁNDAR] Plan Estándar 45 → $45.5/mes
[PREMIUM] Plan Premium 60 → $60.0/mes
[PREMIUM] Plan Premium 99 → $99.99/mes
Total de planes clasificados correctamente: 6
TC-01 Clasificar Planes Por Precio              | PASS |
```

#### Verificación

Confirma que el test case **TC-01** aparece en verde en el reporte HTML:

```bash
# Abrir el reporte (Windows)
start labs\lab_03_00_01\results\report.html

# macOS
open labs/lab_03_00_01/results/report.html

# Linux
xdg-open labs/lab_03_00_01/results/report.html
```

---

### Paso 4 — Implementar validación de clientes con bucle FOR y condicional anidado

**Objetivo:** Crear la keyword `Validar Cliente` que usa `IF / ELSE IF / ELSE` para verificar el estado de cuenta según el estado del cliente, y `Validar Todos Los Clientes` que la invoca en un bucle `FOR`.

#### Instrucciones

1. Agrega las siguientes keywords a la sección `*** Keywords ***`:

```robotframework
Validar Cliente
    [Documentation]    Valida el estado de cuenta de un cliente según su estado.
    ...                Reglas de negocio TelecomRF:
    ...                  - activo     → saldo debe ser 0.00 (cuenta al día)
    ...                  - suspendido → saldo debe ser 0.00 (suspendido por política)
    ...                  - moroso     → saldo debe ser > 0 (tiene deuda pendiente)
    ...                Argumentos:
    ...                  nombre   — nombre completo del cliente
    ...                  telefono — número de teléfono (solo para logging)
    ...                  estado   — estado de la cuenta: activo|suspendido|moroso
    ...                  saldo    — saldo adeudado en USD como string numérico
    [Arguments]    ${nombre}    ${telefono}    ${estado}    ${saldo}
    ${saldo_num}=    Convert To Number    ${saldo}
    Log    Validando cliente: ${nombre} | Estado: ${estado} | Saldo: $${saldo_num}    console=True
    IF    '${estado}' == 'activo'
        Should Be Equal As Numbers    ${saldo_num}    0
        ...    Cliente activo "${nombre}" no debería tener saldo pendiente (saldo: $${saldo_num})
        Log    ✓ Cliente activo sin deuda — OK    console=True
    ELSE IF    '${estado}' == 'suspendido'
        Should Be Equal As Numbers    ${saldo_num}    0
        ...    Cliente suspendido "${nombre}" no debería tener saldo pendiente (saldo: $${saldo_num})
        Log    ✓ Cliente suspendido sin deuda — OK    console=True
    ELSE IF    '${estado}' == 'moroso'
        Should Be True    ${saldo_num} > 0
        ...    Cliente moroso "${nombre}" debe tener saldo > 0 (saldo registrado: $${saldo_num})
        Log    ✓ Cliente moroso con deuda $${saldo_num} — registrado correctamente    console=True
    ELSE
        Fail    Estado de cuenta desconocido "${estado}" para el cliente "${nombre}"
    END

Validar Todos Los Clientes
    [Documentation]    Itera sobre la lista global CLIENTES y valida cada registro.
    ...                Usa bucle FOR con separación de campos por "|".
    ...                Acumula contadores por tipo de estado para reporte final.
    ${total_activos}=      Set Variable    ${0}
    ${total_suspendidos}=  Set Variable    ${0}
    ${total_morosos}=      Set Variable    ${0}
    FOR    ${cliente_raw}    IN    @{CLIENTES}
        ${campos}=    Separar Campos De Cadena    ${cliente_raw}
        ${nombre}=    Obtener Campo    ${campos}    ${0}
        ${tel}=       Obtener Campo    ${campos}    ${1}
        ${estado}=    Obtener Campo    ${campos}    ${2}
        ${saldo}=     Obtener Campo    ${campos}    ${3}
        Validar Cliente    ${nombre}    ${tel}    ${estado}    ${saldo}
        # Contabilizar por estado usando IF anidado
        IF    '${estado}' == 'activo'
            ${total_activos}=    Evaluate    ${total_activos} + 1
        ELSE IF    '${estado}' == 'suspendido'
            ${total_suspendidos}=    Evaluate    ${total_suspendidos} + 1
        ELSE IF    '${estado}' == 'moroso'
            ${total_morosos}=    Evaluate    ${total_morosos} + 1
        END
    END
    # Resumen de validación
    Log    ═══ RESUMEN DE CLIENTES ═══    console=True
    Log    Activos: ${total_activos} | Suspendidos: ${total_suspendidos} | Morosos: ${total_morosos}    console=True
    # Verificar que se procesaron exactamente 5 clientes
    ${total}=    Evaluate    ${total_activos} + ${total_suspendidos} + ${total_morosos}
    Should Be Equal As Numbers    ${total}    5
    ...    Se esperaban 5 clientes en total, se procesaron ${total}
    # Verificar distribución esperada según datos de prueba
    Should Be Equal As Numbers    ${total_morosos}    2
    ...    Se esperaban 2 clientes morosos en los datos de prueba
```

2. Guarda el archivo.

#### Salida esperada al ejecutar TC-02

```bash
robot --test "TC-02 Validar Estado De Cuenta De Clientes" \
      --outputdir labs/lab_03_00_01/results \
      labs/lab_03_00_01/suite_control_flujo.robot
```

```
TC-02 Validar Estado De Cuenta De Clientes
Validando cliente: Ana Torres | Estado: activo | Saldo: $0.0
✓ Cliente activo sin deuda — OK
Validando cliente: Carlos Ruiz | Estado: moroso | Saldo: $150.75
✓ Cliente moroso con deuda $150.75 — registrado correctamente
Validando cliente: María López | Estado: activo | Saldo: $0.0
✓ Cliente activo sin deuda — OK
Validando cliente: Jorge Pérez | Estado: suspendido | Saldo: $0.0
✓ Cliente suspendido sin deuda — OK
Validando cliente: Laura Gómez | Estado: moroso | Saldo: $89.5
✓ Cliente moroso con deuda $89.5 — registrado correctamente
═══ RESUMEN DE CLIENTES ═══
Activos: 2 | Suspendidos: 1 | Morosos: 2
TC-02 Validar Estado De Cuenta De Clientes      | PASS |
```

#### Verificación

Confirma que el log HTML muestra el árbol de keywords con todas las ramas `IF` evaluadas correctamente. En el reporte, cada cliente debe aparecer como una llamada exitosa a `Validar Cliente`.

---

### Paso 5 — Implementar el bucle WHILE con simulación de reintentos

**Objetivo:** Crear la keyword `Simular Conexion Con Reintentos` que usa un bucle `WHILE` con límite máximo de iteraciones y `BREAK` para salir cuando la conexión es exitosa.

#### Instrucciones

1. Agrega las siguientes keywords:

```robotframework
Simular Intento De Conexion
    [Documentation]    Simula un intento de conexión a un servidor de red.
    ...                En este ejercicio, la conexión tiene éxito en el intento
    ...                número 3 (simulado con una comparación de contador).
    ...                Argumento: numero_intento — entero con el número de intento actual
    ...                Retorna: ${TRUE} si la conexión fue exitosa, ${FALSE} si falló
    [Arguments]    ${numero_intento}
    Log    Intento de conexión #${numero_intento}...    console=True
    # Simulación: la conexión tiene éxito en el intento 3
    IF    ${numero_intento} == 3
        Log    ✓ Conexión establecida exitosamente en el intento ${numero_intento}    console=True
        RETURN    ${TRUE}
    ELSE
        Log    ✗ Conexión fallida — reintentando...    console=True
        RETURN    ${FALSE}
    END

Simular Conexion Con Reintentos
    [Documentation]    Ejecuta reintentos de conexión usando un bucle WHILE.
    ...                El bucle se detiene cuando:
    ...                  a) La conexión es exitosa (BREAK), o
    ...                  b) Se alcanza el límite máximo de reintentos
    ...                Argumento: max_intentos — número máximo de intentos permitidos
    ...                Retorna: número total de intentos realizados
    [Arguments]    ${max_intentos}
    ${intento_actual}=    Set Variable    ${1}
    ${conexion_ok}=       Set Variable    ${FALSE}
    WHILE    not ${conexion_ok}    limit=${max_intentos}
        ${conexion_ok}=    Simular Intento De Conexion    ${intento_actual}
        IF    ${conexion_ok}
            Log    Conexión lograda en ${intento_actual} intento(s)    console=True
            BREAK
        END
        ${intento_actual}=    Evaluate    ${intento_actual} + 1
    END
    # Verificar que la conexión fue eventualmente exitosa
    Should Be True    ${conexion_ok}
    ...    La conexión no pudo establecerse en ${max_intentos} intentos
    # Verificar que el número de intentos está dentro del rango esperado
    Should Be True    ${intento_actual} >= 1
    ...    El contador de intentos debe ser al menos 1
    Should Be True    ${intento_actual} <= ${max_intentos}
    ...    El contador de intentos (${intento_actual}) superó el máximo (${max_intentos})
    Log    ═══ RESULTADO CONEXIÓN ═══    console=True
    Log    Intentos realizados: ${intento_actual} / ${max_intentos}    console=True
    RETURN    ${intento_actual}
```

2. Guarda el archivo.

#### Salida esperada al ejecutar TC-03

```bash
robot --test "TC-03 Simular Reintentos De Conexion" \
      --outputdir labs/lab_03_00_01/results \
      labs/lab_03_00_01/suite_control_flujo.robot
```

```
TC-03 Simular Reintentos De Conexion
Intento de conexión #1...
✗ Conexión fallida — reintentando...
Intento de conexión #2...
✗ Conexión fallida — reintentando...
Intento de conexión #3...
✓ Conexión establecida exitosamente en el intento 3
Conexión lograda en 3 intento(s)
═══ RESULTADO CONEXIÓN ═══
Intentos realizados: 3 / 5
TC-03 Simular Reintentos De Conexion            | PASS |
```

#### Verificación

Modifica temporalmente el valor de `Simular Intento De Conexion` para que nunca retorne `${TRUE}` (cambia `== 3` por `== 99`) y ejecuta de nuevo TC-03. Debes observar que el bucle WHILE se detiene al alcanzar el `limit=${max_intentos}` y el test **falla** con el mensaje de aserción. Restaura el valor original después de verificar.

---

### Paso 6 — Implementar validación de formato telefónico con Should Match Regexp

**Objetivo:** Crear la keyword `Validar Formato Telefonos De Clientes` que usa `Should Match Regexp` en un bucle `FOR` para verificar que todos los números de teléfono cumplen el patrón internacional definido.

#### Instrucciones

1. Agrega la siguiente keyword:

```robotframework
Validar Formato Telefono Individual
    [Documentation]    Valida que un número de teléfono cumple el patrón internacional.
    ...                Patrón esperado: +XX-XXX-XXX-XXXX (ej: +52-555-123-4567)
    ...                Argumentos:
    ...                  nombre   — nombre del cliente (para mensajes de error)
    ...                  telefono — número de teléfono a validar
    [Arguments]    ${nombre}    ${telefono}
    Should Not Be Empty    ${telefono}
    ...    El número de teléfono del cliente "${nombre}" no puede estar vacío
    Should Match Regexp    ${telefono}    ${PATRON_TELEFONO}
    ...    Teléfono "${telefono}" del cliente "${nombre}" no cumple el formato +XX-XXX-XXX-XXXX
    Log    ✓ Teléfono válido: ${telefono} (${nombre})    console=True

Validar Formato Telefonos De Clientes
    [Documentation]    Itera sobre todos los clientes y valida el formato de teléfono.
    ...                Usa bucle FOR con CONTINUE para omitir entradas con nombre vacío
    ...                (aunque en este dataset no existen, se incluye como buena práctica).
    ${telefonos_validados}=    Set Variable    ${0}
    FOR    ${cliente_raw}    IN    @{CLIENTES}
        ${campos}=    Separar Campos De Cadena    ${cliente_raw}
        ${nombre}=    Obtener Campo    ${campos}    ${0}
        ${tel}=       Obtener Campo    ${campos}    ${1}
        # Usar CONTINUE para omitir registros con nombre vacío (buena práctica defensiva)
        IF    '${nombre}' == ''
            Log    ADVERTENCIA: Registro sin nombre omitido    console=True
            CONTINUE
        END
        Validar Formato Telefono Individual    ${nombre}    ${tel}
        ${telefonos_validados}=    Evaluate    ${telefonos_validados} + 1
    END
    Log    Total de teléfonos validados: ${telefonos_validados}    console=True
    Should Be Equal As Numbers    ${telefonos_validados}    5
    ...    Se esperaban 5 teléfonos validados, se procesaron ${telefonos_validados}
```

2. Guarda el archivo.

#### Salida esperada al ejecutar TC-04

```bash
robot --test "TC-04 Validar Formato De Telefonos" \
      --outputdir labs/lab_03_00_01/results \
      labs/lab_03_00_01/suite_control_flujo.robot
```

```
TC-04 Validar Formato De Telefonos
✓ Teléfono válido: +52-555-123-4567 (Ana Torres)
✓ Teléfono válido: +52-555-987-6543 (Carlos Ruiz)
✓ Teléfono válido: +52-555-456-7890 (María López)
✓ Teléfono válido: +52-555-321-0987 (Jorge Pérez)
✓ Teléfono válido: +52-555-654-3210 (Laura Gómez)
Total de teléfonos validados: 5
TC-04 Validar Formato De Telefonos              | PASS |
```

#### Verificación

Introduce intencionalmente un número con formato incorrecto en `@{CLIENTES}` del archivo resource (por ejemplo, cambia `+52-555-123-4567` por `555-123-4567`) y ejecuta TC-04. El test debe **fallar** con un mensaje claro indicando qué número no cumple el patrón. Restaura el valor original.

---

### Paso 7 — Implementar el test integrador con lógica anidada

**Objetivo:** Crear la keyword `Ejecutar Validacion Integrada` que combina clasificación de planes y validación de clientes en un único flujo, usando condicionales anidados para generar un reporte de estado consolidado.

#### Instrucciones

1. Agrega la keyword integradora:

```robotframework
Evaluar Estado General Del Plan
    [Documentation]    Keyword auxiliar que determina si un plan es recomendable
    ...                para un cliente según su estado de cuenta.
    ...                Lógica:
    ...                  - Cliente moroso + plan premium → NO recomendable
    ...                  - Cliente activo + cualquier plan → recomendable
    ...                  - Otros casos → evaluación pendiente
    ...                Argumentos:
    ...                  estado_cliente — estado de cuenta del cliente
    ...                  categoria_plan — categoría del plan: básico|estándar|premium
    ...                Retorna: string con la recomendación
    [Arguments]    ${estado_cliente}    ${categoria_plan}
    IF    '${estado_cliente}' == 'activo'
        ${recomendacion}=    Set Variable    RECOMENDABLE
    ELSE IF    '${estado_cliente}' == 'moroso'
        IF    '${categoria_plan}' == 'premium'
            ${recomendacion}=    Set Variable    NO RECOMENDABLE
        ELSE
            ${recomendacion}=    Set Variable    CONDICIONALMENTE RECOMENDABLE
        END
    ELSE
        ${recomendacion}=    Set Variable    EVALUACIÓN PENDIENTE
    END
    RETURN    ${recomendacion}

Ejecutar Validacion Integrada
    [Documentation]    Flujo integrador que combina clasificación de planes y
    ...                validación de clientes. Para cada cliente moroso, evalúa
    ...                si el plan premium sería recomendable.
    ...                Genera un reporte consolidado al finalizar.
    Log    ═══ INICIO VALIDACIÓN INTEGRADA ═══    console=True
    # Paso 1: Clasificar todos los planes y recolectar categorías
    ${categorias}=    Create List
    FOR    ${plan_raw}    IN    @{PLANES}
        ${campos}=       Separar Campos De Cadena    ${plan_raw}
        ${nombre}=       Obtener Campo    ${campos}    ${0}
        ${precio}=       Obtener Campo    ${campos}    ${1}
        ${cat_esp}=      Obtener Campo    ${campos}    ${2}
        ${cat_calc}=     Clasificar Plan Por Precio    ${nombre}    ${precio}    ${cat_esp}
        Append To List    ${categorias}    ${cat_calc}
    END
    Should Contain    ${categorias}    premium
    ...    El catálogo debe contener al menos un plan premium
    Should Contain    ${categorias}    básico
    ...    El catálogo debe contener al menos un plan básico
    # Paso 2: Para cada cliente moroso, evaluar recomendación con plan premium
    Log    ═══ EVALUACIÓN DE CLIENTES MOROSOS ═══    console=True
    ${evaluaciones}=    Create List
    FOR    ${cliente_raw}    IN    @{CLIENTES}
        ${campos}=   Separar Campos De Cadena    ${cliente_raw}
        ${nombre}=   Obtener Campo    ${campos}    ${0}
        ${estado}=   Obtener Campo    ${campos}    ${2}
        IF    '${estado}' == 'moroso'
            ${rec}=    Evaluar Estado General Del Plan    ${estado}    premium
            Log    ${nombre} (moroso) + Plan Premium → ${rec}    console=True
            Append To List    ${evaluaciones}    ${rec}
        END
    END
    # Verificar que todos los clientes morosos recibieron evaluación NO RECOMENDABLE
    FOR    ${evaluacion}    IN    @{evaluaciones}
        Should Be Equal    ${evaluacion}    NO RECOMENDABLE
        ...    Un cliente moroso no debería recibir recomendación positiva para plan premium
    END
    Log    ═══ FIN VALIDACIÓN INTEGRADA ═══    console=True
    Log    Planes categorizados: ${categorias}    console=True
    Log    Evaluaciones morosos: ${evaluaciones}    console=True
```

2. Guarda el archivo.

#### Salida esperada al ejecutar TC-05

```bash
robot --test "TC-05 Flujo Integrado Clasificacion Y Validacion" \
      --outputdir labs/lab_03_00_01/results \
      labs/lab_03_00_01/suite_control_flujo.robot
```

```
TC-05 Flujo Integrado Clasificacion Y Validacion
═══ INICIO VALIDACIÓN INTEGRADA ═══
[BÁSICO] Plan Básico 10 → $10.0/mes
... (clasificación de todos los planes)
═══ EVALUACIÓN DE CLIENTES MOROSOS ═══
Carlos Ruiz (moroso) + Plan Premium → NO RECOMENDABLE
Laura Gómez (moroso) + Plan Premium → NO RECOMENDABLE
═══ FIN VALIDACIÓN INTEGRADA ═══
TC-05 Flujo Integrado Clasificacion Y Validacion | PASS |
```

#### Verificación

Revisa el log HTML y confirma que el árbol de keywords muestra correctamente la anidación: `Ejecutar Validacion Integrada` → `Clasificar Plan Por Precio` (×6) → `Evaluar Estado General Del Plan` (×2).

---

## Validación y Pruebas

### Ejecución completa de la suite

Ejecuta todos los test cases de una vez y genera el reporte completo:

```bash
robot \
    --outputdir labs/lab_03_00_01/results \
    --log log_lab03.html \
    --report report_lab03.html \
    --output output_lab03.xml \
    labs/lab_03_00_01/suite_control_flujo.robot
```

**Variante Windows (una sola línea):**

```cmd
robot --outputdir labs\lab_03_00_01\results --log log_lab03.html --report report_lab03.html --output output_lab03.xml labs\lab_03_00_01\suite_control_flujo.robot
```

### Resultado esperado de la suite completa

```
==============================================================================
Suite Control Flujo :: Suite de pruebas: Validación de catálogo de planes ...
==============================================================================
TC-01 Clasificar Planes Por Precio                              | PASS |
TC-02 Validar Estado De Cuenta De Clientes                      | PASS |
TC-03 Simular Reintentos De Conexion                            | PASS |
TC-04 Validar Formato De Telefonos                              | PASS |
TC-05 Flujo Integrado Clasificacion Y Validacion                | PASS |
==============================================================================
Suite Control Flujo :: Suite de pruebas: Validación de ...      | PASS |
5 tests, 5 passed, 0 failed
==============================================================================
Output:  .../results/output_lab03.xml
Log:     .../results/log_lab03.html
Report:  .../results/report_lab03.html
```

### Lista de verificación de criterios de éxito

Antes de dar por completado el laboratorio, confirma cada punto:

| # | Criterio | Verificación |
|---|----------|--------------|
| 1 | Los 5 test cases pasan sin errores | `5 tests, 5 passed, 0 failed` en consola |
| 2 | TC-01 clasifica correctamente los 6 planes | Log muestra [BÁSICO]×2, [ESTÁNDAR]×2, [PREMIUM]×2 |
| 3 | TC-02 valida los 5 clientes y detecta 2 morosos | Resumen muestra `Morosos: 2` |
| 4 | TC-03 establece conexión en el intento 3 | Log muestra `Intentos realizados: 3 / 5` |
| 5 | TC-04 valida los 5 teléfonos con regexp | Log muestra `Total de teléfonos validados: 5` |
| 6 | TC-05 evalúa 2 clientes morosos como NO RECOMENDABLE | Log muestra evaluaciones de Carlos Ruiz y Laura Gómez |
| 7 | Todas las keywords tienen `[Documentation]` | Visible en el log HTML de cada keyword |
| 8 | El reporte HTML se genera correctamente | Archivo `report_lab03.html` abre en el navegador |

### Ejecución por etiquetas (opcional)

```bash
# Ejecutar solo tests de lógica condicional
robot --include condicional --outputdir labs/lab_03_00_01/results labs/lab_03_00_01/suite_control_flujo.robot

# Ejecutar solo tests de bucles
robot --include bucle-for --include bucle-while --outputdir labs/lab_03_00_01/results labs/lab_03_00_01/suite_control_flujo.robot

# Ejecutar solo el test avanzado
robot --include avanzado --outputdir labs/lab_03_00_01/results labs/lab_03_00_01/suite_control_flujo.robot
```

---

## Resolución de Problemas

### Problema 1: El bucle WHILE no termina o genera error `WHILE loop was aborted`

**Síntomas:**
- El test TC-03 falla con el mensaje `WHILE loop was aborted because it did not finish within the limit of X iterations`
- O bien, el test corre indefinidamente sin terminar

**Causa:**
El parámetro `limit=` del bucle `WHILE` controla el número máximo de iteraciones permitidas. Si la condición de salida (`${conexion_ok}`) nunca se vuelve `${TRUE}` —por ejemplo, porque la lógica de `Simular Intento De Conexion` nunca retorna `${TRUE}`— el bucle alcanza el límite y Robot Framework lo aborta automáticamente lanzando una excepción. Esto es un mecanismo de seguridad para evitar bucles infinitos.

**Solución:**
1. Verifica que `Simular Intento De Conexion` retorna `${TRUE}` cuando `${numero_intento} == 3`:
   ```robotframework
   IF    ${numero_intento} == 3
       RETURN    ${TRUE}
   ```
2. Confirma que `${MAX_REINTENTOS}` en `telecom_data.resource` tiene el valor `5` (mayor que 3).
3. Si modificaste la keyword durante las pruebas del Paso 5, restaura el valor `== 3`.
4. Asegúrate de que el parámetro `limit` usa la sintaxis correcta: `WHILE    not ${conexion_ok}    limit=${max_intentos}` (con el argumento `limit=` sin espacios alrededor del `=`).

---

### Problema 2: `Should Match Regexp` falla con error de patrón inválido o no coincide

**Síntomas:**
- TC-04 falla con el mensaje `Regular expression pattern '...' is not valid` o `'+52-555-123-4567' does not match '^\\+\\d{2}-\\d{3}-\\d{3}-\\d{4}$'`
- El error aparece incluso con números de teléfono que visualmente parecen correctos

**Causa:**
Existen dos causas frecuentes:
1. **Doble escape en el archivo resource:** En archivos `.resource` y `.robot`, las barras invertidas deben escaparse con doble barra `\\` para que el motor de Robot Framework las pase correctamente al motor de expresiones regulares de Python. Si el patrón tiene `\+` en lugar de `\\+`, el carácter `+` no se escapa correctamente.
2. **Espacios invisibles en los datos:** Al separar campos con `Split String`, si los datos en `@{CLIENTES}` tienen espacios antes o después del número de teléfono, la regexp no coincidirá.

**Solución:**
1. Verifica que el patrón en `telecom_data.resource` usa doble barra invertida:
   ```robotframework
   ${PATRON_TELEFONO}    ^\\+\\d{2}-\\d{3}-\\d{3}-\\d{4}$
   ```
2. Si sospechas de espacios, agrega un `Strip String` antes de la validación:
   ```robotframework
   ${tel_limpio}=    Strip String    ${tel}
   Should Match Regexp    ${tel_limpio}    ${PATRON_TELEFONO}
   ```
3. Para depurar, agrega temporalmente un `Log` que muestre la longitud del string:
   ```robotframework
   ${longitud}=    Get Length    ${tel}
   Log    Teléfono: "${tel}" | Longitud: ${longitud}    console=True
   ```
   Un número `+52-555-123-4567` debe tener exactamente 16 caracteres.

---

## Limpieza

Al finalizar el laboratorio, realiza los siguientes pasos de limpieza:

### 1. Guardar una copia de respaldo del proyecto

```bash
# Windows (PowerShell)
Copy-Item -Recurse labs\lab_03_00_01 labs\lab_03_00_01_backup

# macOS / Linux
cp -r labs/lab_03_00_01 labs/lab_03_00_01_backup
```

### 2. Limpiar archivos temporales de resultados (opcional)

Si deseas liberar espacio, puedes eliminar los archivos XML de salida (conserva los HTML para revisión):

```bash
# Windows
del labs\lab_03_00_01\results\output_lab03.xml

# macOS / Linux
rm labs/lab_03_00_01/results/output_lab03.xml
```

### 3. Desactivar el entorno virtual

```bash
deactivate
```

### 4. Verificar estructura final del proyecto

La estructura final del laboratorio debe verse así:

```
labs/lab_03_00_01/
├── resources/
│   └── telecom_data.resource          ✓ Variables y keywords de datos
├── results/
│   ├── log_lab03.html                 ✓ Log detallado con árbol de keywords
│   └── report_lab03.html              ✓ Reporte de ejecución (5/5 PASS)
└── suite_control_flujo.robot          ✓ Suite principal con 5 test cases
```

---

## Resumen

En este laboratorio implementaste un conjunto completo de técnicas de control de flujo avanzado en Robot Framework 7.x aplicadas a un escenario de telecomunicaciones:

| Técnica | Aplicación en el laboratorio |
|---------|------------------------------|
| `IF / ELSE IF / ELSE` | Clasificación de planes por precio y validación de estado de clientes |
| `FOR` sobre lista `@{}` | Iteración sobre catálogo de planes y lista de clientes |
| `WHILE` con `limit=` | Simulación de reintentos de conexión con límite máximo |
| `BREAK` | Salida anticipada del bucle WHILE al lograr la conexión |
| `CONTINUE` | Omisión defensiva de registros vacíos en el bucle FOR |
| `Should Be Equal As Numbers` | Validación numérica de saldos y contadores |
| `Should Match Regexp` | Validación de formato de números telefónicos |
| `Should Contain` | Verificación de presencia de categorías en listas |
| `Should Not Be Empty` | Validación defensiva de campos obligatorios |
| `[Documentation]` | Documentación de todas las keywords con propósito y argumentos |
| IF anidado | Lógica de recomendación en el test integrador TC-05 |

### Conceptos clave reforzados

- La sintaxis `IF / ELSE IF / ELSE` nativa (RF 4.0+) es más legible y mantenible que `Run Keyword If`; ambas son válidas pero se prefiere la nativa en proyectos nuevos.
- Las condiciones en bloques `IF` se evalúan como expresiones Python: las cadenas deben ir entre comillas simples (`'${var}' == 'valor'`), mientras que los números se comparan directamente (`${num} > 20`).
- El parámetro `limit=` en `WHILE` es obligatorio en Robot Framework 7.x para prevenir bucles infinitos; su ausencia genera un error de sintaxis.
- La combinación de `BREAK` y `CONTINUE` permite un control preciso del flujo dentro de bucles sin necesidad de variables de bandera adicionales.

### Recursos adicionales

- [Documentación oficial — IF/ELSE structure](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#if-else-if-else-structure)
- [Documentación oficial — FOR loops](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#for-loops)
- [Documentación oficial — WHILE loops](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#while-loops)
- [Librería BuiltIn — referencia completa de aserciones](https://robotframework.org/robotframework/latest/libraries/BuiltIn.html)
- [Librería Collections — Get From List, Append To List](https://robotframework.org/robotframework/latest/libraries/Collections.html)

---

---

# Suite robusta con manejo de fallas y recuperación

## Metadatos

| Campo            | Valor                                      |
|------------------|--------------------------------------------|
| **Duración**     | 72 minutos                                 |
| **Complejidad**  | Alta                                       |
| **Nivel Bloom**  | Aplicar (Apply)                            |
| **Módulo**       | 3 — Control de Flujo y Manejo de Errores   |
| **Laboratorio**  | 03-00-02 (Práctica 6)                      |

---

## Descripción General

En este laboratorio construirás una suite de prueba robusta para un flujo ficticio de **activación de servicios de telecomunicaciones**. Aprenderás a provocar errores controlados y capturarlos con `Run Keyword And Expect Error`, a acumular múltiples fallas no críticas con `Run Keyword And Continue On Failure`, y a implementar un mecanismo de limpieza condicional en `Test Teardown` que se ejecuta únicamente cuando un test case falla. Al finalizar, la suite será capaz de completar toda su ejecución reportando todas las fallas encontradas en un único ciclo, en lugar de detenerse ante el primer error.

---

## Objetivos de Aprendizaje

Al completar este laboratorio serás capaz de:

- [ ] Usar `Run Keyword And Expect Error` para verificar que una keyword falla con el mensaje de error exacto esperado en condiciones de borde.
- [ ] Implementar `Run Keyword And Continue On Failure` para acumular múltiples validaciones independientes sin detener el test case.
- [ ] Diseñar un `Test Teardown` con `Run Keyword If Test Failed` que ejecute lógica de limpieza condicional según el resultado del test.
- [ ] Construir una suite completa que diferencie fallas críticas (que detienen el test) de fallas no críticas (que se acumulan en el reporte).
- [ ] Leer e interpretar correctamente el reporte `log.html` de una suite con múltiples fallas acumuladas.

---

## Prerrequisitos

### Conocimiento previo

- Haber completado el **Laboratorio 03-00-01** con estructuras de control `IF/FOR/WHILE` funcionales.
- Comprensión de `Suite Setup`, `Suite Teardown`, `Test Setup` y `Test Teardown` a nivel de test case y suite.
- Familiaridad con la lectura de mensajes de error en `log.html` y `report.html`.
- Conocimiento básico de keywords de la librería `BuiltIn`: `Log`, `Fail`, `Should Be Equal`, `Should Contain`.

### Acceso y recursos

- Entorno virtual Python activo con Robot Framework 7.x instalado (heredado del Lab 03-00-01).
- Visual Studio Code con la extensión **Robot Framework Language Server** instalada.
- Conexión a internet **no requerida** para este laboratorio (todos los datos son ficticios y locales).

---

## Entorno de Laboratorio

### Requisitos de hardware

| Componente       | Mínimo requerido                              |
|------------------|-----------------------------------------------|
| Procesador       | Intel Core i5 8ª gen / AMD Ryzen 5 (4 núcleos)|
| RAM              | 8 GB                                          |
| Almacenamiento   | 500 MB libres para artefactos del laboratorio |
| Pantalla         | Resolución mínima 1280×768                    |

### Requisitos de software

| Software                        | Versión mínima |
|---------------------------------|----------------|
| Python                          | 3.10+          |
| Robot Framework                 | 7.x            |
| Visual Studio Code              | 1.85+          |
| Robot Framework Language Server | 1.12+          |

### Preparación del entorno

Abre una terminal y verifica que el entorno virtual del módulo 3 esté activo. Si no lo has activado aún, ejecuta los siguientes comandos según tu sistema operativo:

```bash
# ── Windows (cmd) ──────────────────────────────────────────────
cd C:\cursoRF\modulo03
venv\Scripts\activate

# ── Windows (PowerShell) ───────────────────────────────────────
cd C:\cursoRF\modulo03
.\venv\Scripts\Activate.ps1

# ── macOS / Linux (bash/zsh) ───────────────────────────────────
cd ~/cursoRF/modulo03
source venv/bin/activate
```

Confirma que Robot Framework está disponible:

```bash
robot --version
# Salida esperada: Robot Framework 7.x.x (Python 3.x.x ...)
```

Crea la estructura de carpetas para este laboratorio:

```bash
# ── Windows (cmd) ──────────────────────────────────────────────
mkdir lab_03_02\resources
mkdir lab_03_02\tests
mkdir lab_03_02\results

# ── macOS / Linux (bash/zsh) ───────────────────────────────────
mkdir -p lab_03_02/resources lab_03_02/tests lab_03_02/results
```

Cambia al directorio del laboratorio:

```bash
cd lab_03_02
```

> **Nota importante:** Todos los archivos creados en los pasos siguientes se ubican dentro de `lab_03_02/`. Asegúrate de estar posicionado en ese directorio antes de ejecutar cualquier comando `robot`.

---

## Instrucciones Paso a Paso

---

### Paso 1 — Crear el archivo de variables y datos del dominio de telecomunicaciones

**Objetivo:** Definir las variables de negocio que representan el estado de los servicios de telecomunicaciones. Estas variables serán la base de todas las validaciones del laboratorio.

#### Instrucciones

1. En Visual Studio Code, abre la carpeta `lab_03_02`.

2. Crea el archivo `resources/telecom_variables.resource` con el siguiente contenido:

```robotframework
*** Settings ***
Documentation    Variables de dominio para el flujo de activación de servicios
...              de telecomunicaciones — Empresa ficticia TelcoDemo S.A.

*** Variables ***
# ── Datos de cliente ───────────────────────────────────────────
${CLIENTE_ID}           CLI-2024-001
${CLIENTE_NOMBRE}       Juan Pérez
${CLIENTE_PLAN}         PLAN_FIBRA_500

# ── Estados válidos del servicio ───────────────────────────────
${ESTADO_PENDIENTE}     PENDIENTE
${ESTADO_ACTIVO}        ACTIVO
${ESTADO_SUSPENDIDO}    SUSPENDIDO
${ESTADO_ERROR}         ERROR

# ── Límites y umbrales de validación ───────────────────────────
${VELOCIDAD_MINIMA}     ${100}
${VELOCIDAD_MAXIMA}     ${1000}
${LATENCIA_MAXIMA}      ${50}
${PAQUETES_PERDIDOS_MAX}    ${0.05}

# ── Mensajes de error esperados (usados con Run Keyword And Expect Error) ──
${MSG_CLIENTE_INVALIDO}     Cliente no encontrado: CLI-INVALIDO
${MSG_PLAN_INVALIDO}        Plan de servicio no reconocido: PLAN_INEXISTENTE
${MSG_VELOCIDAD_FUERA_RANGO}    Velocidad fuera de rango permitido: -50
${MSG_ESTADO_INVALIDO}      Transición de estado inválida: ACTIVO -> PENDIENTE

# ── Variables de estado del sistema (se modifican durante los tests) ──
${SERVICIO_ACTIVADO}    ${FALSE}
${ROLLBACK_REQUERIDO}   ${FALSE}
```

3. Guarda el archivo.

#### Salida esperada

El archivo `resources/telecom_variables.resource` existe y Visual Studio Code no muestra errores de sintaxis en el panel de problemas.

#### Verificación

```bash
# Verifica que el archivo existe y tiene contenido
# ── Windows (cmd) ──────────────────────────────────────────────
type resources\telecom_variables.resource

# ── macOS / Linux ──────────────────────────────────────────────
cat resources/telecom_variables.resource
```

---

### Paso 2 — Crear el archivo Resource con keywords de negocio (incluyendo keywords que fallan intencionalmente)

**Objetivo:** Implementar las keywords de negocio que simulan el sistema de telecomunicaciones. Algunas keywords están diseñadas para **fallar intencionalmente** con mensajes de error específicos; estas serán el objetivo de `Run Keyword And Expect Error`.

#### Instrucciones

1. Crea el archivo `resources/telecom_keywords.resource`:

```robotframework
*** Settings ***
Documentation    Keywords de negocio para el flujo de activación de servicios
...              TelcoDemo S.A. — Módulo 3, Laboratorio 03-00-02
Library          BuiltIn
Resource         telecom_variables.resource

*** Keywords ***
# ══════════════════════════════════════════════════════════════
# KEYWORDS DE VALIDACIÓN — algunas fallan intencionalmente
# ══════════════════════════════════════════════════════════════

Validar Cliente
    [Documentation]    Valida que el ID de cliente existe en el sistema.
    ...                Falla con mensaje específico si el cliente no es válido.
    [Arguments]    ${cliente_id}
    Log    Validando cliente: ${cliente_id}
    # Simula la lógica de validación: solo acepta IDs con prefijo CLI-
    IF    not '${cliente_id}'.startswith('CLI-')
        Fail    Cliente no encontrado: ${cliente_id}
    END
    Log    Cliente ${cliente_id} validado correctamente

Validar Plan De Servicio
    [Documentation]    Verifica que el plan solicitado existe en el catálogo.
    ...                Falla con mensaje específico si el plan no existe.
    [Arguments]    ${plan}
    Log    Verificando plan de servicio: ${plan}
    @{planes_validos}=    Create List
    ...    PLAN_FIBRA_100    PLAN_FIBRA_500    PLAN_FIBRA_1000
    ...    PLAN_MOVIL_BASICO    PLAN_MOVIL_PRO
    ${plan_existe}=    Run Keyword And Return Status
    ...    Should Contain    ${planes_validos}    ${plan}
    IF    not ${plan_existe}
        Fail    Plan de servicio no reconocido: ${plan}
    END
    Log    Plan ${plan} encontrado en catálogo

Validar Velocidad De Servicio
    [Documentation]    Verifica que la velocidad contratada está dentro del rango permitido.
    ...                Falla si la velocidad es negativa o supera el máximo.
    [Arguments]    ${velocidad}
    Log    Verificando velocidad: ${velocidad} Mbps
    IF    ${velocidad} < 0
        Fail    Velocidad fuera de rango permitido: ${velocidad}
    ELSE IF    ${velocidad} > ${VELOCIDAD_MAXIMA}
        Fail    Velocidad supera el máximo permitido: ${velocidad}
    END
    Log    Velocidad ${velocidad} Mbps dentro del rango permitido

Validar Transicion De Estado
    [Documentation]    Verifica que la transición de estado solicitada es válida.
    ...                No se puede volver a PENDIENTE desde ACTIVO.
    [Arguments]    ${estado_actual}    ${estado_nuevo}
    Log    Verificando transición: ${estado_actual} -> ${estado_nuevo}
    IF    '${estado_actual}' == 'ACTIVO' and '${estado_nuevo}' == 'PENDIENTE'
        Fail    Transición de estado inválida: ${estado_actual} -> ${estado_nuevo}
    END
    Log    Transición ${estado_actual} -> ${estado_nuevo} permitida

# ══════════════════════════════════════════════════════════════
# KEYWORDS DE ACTIVACIÓN DEL SERVICIO
# ══════════════════════════════════════════════════════════════

Activar Servicio Para Cliente
    [Documentation]    Ejecuta el flujo completo de activación de servicio.
    ...                Establece SERVICIO_ACTIVADO en TRUE al completarse.
    [Arguments]    ${cliente_id}    ${plan}
    Log    Iniciando activación de servicio para ${cliente_id} con plan ${plan}
    Validar Cliente    ${cliente_id}
    Validar Plan De Servicio    ${plan}
    # Simula la activación en el sistema backend
    Log    Provisionando servicio en infraestructura...
    Log    Configurando parámetros de red...
    Log    Servicio activado exitosamente para ${cliente_id}
    Set Test Variable    ${SERVICIO_ACTIVADO}    ${TRUE}

Revertir Activacion De Servicio
    [Documentation]    Ejecuta el rollback del servicio activado.
    ...                Se llama desde el Teardown cuando el test falla.
    [Arguments]    ${cliente_id}
    Log    *** ROLLBACK: Revirtiendo activación de servicio para ${cliente_id} ***    WARN
    Log    Desaprovisionando recursos de red...
    Log    Liberando IP asignada...
    Log    Limpiando registros de facturación...
    Log    Rollback completado para ${cliente_id}
    Set Test Variable    ${SERVICIO_ACTIVADO}    ${FALSE}

Registrar Falla En Sistema De Monitoreo
    [Documentation]    Simula el registro de la falla en el sistema de monitoreo.
    ...                Se invoca desde el Teardown para trazabilidad.
    [Arguments]    ${cliente_id}    ${motivo}
    Log    [MONITOREO] Falla registrada — Cliente: ${cliente_id} | Motivo: ${motivo}    WARN

# ══════════════════════════════════════════════════════════════
# KEYWORDS DE VALIDACIÓN POST-ACTIVACIÓN (para Continue On Failure)
# ══════════════════════════════════════════════════════════════

Verificar Velocidad Downstream
    [Documentation]    Verifica que la velocidad de bajada cumple el SLA.
    [Arguments]    ${velocidad_medida}    ${velocidad_contratada}
    Log    Verificando velocidad downstream: ${velocidad_medida} Mbps (contratada: ${velocidad_contratada} Mbps)
    Should Be True    ${velocidad_medida} >= ${velocidad_contratada} * 0.8
    ...    La velocidad downstream ${velocidad_medida} Mbps está por debajo del 80% del SLA

Verificar Latencia
    [Documentation]    Verifica que la latencia está dentro del umbral aceptable.
    [Arguments]    ${latencia_ms}
    Log    Verificando latencia: ${latencia_ms} ms (máximo permitido: ${LATENCIA_MAXIMA} ms)
    Should Be True    ${latencia_ms} <= ${LATENCIA_MAXIMA}
    ...    Latencia ${latencia_ms} ms supera el máximo permitido de ${LATENCIA_MAXIMA} ms

Verificar Perdida De Paquetes
    [Documentation]    Verifica que la pérdida de paquetes está dentro del umbral.
    [Arguments]    ${porcentaje_perdida}
    Log    Verificando pérdida de paquetes: ${porcentaje_perdida}% (máximo: ${PAQUETES_PERDIDOS_MAX}%)
    Should Be True    ${porcentaje_perdida} <= ${PAQUETES_PERDIDOS_MAX}
    ...    Pérdida de paquetes ${porcentaje_perdida}% supera el umbral de ${PAQUETES_PERDIDOS_MAX}%

Verificar DNS Resuelve Correctamente
    [Documentation]    Simula la verificación de resolución DNS post-activación.
    [Arguments]    ${resultado_dns}
    Log    Verificando resolución DNS: ${resultado_dns}
    Should Be Equal    ${resultado_dns}    OK
    ...    La resolución DNS falló con resultado: ${resultado_dns}

# ══════════════════════════════════════════════════════════════
# KEYWORD DE TEARDOWN CONDICIONAL
# ══════════════════════════════════════════════════════════════

Teardown Condicional De Activacion
    [Documentation]    Teardown inteligente: solo ejecuta rollback si el test falló
    ...                y el servicio fue activado previamente.
    [Arguments]    ${cliente_id}
    Log    Ejecutando teardown condicional para ${cliente_id}
    Run Keyword If Test Failed
    ...    Ejecutar Rollback Si Servicio Activo    ${cliente_id}
    Run Keyword If Test Passed
    ...    Log    Test completado exitosamente — no se requiere rollback

Ejecutar Rollback Si Servicio Activo
    [Documentation]    Ejecuta rollback solo si el servicio fue activado en este test.
    [Arguments]    ${cliente_id}
    Log    Test falló — verificando si se requiere rollback...
    ${servicio_esta_activo}=    Get Variable Value    ${SERVICIO_ACTIVADO}    ${FALSE}
    IF    ${servicio_esta_activo}
        Log    Servicio activo detectado — iniciando rollback    WARN
        Revertir Activacion De Servicio    ${cliente_id}
        Registrar Falla En Sistema De Monitoreo
        ...    ${cliente_id}
        ...    Test fallido con servicio parcialmente activado
    ELSE
        Log    Servicio no fue activado — rollback no necesario
    END
```

2. Guarda el archivo.

#### Salida esperada

El archivo `resources/telecom_keywords.resource` existe sin errores de sintaxis. En VS Code, el panel de problemas no muestra errores de indentación ni de sintaxis Robot Framework.

#### Verificación

Ejecuta una validación de sintaxis con `--dryrun`:

```bash
robot --dryrun --outputdir results resources/telecom_keywords.resource
```

> **Nota:** Este comando fallará porque un archivo Resource no es ejecutable directamente, pero no debe mostrar errores de sintaxis. La salida esperada incluye `ERROR: Suite 'Telecom Keywords' contains no tests or tasks.` — esto es **normal** para un archivo Resource.

---

### Paso 3 — Implementar el primer test case: verificación de errores esperados con `Run Keyword And Expect Error`

**Objetivo:** Crear test cases que usen `Run Keyword And Expect Error` para confirmar que el sistema rechaza correctamente entradas inválidas con los mensajes de error exactos definidos en las variables.

#### Instrucciones

1. Crea el archivo `tests/suite_activacion_robusta.robot`:

```robotframework
*** Settings ***
Documentation    Suite robusta de pruebas para el flujo de activación de servicios
...              TelcoDemo S.A. — Demuestra manejo avanzado de errores y recuperación
...
...              Conceptos aplicados:
...              - Run Keyword And Expect Error
...              - Run Keyword And Continue On Failure
...              - Run Keyword If Test Failed / Test Passed
...              - Test Teardown con lógica condicional
Resource         ../resources/telecom_keywords.resource
Library          BuiltIn

Suite Setup      Log    Iniciando suite de pruebas de activación robusta TelcoDemo    console=True
Suite Teardown   Log    Suite de pruebas finalizada — revisar log.html para detalles    console=True

*** Test Cases ***

# ══════════════════════════════════════════════════════════════
# GRUPO 1: Verificación de errores esperados (Run Keyword And Expect Error)
# ══════════════════════════════════════════════════════════════

TC-01 Verificar Rechazo De Cliente Invalido
    [Documentation]    Confirma que el sistema rechaza un ID de cliente inválido
    ...                con el mensaje de error exacto esperado.
    ...                Usa Run Keyword And Expect Error para capturar el error.
    [Tags]    errores-esperados    validacion-cliente
    [Teardown]    Teardown Condicional De Activacion    ${CLIENTE_ID}

    Log    === TC-01: Verificando rechazo de cliente inválido ===
    Log    Se intentará activar servicio con ID de cliente inválido: CLI-INVALIDO

    # Run Keyword And Expect Error captura el error y verifica el mensaje exacto
    # Si la keyword NO falla, o falla con un mensaje DIFERENTE, este test falla
    Run Keyword And Expect Error
    ...    ${MSG_CLIENTE_INVALIDO}
    ...    Validar Cliente    CLI-INVALIDO

    Log    CORRECTO: El sistema rechazó al cliente inválido con el mensaje esperado

TC-02 Verificar Rechazo De Plan Inexistente
    [Documentation]    Confirma que el sistema rechaza un plan de servicio que no
    ...                existe en el catálogo, verificando el mensaje exacto.
    [Tags]    errores-esperados    validacion-plan
    [Teardown]    Teardown Condicional De Activacion    ${CLIENTE_ID}

    Log    === TC-02: Verificando rechazo de plan inexistente ===

    Run Keyword And Expect Error
    ...    ${MSG_PLAN_INVALIDO}
    ...    Validar Plan De Servicio    PLAN_INEXISTENTE

    Log    CORRECTO: El sistema rechazó el plan inexistente con el mensaje esperado

TC-03 Verificar Rechazo De Velocidad Negativa
    [Documentation]    Confirma que el sistema rechaza una velocidad negativa
    ...                en la validación de parámetros de servicio.
    [Tags]    errores-esperados    validacion-velocidad
    [Teardown]    Teardown Condicional De Activacion    ${CLIENTE_ID}

    Log    === TC-03: Verificando rechazo de velocidad negativa ===
    Log    Intentando configurar velocidad: -50 Mbps (valor inválido)

    Run Keyword And Expect Error
    ...    ${MSG_VELOCIDAD_FUERA_RANGO}
    ...    Validar Velocidad De Servicio    ${-50}

    Log    CORRECTO: El sistema rechazó la velocidad negativa con el mensaje esperado

TC-04 Verificar Rechazo De Transicion De Estado Invalida
    [Documentation]    Confirma que el sistema rechaza la transición ACTIVO -> PENDIENTE
    ...                que es una operación no permitida por las reglas de negocio.
    [Tags]    errores-esperados    validacion-estado
    [Teardown]    Teardown Condicional De Activacion    ${CLIENTE_ID}

    Log    === TC-04: Verificando rechazo de transición de estado inválida ===

    Run Keyword And Expect Error
    ...    ${MSG_ESTADO_INVALIDO}
    ...    Validar Transicion De Estado    ACTIVO    PENDIENTE

    Log    CORRECTO: El sistema rechazó la transición inválida con el mensaje esperado
```

2. Guarda el archivo (aún no está completo; se añadirán más test cases en los pasos siguientes).

#### Salida esperada

El archivo se guarda sin errores de sintaxis visibles en VS Code.

#### Verificación

Ejecuta únicamente el primer grupo de tests con la etiqueta `errores-esperados`:

```bash
robot --include errores-esperados --outputdir results tests/suite_activacion_robusta.robot
```

**Resultado esperado en consola:**

```
==============================================================================
Suite Activacion Robusta
==============================================================================
TC-01 Verificar Rechazo De Cliente Invalido                           | PASS |
TC-02 Verificar Rechazo De Plan Inexistente                           | PASS |
TC-03 Verificar Rechazo De Velocidad Negativa                         | PASS |
TC-04 Verificar Rechazo De Transicion De Estado Invalida              | PASS |
==============================================================================
Suite Activacion Robusta                                              | PASS |
4 tests, 4 passed, 0 failed
```

> **Punto de reflexión:** Los cuatro tests pasan porque `Run Keyword And Expect Error` "consume" el error. Si el mensaje de error no coincide exactamente con el esperado, el test **fallará**. Esto es precisamente lo que queremos: verificar que el sistema falla de la forma correcta.

---

### Paso 4 — Implementar test cases con `Run Keyword And Continue On Failure` para validaciones múltiples

**Objetivo:** Agregar test cases que ejecuten múltiples validaciones independientes dentro de un mismo test case, acumulando todas las fallas en el reporte sin detener la ejecución al primer error.

#### Instrucciones

1. Abre `tests/suite_activacion_robusta.robot` y agrega los siguientes test cases **después de TC-04**:

```robotframework
# ══════════════════════════════════════════════════════════════
# GRUPO 2: Validaciones múltiples con Continue On Failure
# ══════════════════════════════════════════════════════════════

TC-05 Validar Calidad De Servicio Post-Activacion Con Multiples Metricas
    [Documentation]    Activa el servicio y luego verifica múltiples métricas de calidad
    ...                de forma independiente. Usa Run Keyword And Continue On Failure
    ...                para que TODAS las métricas sean verificadas incluso si alguna falla.
    ...
    ...                COMPORTAMIENTO ESPERADO: Este test FALLA porque la latencia
    ...                (75ms) y la pérdida de paquetes (0.08%) superan los umbrales.
    ...                Sin embargo, TODAS las verificaciones se ejecutan y reportan.
    [Tags]    continue-on-failure    calidad-servicio
    [Teardown]    Teardown Condicional De Activacion    ${CLIENTE_ID}

    Log    === TC-05: Validación de calidad de servicio con múltiples métricas ===

    # Paso 1: Activar el servicio (operación crítica — si falla, detiene el test)
    Log    Paso 1: Activando servicio para el cliente...
    Activar Servicio Para Cliente    ${CLIENTE_ID}    ${CLIENTE_PLAN}
    Log    Servicio activado. Iniciando verificaciones de calidad...

    # Paso 2: Verificar métricas de calidad (no críticas — se acumulan los errores)
    # Simulamos métricas medidas: algunas correctas, algunas con problemas
    Log    Paso 2: Verificando métricas de calidad de forma independiente...

    # Esta verificación PASA (velocidad 420 Mbps >= 80% de 500 Mbps = 400 Mbps)
    Run Keyword And Continue On Failure
    ...    Verificar Velocidad Downstream    ${420}    ${500}

    # Esta verificación FALLA (latencia 75ms > máximo 50ms)
    Run Keyword And Continue On Failure
    ...    Verificar Latencia    ${75}

    # Esta verificación FALLA (pérdida 0.08% > máximo 0.05%)
    Run Keyword And Continue On Failure
    ...    Verificar Perdida De Paquetes    ${0.08}

    # Esta verificación PASA (DNS responde OK)
    Run Keyword And Continue On Failure
    ...    Verificar DNS Resuelve Correctamente    OK

    Log    Todas las verificaciones de calidad completadas — revisar log.html para detalles

TC-06 Validar Calidad De Servicio Optimo
    [Documentation]    Verifica métricas de calidad con valores que cumplen todos los SLAs.
    ...                Todas las verificaciones deben pasar.
    ...                COMPORTAMIENTO ESPERADO: Este test PASA completamente.
    [Tags]    continue-on-failure    calidad-servicio    camino-feliz
    [Teardown]    Teardown Condicional De Activacion    ${CLIENTE_ID}

    Log    === TC-06: Validación de servicio óptimo (todos los SLAs cumplidos) ===

    # Activar servicio
    Activar Servicio Para Cliente    ${CLIENTE_ID}    ${CLIENTE_PLAN}

    # Verificar métricas óptimas — todas deben pasar
    Run Keyword And Continue On Failure
    ...    Verificar Velocidad Downstream    ${490}    ${500}

    Run Keyword And Continue On Failure
    ...    Verificar Latencia    ${25}

    Run Keyword And Continue On Failure
    ...    Verificar Perdida De Paquetes    ${0.01}

    Run Keyword And Continue On Failure
    ...    Verificar DNS Resuelve Correctamente    OK

    Log    === Servicio verificado: TODOS los SLAs cumplidos ===

TC-07 Validar Multiples Planes Con Errores Mixtos
    [Documentation]    Verifica la validación de múltiples planes en una sola ejecución.
    ...                Mezcla planes válidos e inválidos para demostrar que Continue On Failure
    ...                permite reportar todos los problemas en una sola pasada.
    ...                COMPORTAMIENTO ESPERADO: Este test FALLA por los planes inválidos,
    ...                pero TODOS los planes son evaluados.
    [Tags]    continue-on-failure    validacion-plan    data-driven-manual
    [Teardown]    Teardown Condicional De Activacion    ${CLIENTE_ID}

    Log    === TC-07: Validación de múltiples planes (mezcla válidos/inválidos) ===
    Log    Verificando catálogo de planes para migración masiva de clientes...

    # Plan válido — PASA
    Run Keyword And Continue On Failure
    ...    Validar Plan De Servicio    PLAN_FIBRA_100

    # Plan inválido — FALLA (acumulado, no detiene el test)
    Run Keyword And Continue On Failure
    ...    Validar Plan De Servicio    PLAN_COAXIAL_LEGACY

    # Plan válido — PASA
    Run Keyword And Continue On Failure
    ...    Validar Plan De Servicio    PLAN_MOVIL_PRO

    # Plan inválido — FALLA (acumulado)
    Run Keyword And Continue On Failure
    ...    Validar Plan De Servicio    PLAN_SATELITAL_BETA

    # Plan válido — PASA
    Run Keyword And Continue On Failure
    ...    Validar Plan De Servicio    PLAN_FIBRA_1000

    Log    Revisión de catálogo completada — 2 planes inválidos detectados (ver log.html)
```

2. Guarda el archivo.

#### Salida esperada

Al ejecutar el grupo `continue-on-failure`, TC-05 y TC-07 fallan pero **completan toda su ejecución**. TC-06 pasa completamente.

#### Verificación

```bash
robot --include continue-on-failure --outputdir results tests/suite_activacion_robusta.robot
```

**Resultado esperado en consola:**

```
TC-05 Validar Calidad De Servicio Post-Activacion Con Multiples Metricas | FAIL |
Several failures occurred:
1) Latencia 75 ms supera el máximo permitido de 50 ms
2) Pérdida de paquetes 0.08% supera el umbral de 0.05%

TC-06 Validar Calidad De Servicio Optimo                               | PASS |

TC-07 Validar Multiples Planes Con Errores Mixtos                      | FAIL |
Several failures occurred:
1) Plan de servicio no reconocido: PLAN_COAXIAL_LEGACY
2) Plan de servicio no reconocido: PLAN_SATELITAL_BETA
```

> **Observación clave:** Nota la frase **"Several failures occurred"** en el reporte. Esta es la señal visual de que `Run Keyword And Continue On Failure` acumuló múltiples fallas. Ábrela en `log.html` para ver cada falla listada individualmente.

---

### Paso 5 — Implementar el Test Teardown con lógica condicional

**Objetivo:** Verificar el comportamiento del `Test Teardown` condicional ya implementado en `telecom_keywords.resource`, y agregar un test case que demuestre explícitamente el rollback cuando el test falla con el servicio parcialmente activado.

#### Instrucciones

1. Agrega los siguientes test cases al final de `tests/suite_activacion_robusta.robot`:

```robotframework
# ══════════════════════════════════════════════════════════════
# GRUPO 3: Teardown condicional y recuperación del sistema
# ══════════════════════════════════════════════════════════════

TC-08 Falla Critica Despues De Activacion Requiere Rollback
    [Documentation]    Demuestra el mecanismo de rollback en Teardown.
    ...                El servicio se activa exitosamente, pero luego ocurre
    ...                una falla crítica que detiene el test.
    ...                El Teardown detecta que el test falló Y que el servicio
    ...                está activo, y ejecuta el rollback automáticamente.
    ...
    ...                COMPORTAMIENTO ESPERADO:
    ...                - El test FALLA (intencionalmente)
    ...                - El Teardown ejecuta el rollback (visible en log.html)
    ...                - El sistema queda en estado limpio
    [Tags]    teardown-condicional    rollback
    [Teardown]    Teardown Condicional De Activacion    ${CLIENTE_ID}

    Log    === TC-08: Falla crítica post-activación con rollback automático ===

    # Paso 1: Activación exitosa
    Log    Paso 1: Activando servicio...
    Activar Servicio Para Cliente    ${CLIENTE_ID}    ${CLIENTE_PLAN}
    Log    Servicio activado exitosamente. Estado: ${SERVICIO_ACTIVADO}

    # Paso 2: Validaciones previas a la falla
    Log    Paso 2: Ejecutando validaciones post-activación...
    Validar Velocidad De Servicio    ${500}
    Validar Transicion De Estado    ${ESTADO_PENDIENTE}    ${ESTADO_ACTIVO}

    # Paso 3: Falla crítica simulada (detiene el test inmediatamente)
    Log    Paso 3: Detectando condición crítica en el sistema...    WARN
    Fail
    ...    ERROR CRÍTICO: Fallo en el sistema de facturación post-activación — rollback requerido

TC-09 Activacion Exitosa Sin Rollback En Teardown
    [Documentation]    Demuestra que el Teardown NO ejecuta rollback cuando el test pasa.
    ...                Verifica el comportamiento de Run Keyword If Test Passed.
    ...
    ...                COMPORTAMIENTO ESPERADO:
    ...                - El test PASA completamente
    ...                - El Teardown registra éxito pero NO ejecuta rollback
    [Tags]    teardown-condicional    camino-feliz
    [Teardown]    Teardown Condicional De Activacion    ${CLIENTE_ID}

    Log    === TC-09: Activación exitosa — Teardown no debe ejecutar rollback ===

    Activar Servicio Para Cliente    ${CLIENTE_ID}    ${CLIENTE_PLAN}
    Validar Velocidad De Servicio    ${500}
    Validar Transicion De Estado    ${ESTADO_PENDIENTE}    ${ESTADO_ACTIVO}

    Log    Activación completada sin errores — Teardown solo registrará éxito

TC-10 Falla Antes De Activacion No Requiere Rollback
    [Documentation]    Demuestra que el Teardown NO ejecuta rollback cuando el test
    ...                falla ANTES de que el servicio sea activado.
    ...                La variable SERVICIO_ACTIVADO permanece en FALSE.
    ...
    ...                COMPORTAMIENTO ESPERADO:
    ...                - El test FALLA (por cliente inválido)
    ...                - El Teardown detecta que el servicio NO fue activado
    ...                - No se ejecuta rollback (no hay nada que revertir)
    [Tags]    teardown-condicional    rollback    errores-esperados
    [Teardown]    Teardown Condicional De Activacion    CLI-INVALIDO

    Log    === TC-10: Falla antes de activación — rollback no debe ejecutarse ===
    Log    Intentando activar servicio con cliente inválido...

    # Esta llamada falla inmediatamente — SERVICIO_ACTIVADO nunca se establece en TRUE
    Activar Servicio Para Cliente    CLI-INVALIDO    ${CLIENTE_PLAN}

    Log    Esta línea NUNCA se ejecuta porque la anterior falla
```

2. Guarda el archivo.

#### Salida esperada

Los tres test cases se agregan sin errores de sintaxis.

#### Verificación

```bash
robot --include teardown-condicional --outputdir results tests/suite_activacion_robusta.robot
```

**Resultado esperado en consola:**

```
TC-08 Falla Critica Despues De Activacion Requiere Rollback            | FAIL |
ERROR CRÍTICO: Fallo en el sistema de facturación post-activación — rollback requerido

TC-09 Activacion Exitosa Sin Rollback En Teardown                      | PASS |

TC-10 Falla Antes De Activacion No Requiere Rollback                   | FAIL |
Cliente no encontrado: CLI-INVALIDO
```

Abre `results/log.html` y verifica que:
- En **TC-08**: el Teardown muestra los mensajes `*** ROLLBACK: Revirtiendo activación...` y `Rollback completado`.
- En **TC-09**: el Teardown muestra `Test completado exitosamente — no se requiere rollback`.
- En **TC-10**: el Teardown muestra `Servicio no fue activado — rollback no necesario`.

---

### Paso 6 — Ejecutar la suite completa y analizar el reporte

**Objetivo:** Ejecutar todos los test cases en una sola pasada y analizar el reporte HTML para comprender la diferencia entre fallas críticas y no críticas.

#### Instrucciones

1. Ejecuta la suite completa con un nombre de reporte descriptivo:

```bash
robot --outputdir results \
      --log log_suite_robusta.html \
      --report report_suite_robusta.html \
      --output output_suite_robusta.xml \
      tests/suite_activacion_robusta.robot
```

> **Windows (cmd) — usa `^` para continuar en la siguiente línea:**

```cmd
robot --outputdir results ^
      --log log_suite_robusta.html ^
      --report report_suite_robusta.html ^
      --output output_suite_robusta.xml ^
      tests\suite_activacion_robusta.robot
```

2. Observa el resumen en consola. Deberías ver algo similar a:

```
==============================================================================
Suite Activacion Robusta
==============================================================================
TC-01 Verificar Rechazo De Cliente Invalido                           | PASS |
TC-02 Verificar Rechazo De Plan Inexistente                           | PASS |
TC-03 Verificar Rechazo De Velocidad Negativa                         | PASS |
TC-04 Verificar Rechazo De Transicion De Estado Invalida              | PASS |
TC-05 Validar Calidad De Servicio Post-Activacion Con Multiples ...   | FAIL |
TC-06 Validar Calidad De Servicio Optimo                              | PASS |
TC-07 Validar Multiples Planes Con Errores Mixtos                     | FAIL |
TC-08 Falla Critica Despues De Activacion Requiere Rollback           | FAIL |
TC-09 Activacion Exitosa Sin Rollback En Teardown                     | PASS |
TC-10 Falla Antes De Activacion No Requiere Rollback                  | FAIL |
==============================================================================
Suite Activacion Robusta                                              | FAIL |
10 tests, 5 passed, 5 failed
```

3. Abre `results/log_suite_robusta.html` en tu navegador.

4. Realiza las siguientes observaciones en el log:

   **a) Observación en TC-05:** Expande el test case y verifica que aparecen **exactamente 2 entradas de falla** bajo la sección "Several failures occurred". Confirma que las keywords `Verificar Velocidad Downstream` y `Verificar DNS Resuelve Correctamente` aparecen en verde (PASS) mientras que `Verificar Latencia` y `Verificar Perdida De Paquetes` aparecen en rojo (FAIL).

   **b) Observación en TC-08:** Expande la sección **Teardown** del test case. Verifica que aparece la secuencia completa de rollback: mensajes de desaprovisionamiento, liberación de IP y limpieza de registros.

   **c) Observación en TC-10:** Expande el Teardown y confirma que aparece el mensaje `Servicio no fue activado — rollback no necesario`, sin ningún mensaje de rollback.

#### Salida esperada

El archivo `results/log_suite_robusta.html` se abre correctamente en el navegador y muestra los 10 test cases con sus respectivos estados y detalles de ejecución.

#### Verificación

```bash
# Verifica que los archivos de reporte fueron generados
# ── Windows (cmd) ──────────────────────────────────────────────
dir results\

# ── macOS / Linux ──────────────────────────────────────────────
ls -la results/
```

Debes ver al menos:
- `log_suite_robusta.html`
- `report_suite_robusta.html`
- `output_suite_robusta.xml`

---

## Validación y Pruebas

### Lista de verificación final

Completa la siguiente lista de verificación antes de dar por finalizado el laboratorio. Cada ítem debe poder verificarse directamente en `log.html`:

| # | Verificación | Cómo confirmarlo |
|---|---|---|
| 1 | TC-01 al TC-04 pasan usando `Run Keyword And Expect Error` | Estado PASS en report.html, sin errores en log |
| 2 | TC-05 muestra "Several failures occurred" con exactamente 2 fallas | Expandir TC-05 en log.html |
| 3 | TC-05 ejecutó las 4 verificaciones (2 PASS + 2 FAIL) | Contar entradas verdes y rojas en TC-05 |
| 4 | TC-06 pasa completamente con todas las métricas en verde | Estado PASS + 4 entradas verdes en TC-06 |
| 5 | TC-07 muestra exactamente 2 fallas de planes inválidos | Expandir TC-07, contar fallas rojas |
| 6 | TC-08 Teardown muestra la secuencia completa de rollback | Expandir sección Teardown de TC-08 |
| 7 | TC-09 Teardown muestra "no se requiere rollback" | Expandir sección Teardown de TC-09 |
| 8 | TC-10 Teardown muestra "rollback no necesario" (sin rollback ejecutado) | Expandir sección Teardown de TC-10 |
| 9 | El resumen final es: 10 tests, 5 passed, 5 failed | Encabezado de report.html |
| 10 | Ningún test case muestra estado "ERROR" (solo PASS o FAIL) | Columna de estado en report.html |

### Prueba de regresión: verificar que el mensaje de error importa

Ejecuta el siguiente comando para confirmar que `Run Keyword And Expect Error` falla cuando el mensaje NO coincide. Crea temporalmente un archivo de prueba:

```bash
# ── macOS / Linux ──────────────────────────────────────────────
cat > /tmp/test_mensaje_incorrecto.robot << 'EOF'
*** Settings ***
Resource    resources/telecom_keywords.resource

*** Test Cases ***
TC-REGRESION Mensaje De Error Incorrecto Debe Fallar El Test
    [Documentation]    Este test DEBE fallar porque el mensaje esperado
    ...                no coincide con el mensaje real del sistema.
    Run Keyword And Expect Error
    ...    Este mensaje NO es el correcto
    ...    Validar Cliente    CLI-INVALIDO
EOF
robot --outputdir results /tmp/test_mensaje_incorrecto.robot

# ── Windows (cmd) — crea el archivo manualmente en VS Code y ejecuta:
robot --outputdir results tests\test_mensaje_incorrecto.robot
```

**Resultado esperado:** El test falla con un mensaje similar a:

```
Expected error message 'Este mensaje NO es el correcto' but got
'Cliente no encontrado: CLI-INVALIDO'
```

Esto confirma que `Run Keyword And Expect Error` verifica el mensaje con precisión exacta.

---

## Solución de Problemas

### Problema 1: `Run Keyword And Expect Error` falla con "Expected error message ... but got ..."

**Síntoma:** Un test case del Grupo 1 (TC-01 al TC-04) falla con un mensaje como:
```
Expected error message 'Cliente no encontrado: CLI-INVALIDO' but got
'AssertionError: Cliente no encontrado: CLI-INVALIDO'
```

**Causa:** Robot Framework a veces antepone el tipo de excepción Python (`AssertionError:`) al mensaje cuando la keyword usa `Fail` internamente. La versión de RF o la forma en que se lanza el error puede incluir o no este prefijo.

**Solución:** Usa el comodín `*` al inicio del mensaje esperado para ignorar el prefijo:

```robotframework
# En lugar de:
Run Keyword And Expect Error
...    Cliente no encontrado: CLI-INVALIDO
...    Validar Cliente    CLI-INVALIDO

# Usa el comodín:
Run Keyword And Expect Error
...    *Cliente no encontrado: CLI-INVALIDO
...    Validar Cliente    CLI-INVALIDO
```

Alternativamente, actualiza la variable en `telecom_variables.resource`:

```robotframework
${MSG_CLIENTE_INVALIDO}    *Cliente no encontrado: CLI-INVALIDO
```

El asterisco `*` al inicio indica que el mensaje puede comenzar con cualquier texto antes de la cadena especificada.

---

### Problema 2: El Teardown no ejecuta el rollback en TC-08 aunque el test falló

**Síntoma:** En `log.html`, el Teardown de TC-08 muestra el mensaje `Servicio no fue activado — rollback no necesario` en lugar de la secuencia de rollback, a pesar de que el test falló después de activar el servicio.

**Causa:** La variable `${SERVICIO_ACTIVADO}` no se propagó correctamente al scope del test case. Esto ocurre cuando `Set Test Variable` se llama desde dentro de una keyword anidada y la variable no se establece en el scope correcto, o cuando el test se ejecuta de forma aislada sin que `${SERVICIO_ACTIVADO}` tenga un valor inicial definido.

**Solución:** Verifica que la keyword `Activar Servicio Para Cliente` use explícitamente `Set Test Variable` (no `Set Variable`):

```robotframework
# CORRECTO — usa Set Test Variable para que sea visible en el Teardown
Set Test Variable    ${SERVICIO_ACTIVADO}    ${TRUE}

# INCORRECTO — Set Variable solo crea una variable local a la keyword
${SERVICIO_ACTIVADO}=    Set Variable    ${TRUE}
```

Además, verifica que `Get Variable Value` en `Ejecutar Rollback Si Servicio Activo` usa el valor por defecto `${FALSE}` como tercer argumento para evitar errores si la variable nunca fue establecida:

```robotframework
${servicio_esta_activo}=    Get Variable Value    ${SERVICIO_ACTIVADO}    ${FALSE}
```

Si el problema persiste, agrega un `Log Variables` al inicio del Teardown para inspeccionar el estado de todas las variables disponibles en ese momento.

---

## Limpieza

Una vez completado el laboratorio y verificada la lista de verificación, ejecuta los siguientes pasos de limpieza:

1. **Elimina archivos temporales de prueba** (si los creaste):

```bash
# ── macOS / Linux ──────────────────────────────────────────────
rm -f /tmp/test_mensaje_incorrecto.robot

# ── Windows (cmd) ──────────────────────────────────────────────
del tests\test_mensaje_incorrecto.robot
```

2. **Archiva los resultados** para referencia futura:

```bash
# ── macOS / Linux ──────────────────────────────────────────────
cp -r results results_backup_lab03_02

# ── Windows (cmd) ──────────────────────────────────────────────
xcopy results results_backup_lab03_02 /E /I
```

3. **Desactiva el entorno virtual** al finalizar la sesión:

```bash
deactivate
```

4. **Crea una copia de respaldo del proyecto completo** (recomendado antes del siguiente módulo):

```bash
# ── macOS / Linux ──────────────────────────────────────────────
cd ..
cp -r lab_03_02 lab_03_02_BACKUP_$(date +%Y%m%d)

# ── Windows (cmd) ──────────────────────────────────────────────
cd ..
xcopy lab_03_02 lab_03_02_BACKUP /E /I
```

> **Recordatorio:** El proyecto `lab_03_02` puede ser requerido como base para laboratorios del Módulo 4. Mantén la copia de respaldo accesible.

---

## Resumen

En este laboratorio construiste una suite de prueba robusta para un flujo de activación de servicios de telecomunicaciones, aplicando tres mecanismos avanzados de manejo de errores de Robot Framework:

| Mecanismo | Cuándo usarlo | Efecto en el test |
|---|---|---|
| `Run Keyword And Expect Error` | Verificar que el sistema rechaza entradas inválidas con el mensaje correcto | El test **pasa** si el error coincide; **falla** si no hay error o el mensaje difiere |
| `Run Keyword And Continue On Failure` | Ejecutar múltiples validaciones independientes en un mismo test | El test **acumula** fallas y las reporta todas; no se detiene ante la primera |
| `Run Keyword If Test Failed` en Teardown | Ejecutar lógica de limpieza condicional post-falla | El rollback solo se ejecuta cuando es necesario, evitando efectos secundarios |

### Conceptos clave consolidados

- **Falla crítica vs. no crítica:** Una falla crítica (keyword sin `Continue On Failure`) detiene el test inmediatamente. Una falla no crítica (envuelta en `Run Keyword And Continue On Failure`) se acumula y el test continúa.
- **El Teardown siempre se ejecuta:** Independientemente del resultado del test case, el `Test Teardown` se ejecuta. La lógica condicional con `Run Keyword If Test Failed` permite decidir **qué hacer** dentro del Teardown según ese resultado.
- **`Get Variable Value` con valor por defecto:** Es una práctica defensiva esencial en Teardowns para evitar errores cuando una variable puede no haber sido establecida si el test falló antes de alcanzar ese punto.
- **Mensajes de error exactos:** `Run Keyword And Expect Error` verifica el mensaje con precisión. El comodín `*` al inicio permite ignorar prefijos variables del tipo de excepción.

### Recursos adicionales

- [Documentación oficial: BuiltIn — Run Keyword And Expect Error](https://robotframework.org/robotframework/latest/libraries/BuiltIn.html#Run%20Keyword%20And%20Expect%20Error)
- [Documentación oficial: BuiltIn — Run Keyword And Continue On Failure](https://robotframework.org/robotframework/latest/libraries/BuiltIn.html#Run%20Keyword%20And%20Continue%20On%20Failure)
- [Documentación oficial: BuiltIn — Run Keyword If Test Failed](https://robotframework.org/robotframework/latest/libraries/BuiltIn.html#Run%20Keyword%20If%20Test%20Failed)
- [Guía del usuario RF: Test Setup y Teardown](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#test-setup-and-teardown)
- [Guía del usuario RF: Continuar en caso de fallo](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#continuing-on-failure)

---
*Laboratorio 03-00-02 — Módulo 3: Control de Flujo y Manejo de Errores | Robot Framework 7.x*
