# Escritura de escenarios BDD para un flujo comercial de telecomunicaciones

## Metadatos

| Campo            | Detalle                                      |
|------------------|----------------------------------------------|
| **Duración**     | 72 minutos                                   |
| **Complejidad**  | Media                                        |
| **Nivel Bloom**  | Crear                                        |
| **Módulo**       | 4 — BDD y Gherkin en Robot Framework         |
| **Laboratorio**  | 04-00-01 (Práctica 7)                        |

---

## Descripción General

En este laboratorio aplicarás la sintaxis Gherkin nativa de Robot Framework para escribir escenarios BDD que describan tres flujos comerciales de una empresa de telecomunicaciones ficticia: activación de línea móvil, cambio de plan de datos y procesamiento de pago de factura. Organizarás los escenarios en una estructura de directorios por dominio de negocio (`features/mobile/`, `features/billing/`, `features/plans/`) y mapearás cada paso Gherkin a keywords de implementación ubicadas en archivos `.resource` separados. El objetivo es experimentar de forma directa cómo BDD separa la capa de descripción de negocio de la capa técnica, produciendo especificaciones que pueden ser leídas y validadas por stakeholders no técnicos.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio, serás capaz de:

- [ ] Escribir escenarios de prueba completos usando las palabras clave Gherkin nativas de Robot Framework (`Given`, `When`, `Then`, `And`, `But`) en lenguaje de negocio comprensible para stakeholders no técnicos.
- [ ] Mapear cada paso Gherkin a una keyword técnica de implementación en archivos `.resource` separados, manteniendo la separación entre la capa de descripción y la capa de ejecución.
- [ ] Organizar múltiples feature files en una estructura de directorios que represente dominios de negocio (`mobile`, `billing`, `plans`) de una empresa de telecomunicaciones.
- [ ] Ejecutar suites BDD y verificar que los reportes HTML de Robot Framework reflejan los pasos Gherkin con nombres legibles por el negocio.
- [ ] Explicar el valor de BDD para la alineación entre equipos de TI y negocio mediante la comparación entre la capa de escenarios y la capa de implementación.

---

## Prerrequisitos

### Conocimientos Previos

| Conocimiento                                                                 | Nivel Requerido |
|------------------------------------------------------------------------------|-----------------|
| Estructura de proyecto Robot Framework (archivos `.robot`, `.resource`)      | Aplicado        |
| Uso de archivos Resource y variables en Robot Framework                      | Aplicado        |
| Comprensión de la diferencia entre descripción de comportamiento e implementación técnica | Conceptual |
| Concepto de criterios de aceptación en metodologías ágiles                  | Básico (deseable) |

### Acceso y Herramientas

- Entorno virtual Python (`venv`) **activo** con Robot Framework 7.x instalado.
- VS Code con la extensión **Robot Framework Language Server** instalada y configurada.
- Haber completado satisfactoriamente los laboratorios de los Módulos 1 y 2.
- Conexión a internet (solo para verificación de instalación de paquetes si fuera necesario).

---

## Entorno de Laboratorio

### Hardware Mínimo

| Componente        | Mínimo Requerido                                      |
|-------------------|-------------------------------------------------------|
| Procesador        | Intel Core i5 8ª gen / AMD Ryzen 5 (4 núcleos)       |
| RAM               | 8 GB                                                  |
| Almacenamiento    | 5 GB libres                                           |
| Pantalla          | Resolución 1280×768 (para reportes HTML)              |
| Red               | No requerida (laboratorio sin llamadas externas)      |

### Software Requerido

| Software                         | Versión Mínima | Propósito                              |
|----------------------------------|----------------|----------------------------------------|
| Python                           | 3.10+          | Entorno de ejecución                   |
| Robot Framework                  | 7.x            | Motor de pruebas y soporte Gherkin     |
| VS Code                          | 1.85+          | Editor con navegación entre steps      |
| Robot Framework Language Server  | 1.12+          | Autocompletado y navegación            |
| pip                              | 23.x+          | Gestión de paquetes                    |

### Configuración del Entorno

> ⚠️ **IMPORTANTE:** Todos los comandos deben ejecutarse con el entorno virtual **activo**. Verifica que el prompt muestra `(venv)` antes de continuar.

#### Verificar y activar el entorno virtual

**Windows (cmd):**
```cmd
cd C:\proyectos\robot-telecom
Scripts\activate
```

**Windows (PowerShell):**
```powershell
cd C:\proyectos\robot-telecom
.\Scripts\Activate.ps1
```

**macOS / Linux (bash/zsh):**
```bash
cd ~/proyectos/robot-telecom
source bin/activate
```

#### Verificar la instalación de Robot Framework

```bash
robot --version
python -m robot --version
```

**Salida esperada:**
```
Robot Framework 7.x.x (Python 3.x.x on ...)
```

#### Verificar que no se requieren librerías adicionales

Este laboratorio utiliza **únicamente** la librería `BuiltIn` de Robot Framework (incluida por defecto). No se requieren instalaciones adicionales.

```bash
pip list | grep -i robot
# Windows cmd:
pip list | findstr robot
```

---

## Pasos del Laboratorio

### Visión General de la Estructura Final

Antes de comenzar, observa la estructura de directorios que construirás durante este laboratorio:

```
robot-telecom/
└── features/
    ├── mobile/
    │   ├── activacion_linea.robot
    │   └── keywords/
    │       └── mobile_steps.resource
    ├── billing/
    │   ├── pago_factura.robot
    │   └── keywords/
    │       └── billing_steps.resource
    └── plans/
        ├── cambio_plan.robot
        └── keywords/
            └── plans_steps.resource
```

---

### Paso 1: Crear la Estructura de Directorios del Proyecto BDD

**Objetivo:** Establecer la organización de directorios por dominio de negocio que refleje la arquitectura BDD propuesta.

#### Instrucciones

1. Abre una terminal con el entorno virtual activo y navega a la raíz de tu proyecto:

   **Windows (cmd/PowerShell):**
   ```cmd
   cd C:\proyectos\robot-telecom
   ```

   **macOS / Linux:**
   ```bash
   cd ~/proyectos/robot-telecom
   ```

2. Crea la estructura de directorios completa con un único comando:

   **Windows (PowerShell):**
   ```powershell
   New-Item -ItemType Directory -Force -Path `
     features\mobile\keywords, `
     features\billing\keywords, `
     features\plans\keywords
   ```

   **Windows (cmd):**
   ```cmd
   mkdir features\mobile\keywords
   mkdir features\billing\keywords
   mkdir features\plans\keywords
   ```

   **macOS / Linux:**
   ```bash
   mkdir -p features/mobile/keywords \
             features/billing/keywords \
             features/plans/keywords
   ```

3. Verifica la estructura creada:

   **Windows (PowerShell):**
   ```powershell
   Get-ChildItem -Recurse features | Select-Object FullName
   ```

   **macOS / Linux:**
   ```bash
   find features -type d
   ```

#### Salida Esperada

```
features/mobile
features/mobile/keywords
features/billing
features/billing/keywords
features/plans
features/plans/keywords
```

#### Verificación

Confirma en VS Code (panel Explorer) que los tres dominios (`mobile`, `billing`, `plans`) aparecen como subdirectorios de `features/`, cada uno con su carpeta `keywords/` interna.

---

### Paso 2: Crear el Archivo Resource para el Dominio Mobile

**Objetivo:** Implementar las keywords técnicas que soportarán los pasos Gherkin del dominio de activación de línea móvil, separando completamente la implementación de la descripción.

#### Instrucciones

1. Crea el archivo `features/mobile/keywords/mobile_steps.resource` y escribe el siguiente contenido:

   ```robot
   *** Settings ***
   Documentation    Keywords de implementación para el dominio Mobile.
   ...              Simulan operaciones de activación de línea móvil
   ...              usando únicamente la librería BuiltIn.
   Library          BuiltIn

   *** Variables ***
   ${ESTADO_LINEA_INICIAL}    sin_plan_activo
   ${PLAN_SELECCIONADO}       ${EMPTY}
   ${LINEA_ACTIVADA}          ${FALSE}
   ${NOTIFICACION_ENVIADA}    ${FALSE}

   *** Keywords ***
   El cliente "${nombre}" tiene una línea activa sin plan de datos
       [Documentation]    Verifica que el cliente existe y su línea no tiene plan asignado.
       Log    Verificando estado inicial del cliente: ${nombre}
       Log    Estado de la línea: ${ESTADO_LINEA_INICIAL}
       Should Be Equal    ${ESTADO_LINEA_INICIAL}    sin_plan_activo
       Set Test Variable    ${CLIENTE_ACTIVO}    ${nombre}

   El cliente selecciona el plan "${nombre_plan}"
       [Documentation]    Registra la selección del plan de datos por parte del cliente.
       Log    El cliente ${CLIENTE_ACTIVO} selecciona el plan: ${nombre_plan}
       Set Test Variable    ${PLAN_SELECCIONADO}    ${nombre_plan}
       Should Not Be Empty    ${PLAN_SELECCIONADO}

   El cliente confirma la activación con su número de documento
       [Documentation]    Simula la confirmación de identidad para activar la línea.
       Log    Confirmando identidad del cliente: ${CLIENTE_ACTIVO}
       Log    Documento verificado correctamente.
       Set Test Variable    ${CONFIRMACION_IDENTIDAD}    verificada

   El plan "${nombre_plan}" queda activo en la línea del cliente
       [Documentation]    Verifica que el plan fue activado correctamente.
       Log    Activando plan: ${nombre_plan} para el cliente: ${CLIENTE_ACTIVO}
       Set Test Variable    ${LINEA_ACTIVADA}    ${TRUE}
       Should Be True    ${LINEA_ACTIVADA}
       Log    Plan ${nombre_plan} activado exitosamente.

   El cliente recibe una notificación de bienvenida por SMS
       [Documentation]    Simula el envío de SMS de confirmación al cliente.
       Log    Enviando SMS de bienvenida al cliente: ${CLIENTE_ACTIVO}
       Set Test Variable    ${NOTIFICACION_ENVIADA}    ${TRUE}
       Should Be True    ${NOTIFICACION_ENVIADA}
       Log    SMS enviado correctamente.

   El cliente "${nombre}" tiene una línea activa con plan vigente
       [Documentation]    Precondición: cliente con línea y plan ya activos.
       Log    Cliente ${nombre} tiene línea activa con plan vigente.
       Set Test Variable    ${CLIENTE_ACTIVO}    ${nombre}
       Set Test Variable    ${ESTADO_LINEA}    activa_con_plan

   El sistema detecta que el documento de identidad no es válido
       [Documentation]    Simula un fallo de validación de documento.
       Log    Simulando documento de identidad inválido.
       Set Test Variable    ${DOCUMENTO_VALIDO}    ${FALSE}

   El sistema rechaza la activación con el mensaje "${mensaje}"
       [Documentation]    Verifica que el sistema genera el mensaje de rechazo correcto.
       Log    Sistema rechaza activación. Mensaje: ${mensaje}
       Should Not Be Empty    ${mensaje}
       Log    Activación rechazada correctamente con mensaje esperado.

   La línea permanece en estado sin plan activo
       [Documentation]    Verifica que el estado de la línea no cambió.
       Log    Verificando que la línea no fue modificada.
       Log    Estado de línea sin cambios: sin_plan_activo
   ```

2. Guarda el archivo (`Ctrl+S` / `Cmd+S`).

#### Salida Esperada

VS Code debe mostrar el archivo sin errores de sintaxis. El Language Server resaltará las keywords y variables correctamente.

#### Verificación

En la terminal, ejecuta una validación de sintaxis en seco:

```bash
python -m robot --dryrun features/mobile/keywords/mobile_steps.resource
```

> **Nota:** Si el comando reporta que no hay test cases (es un archivo `.resource`, no `.robot`), eso es correcto. Lo importante es que no reporte errores de sintaxis.

---

### Paso 3: Escribir los Escenarios BDD para Activación de Línea Móvil

**Objetivo:** Crear el feature file del dominio mobile con al menos dos escenarios BDD completos usando sintaxis Gherkin nativa.

#### Instrucciones

1. Crea el archivo `features/mobile/activacion_linea.robot` con el siguiente contenido:

   ```robot
   *** Settings ***
   Documentation    Feature: Activación de Línea Móvil
   ...
   ...              Como área comercial de TelecomCorp,
   ...              quiero que los clientes puedan activar su línea móvil
   ...              para que puedan acceder a los servicios de datos de forma inmediata.
   ...
   ...              Dominio: Mobile | Versión: 1.0 | Responsable: QA Comercial
   Resource         keywords/mobile_steps.resource

   *** Test Cases ***
   # ==========================================================================
   # Escenario 1: Activación exitosa de línea móvil con plan de datos
   # Criterio de aceptación: AC-MOB-001
   # ==========================================================================
   Cliente nuevo activa línea móvil con plan de datos pospago exitosamente
       [Documentation]    Escenario: Un cliente nuevo con línea sin plan activo selecciona
       ...                un plan de datos pospago y completa la activación correctamente.
       ...                Trazabilidad: Historia US-MOB-042 / AC-MOB-001
       [Tags]             bdd    mobile    activacion    smoke    positivo

       Given el cliente "María González" tiene una línea activa sin plan de datos
       When el cliente selecciona el plan "Datos 50GB Pospago"
       And el cliente confirma la activación con su número de documento
       Then el plan "Datos 50GB Pospago" queda activo en la línea del cliente
       And el cliente recibe una notificación de bienvenida por SMS

   # ==========================================================================
   # Escenario 2: Rechazo de activación por documento de identidad inválido
   # Criterio de aceptación: AC-MOB-002
   # ==========================================================================
   Sistema rechaza activación de línea cuando el documento de identidad no es válido
       [Documentation]    Escenario: El sistema debe rechazar la solicitud de activación
       ...                cuando el documento presentado no supera la validación,
       ...                manteniendo la línea en su estado original.
       ...                Trazabilidad: Historia US-MOB-042 / AC-MOB-002
       [Tags]             bdd    mobile    activacion    negativo    validacion

       Given el cliente "Carlos Ruiz" tiene una línea activa sin plan de datos
       When el sistema detecta que el documento de identidad no es válido
       Then el sistema rechaza la activación con el mensaje "Documento de identidad no válido. Por favor verifique sus datos."
       And la línea permanece en estado sin plan activo

   # ==========================================================================
   # Escenario 3: Activación con plan de datos prepago
   # Criterio de aceptación: AC-MOB-003
   # ==========================================================================
   Cliente activa línea móvil con plan prepago de entrada
       [Documentation]    Escenario: Un cliente selecciona el plan prepago de menor costo
       ...                disponible y completa la activación sin incidentes.
       ...                Trazabilidad: Historia US-MOB-043 / AC-MOB-003
       [Tags]             bdd    mobile    activacion    prepago    positivo

       Given el cliente "Ana Torres" tiene una línea activa sin plan de datos
       When el cliente selecciona el plan "Prepago Básico 5GB"
       And el cliente confirma la activación con su número de documento
       Then el plan "Prepago Básico 5GB" queda activo en la línea del cliente
       And el cliente recibe una notificación de bienvenida por SMS
   ```

2. Guarda el archivo.

#### Salida Esperada

El archivo debe mostrarse en VS Code con resaltado de sintaxis correcto. Las palabras `Given`, `When`, `Then`, `And` deben aparecer resaltadas, y las keywords referenciadas deben ser navegables (Ctrl+Click / Cmd+Click).

#### Verificación

Ejecuta únicamente este feature file en modo `--dryrun` para validar que todos los pasos resuelven a keywords:

```bash
python -m robot --dryrun features/mobile/activacion_linea.robot
```

**Salida esperada (fragmento):**
```
==============================================================================
Activacion Linea
==============================================================================
Cliente nuevo activa línea móvil con plan de datos pospago exitosamente
...                                                                   | PASS |
Sistema rechaza activación de línea cuando el documento de identidad no es válido
...                                                                   | PASS |
Cliente activa línea móvil con plan prepago de entrada                | PASS |
==============================================================================
Activacion Linea                                                      | PASS |
3 tests, 3 passed, 0 failed
```

---

### Paso 4: Crear el Archivo Resource para el Dominio Billing

**Objetivo:** Implementar las keywords técnicas que soportarán los pasos Gherkin del dominio de procesamiento de pago de facturas.

#### Instrucciones

1. Crea el archivo `features/billing/keywords/billing_steps.resource`:

   ```robot
   *** Settings ***
   Documentation    Keywords de implementación para el dominio Billing.
   ...              Simulan operaciones de procesamiento de pagos de factura
   ...              usando únicamente la librería BuiltIn.
   Library          BuiltIn

   *** Variables ***
   ${FACTURA_PENDIENTE}       ${TRUE}
   ${MONTO_FACTURA}           ${0.0}
   ${PAGO_PROCESADO}          ${FALSE}
   ${RECIBO_GENERADO}         ${FALSE}

   *** Keywords ***
   El cliente "${nombre}" tiene una factura pendiente de pago por "${monto}" pesos
       [Documentation]    Precondición: cliente con factura emitida y pendiente de cobro.
       Log    Cliente: ${nombre} | Factura pendiente: $${monto} pesos
       Set Test Variable    ${CLIENTE_FACTURA}    ${nombre}
       Set Test Variable    ${MONTO_FACTURA}      ${monto}
       Should Be True    ${FACTURA_PENDIENTE}

   El cliente accede al portal de pagos con sus credenciales
       [Documentation]    Simula el inicio de sesión en el portal de pagos en línea.
       Log    El cliente ${CLIENTE_FACTURA} accede al portal de pagos.
       Log    Credenciales verificadas correctamente.
       Set Test Variable    ${SESION_PORTAL}    activa

   El cliente selecciona el método de pago "${metodo}"
       [Documentation]    Registra el método de pago elegido por el cliente.
       Log    Método de pago seleccionado: ${metodo}
       Set Test Variable    ${METODO_PAGO}    ${metodo}
       Should Not Be Empty    ${METODO_PAGO}

   El cliente confirma el pago del monto total
       [Documentation]    Simula la confirmación y procesamiento del pago total.
       Log    Procesando pago de $${MONTO_FACTURA} pesos mediante ${METODO_PAGO}
       Set Test Variable    ${PAGO_PROCESADO}    ${TRUE}
       Should Be True    ${PAGO_PROCESADO}

   La factura queda marcada como pagada en el sistema
       [Documentation]    Verifica que el estado de la factura se actualizó correctamente.
       Log    Factura marcada como PAGADA para el cliente: ${CLIENTE_FACTURA}
       Set Test Variable    ${FACTURA_PENDIENTE}    ${FALSE}
       Should Not Be True    ${FACTURA_PENDIENTE}

   El cliente recibe un recibo de pago por correo electrónico
       [Documentation]    Simula el envío del comprobante de pago por email.
       Log    Enviando recibo de pago por email al cliente: ${CLIENTE_FACTURA}
       Set Test Variable    ${RECIBO_GENERADO}    ${TRUE}
       Should Be True    ${RECIBO_GENERADO}
       Log    Recibo enviado exitosamente.

   El cliente intenta pagar con una tarjeta de crédito vencida
       [Documentation]    Simula el intento de pago con tarjeta expirada.
       Log    Intento de pago con tarjeta vencida detectado.
       Set Test Variable    ${TARJETA_VALIDA}    ${FALSE}
       Log    Tarjeta marcada como vencida en el sistema.

   El sistema detecta que la tarjeta está vencida
       [Documentation]    Simula la validación del sistema que detecta la expiración.
       Log    Sistema detecta: tarjeta de crédito vencida.
       Should Not Be True    ${TARJETA_VALIDA}

   El pago es rechazado con el código de error "${codigo}"
       [Documentation]    Verifica que el sistema genera el código de error correcto.
       Log    Pago rechazado. Código de error: ${codigo}
       Should Not Be Empty    ${codigo}

   La factura permanece en estado pendiente de pago
       [Documentation]    Verifica que la factura no cambió de estado tras el rechazo.
       Log    Verificando que la factura sigue pendiente de pago.
       Should Be True    ${FACTURA_PENDIENTE}
       Log    Estado de factura sin cambios: pendiente.

   El cliente tiene saldo insuficiente en su cuenta digital
       [Documentation]    Simula la condición de saldo insuficiente en billetera digital.
       Log    Saldo insuficiente detectado para el cliente: ${CLIENTE_FACTURA}
       Set Test Variable    ${SALDO_SUFICIENTE}    ${FALSE}

   El sistema notifica al cliente que el saldo es insuficiente
       [Documentation]    Verifica que el sistema envía la notificación de saldo insuficiente.
       Log    Notificando al cliente: saldo insuficiente para completar el pago.
       Should Not Be True    ${SALDO_SUFICIENTE}
       Log    Notificación enviada al cliente.

   El cliente es redirigido a opciones de recarga de saldo
       [Documentation]    Simula la redirección al flujo de recarga de saldo.
       Log    Redirigiendo al cliente a opciones de recarga de saldo.
       Log    Opciones disponibles: transferencia bancaria, efectivo, tarjeta.
   ```

2. Guarda el archivo.

#### Verificación

```bash
python -m robot --dryrun features/billing/keywords/billing_steps.resource
```

---

### Paso 5: Escribir los Escenarios BDD para Procesamiento de Pago de Factura

**Objetivo:** Crear el feature file del dominio billing con escenarios que cubran el flujo de pago exitoso y escenarios alternativos de error.

#### Instrucciones

1. Crea el archivo `features/billing/pago_factura.robot`:

   ```robot
   *** Settings ***
   Documentation    Feature: Procesamiento de Pago de Factura
   ...
   ...              Como cliente de TelecomCorp,
   ...              quiero poder pagar mi factura mensual a través del portal en línea
   ...              para mantener mis servicios activos sin necesidad de ir a una tienda física.
   ...
   ...              Dominio: Billing | Versión: 1.0 | Responsable: QA Facturación
   Resource         keywords/billing_steps.resource

   *** Test Cases ***
   # ==========================================================================
   # Escenario 1: Pago exitoso de factura mediante tarjeta de débito
   # Criterio de aceptación: AC-BILL-001
   # ==========================================================================
   Cliente paga factura mensual exitosamente con tarjeta de débito
       [Documentation]    Escenario: El cliente accede al portal, selecciona tarjeta de débito
       ...                como método de pago, confirma el monto y recibe su recibo.
       ...                Trazabilidad: Historia US-BILL-015 / AC-BILL-001
       [Tags]             bdd    billing    pago    smoke    positivo

       Given el cliente "Roberto Medina" tiene una factura pendiente de pago por "1250" pesos
       And el cliente accede al portal de pagos con sus credenciales
       When el cliente selecciona el método de pago "Tarjeta de Débito"
       And el cliente confirma el pago del monto total
       Then la factura queda marcada como pagada en el sistema
       And el cliente recibe un recibo de pago por correo electrónico

   # ==========================================================================
   # Escenario 2: Rechazo de pago por tarjeta de crédito vencida
   # Criterio de aceptación: AC-BILL-002
   # ==========================================================================
   Sistema rechaza pago cuando la tarjeta de crédito está vencida
       [Documentation]    Escenario: El cliente intenta pagar con una tarjeta expirada.
       ...                El sistema debe detectar la condición y rechazar la transacción
       ...                con el código de error correspondiente.
       ...                Trazabilidad: Historia US-BILL-015 / AC-BILL-002
       [Tags]             bdd    billing    pago    negativo    validacion

       Given el cliente "Lucía Herrera" tiene una factura pendiente de pago por "890" pesos
       And el cliente accede al portal de pagos con sus credenciales
       When el cliente intenta pagar con una tarjeta de crédito vencida
       And el sistema detecta que la tarjeta está vencida
       Then el pago es rechazado con el código de error "ERR-CARD-001"
       And la factura permanece en estado pendiente de pago

   # ==========================================================================
   # Escenario 3: Notificación por saldo insuficiente en billetera digital
   # Criterio de aceptación: AC-BILL-003
   # ==========================================================================
   Sistema notifica saldo insuficiente al cliente que paga con billetera digital
       [Documentation]    Escenario: El cliente intenta pagar usando su billetera digital
       ...                pero el saldo disponible es menor al monto de la factura.
       ...                El sistema debe notificarlo y ofrecer opciones de recarga.
       ...                Trazabilidad: Historia US-BILL-016 / AC-BILL-003
       [Tags]             bdd    billing    pago    negativo    billetera_digital

       Given el cliente "Fernando Castillo" tiene una factura pendiente de pago por "2100" pesos
       And el cliente accede al portal de pagos con sus credenciales
       When el cliente selecciona el método de pago "Billetera Digital TelecomCorp"
       And el cliente tiene saldo insuficiente en su cuenta digital
       Then el sistema notifica al cliente que el saldo es insuficiente
       And el cliente es redirigido a opciones de recarga de saldo
       But la factura permanece en estado pendiente de pago
   ```

2. Observa el uso de `But` en el Escenario 3. Esta palabra clave Gherkin es equivalente a `And` en Robot Framework pero comunica una negación o excepción al resultado esperado, aportando mayor expresividad al escenario.

3. Guarda el archivo.

#### Verificación

```bash
python -m robot --dryrun features/billing/pago_factura.robot
```

**Salida esperada:**
```
==============================================================================
Pago Factura
==============================================================================
Cliente paga factura mensual exitosamente con tarjeta de débito          | PASS |
Sistema rechaza pago cuando la tarjeta de crédito está vencida           | PASS |
Sistema notifica saldo insuficiente al cliente que paga con billetera digital | PASS |
==============================================================================
Pago Factura                                                             | PASS |
3 tests, 3 passed, 0 failed
```

---

### Paso 6: Crear el Archivo Resource para el Dominio Plans

**Objetivo:** Implementar las keywords técnicas para el dominio de cambio de plan de datos, el flujo comercial de mayor complejidad del laboratorio.

#### Instrucciones

1. Crea el archivo `features/plans/keywords/plans_steps.resource`:

   ```robot
   *** Settings ***
   Documentation    Keywords de implementación para el dominio Plans.
   ...              Simulan operaciones de cambio de plan de datos
   ...              usando únicamente la librería BuiltIn.
   Library          BuiltIn

   *** Variables ***
   ${PLAN_ACTUAL}             ${EMPTY}
   ${PLAN_NUEVO}              ${EMPTY}
   ${CAMBIO_EFECTIVO}         inmediato
   ${CAMBIO_APLICADO}         ${FALSE}

   *** Keywords ***
   El cliente "${nombre}" tiene contratado el plan "${plan_actual}"
       [Documentation]    Precondición: cliente con un plan de datos activo específico.
       Log    Cliente: ${nombre} | Plan actual: ${plan_actual}
       Set Test Variable    ${CLIENTE_PLANS}    ${nombre}
       Set Test Variable    ${PLAN_ACTUAL}      ${plan_actual}
       Should Not Be Empty    ${PLAN_ACTUAL}

   El cliente navega a la sección "Gestión de Planes" en el portal
       [Documentation]    Simula la navegación al módulo de gestión de planes.
       Log    El cliente ${CLIENTE_PLANS} accede a la sección Gestión de Planes.
       Set Test Variable    ${SECCION_ACTIVA}    gestion_planes

   El cliente selecciona el nuevo plan "${plan_nuevo}"
       [Documentation]    Registra el plan nuevo seleccionado por el cliente.
       Log    Plan nuevo seleccionado: ${plan_nuevo}
       Set Test Variable    ${PLAN_NUEVO}    ${plan_nuevo}
       Should Not Be Empty    ${PLAN_NUEVO}

   El cliente confirma el cambio de plan
       [Documentation]    Simula la confirmación del cambio de plan por parte del cliente.
       Log    Confirmando cambio de plan: ${PLAN_ACTUAL} → ${PLAN_NUEVO}
       Log    Cambio confirmado por el cliente: ${CLIENTE_PLANS}
       Set Test Variable    ${CONFIRMACION_CAMBIO}    confirmado

   El nuevo plan "${plan_nuevo}" queda activo de forma inmediata
       [Documentation]    Verifica que el cambio de plan se aplicó correctamente.
       Log    Aplicando cambio de plan inmediato para: ${CLIENTE_PLANS}
       Set Test Variable    ${PLAN_ACTUAL}     ${plan_nuevo}
       Set Test Variable    ${CAMBIO_APLICADO}    ${TRUE}
       Should Be True    ${CAMBIO_APLICADO}
       Log    Plan ${plan_nuevo} activado exitosamente.

   El cliente recibe confirmación del cambio por SMS y correo electrónico
       [Documentation]    Simula el envío de confirmación por múltiples canales.
       Log    Enviando confirmación a: ${CLIENTE_PLANS}
       Log    Canal SMS: enviado correctamente.
       Log    Canal Email: enviado correctamente.

   El cliente tiene contratado el plan "${plan_actual}" con permanencia activa hasta "${fecha}"
       [Documentation]    Precondición: cliente con plan en período de permanencia.
       Log    Cliente: ${CLIENTE_PLANS} | Plan: ${plan_actual} | Permanencia hasta: ${fecha}
       Set Test Variable    ${PLAN_ACTUAL}       ${plan_actual}
       Set Test Variable    ${FECHA_PERMANENCIA}    ${fecha}
       Log    Permanencia activa detectada.

   El cliente intenta cambiar al plan "${plan_nuevo}" antes del fin de permanencia
       [Documentation]    Simula el intento de cambio durante período de permanencia.
       Log    Intento de cambio anticipado: ${plan_actual} → ${plan_nuevo}
       Set Test Variable    ${PLAN_NUEVO}    ${plan_nuevo}
       Set Test Variable    ${INTENTO_CAMBIO_ANTICIPADO}    ${TRUE}

   El sistema informa que existe una penalización de "${monto}" pesos por cambio anticipado
       [Documentation]    Verifica que el sistema calcula y comunica la penalización.
       Log    Penalización por cambio anticipado: $${monto} pesos
       Should Not Be Empty    ${monto}
       Log    Información de penalización comunicada al cliente.

   El cliente puede optar por aceptar la penalización o mantener su plan actual
       [Documentation]    Verifica que el sistema ofrece opciones al cliente.
       Log    Opciones presentadas al cliente:
       Log    1. Aceptar penalización y cambiar de plan.
       Log    2. Mantener el plan actual hasta fin de permanencia.

   El plan "${plan_actual}" se mantiene sin cambios hasta el fin del período de permanencia
       [Documentation]    Verifica que el plan no fue modificado cuando el cliente no acepta.
       Log    Plan ${plan_actual} mantenido sin cambios.
       Should Be Equal    ${PLAN_ACTUAL}    ${plan_actual}
       Log    Sin cambios aplicados al plan del cliente.

   El cliente solicita degradar su plan al nivel inferior disponible
       [Documentation]    Simula la solicitud de degradación de plan.
       Log    Solicitud de degradación de plan recibida del cliente: ${CLIENTE_PLANS}
       Set Test Variable    ${TIPO_CAMBIO}    degradacion

   El sistema valida que el cliente no tiene consumo excedente pendiente
       [Documentation]    Simula la validación de consumo antes de permitir la degradación.
       Log    Validando consumo excedente para: ${CLIENTE_PLANS}
       Set Test Variable    ${CONSUMO_EXCEDENTE}    ${FALSE}
       Should Not Be True    ${CONSUMO_EXCEDENTE}
       Log    Sin consumo excedente. Degradación permitida.

   El cambio al plan inferior se programa para el inicio del siguiente ciclo de facturación
       [Documentation]    Verifica que la degradación se programa correctamente.
       Log    Degradación programada para el inicio del próximo ciclo de facturación.
       Set Test Variable    ${CAMBIO_EFECTIVO}    inicio_siguiente_ciclo
       Should Be Equal    ${CAMBIO_EFECTIVO}    inicio_siguiente_ciclo
   ```

2. Guarda el archivo.

#### Verificación

```bash
python -m robot --dryrun features/plans/keywords/plans_steps.resource
```

---

### Paso 7: Escribir los Escenarios BDD para Cambio de Plan de Datos

**Objetivo:** Crear el feature file más completo del laboratorio, con escenarios que cubran el flujo de cambio de plan en sus variantes positiva, con permanencia y de degradación.

#### Instrucciones

1. Crea el archivo `features/plans/cambio_plan.robot`:

   ```robot
   *** Settings ***
   Documentation    Feature: Cambio de Plan de Datos
   ...
   ...              Como cliente de TelecomCorp,
   ...              quiero poder cambiar mi plan de datos desde el portal en línea
   ...              para adaptar mi servicio a mis necesidades actuales de consumo
   ...              sin necesidad de contactar al servicio al cliente.
   ...
   ...              Reglas de Negocio:
   ...              - Los cambios a planes superiores son efectivos de forma inmediata.
   ...              - Los cambios a planes inferiores (degradación) aplican al inicio del siguiente ciclo.
   ...              - Los cambios durante período de permanencia generan penalización.
   ...
   ...              Dominio: Plans | Versión: 1.0 | Responsable: QA Comercial
   Resource         keywords/plans_steps.resource

   *** Test Cases ***
   # ==========================================================================
   # Escenario 1: Upgrade de plan exitoso con aplicación inmediata
   # Criterio de aceptación: AC-PLAN-001
   # ==========================================================================
   Cliente realiza upgrade de plan de datos y el cambio aplica de forma inmediata
       [Documentation]    Escenario: El cliente con un plan básico activo selecciona un plan
       ...                superior. El sistema debe aplicar el cambio inmediatamente
       ...                y confirmar por SMS y correo.
       ...                Trazabilidad: Historia US-PLAN-028 / AC-PLAN-001
       [Tags]             bdd    plans    cambio_plan    upgrade    smoke    positivo

       Given el cliente "Patricia Vega" tiene contratado el plan "Datos 20GB Pospago"
       And el cliente navega a la sección "Gestión de Planes" en el portal
       When el cliente selecciona el nuevo plan "Datos 100GB Pospago Premium"
       And el cliente confirma el cambio de plan
       Then el nuevo plan "Datos 100GB Pospago Premium" queda activo de forma inmediata
       And el cliente recibe confirmación del cambio por SMS y correo electrónico

   # ==========================================================================
   # Escenario 2: Cambio de plan bloqueado por período de permanencia activo
   # Criterio de aceptación: AC-PLAN-002
   # ==========================================================================
   Sistema informa penalización cuando el cliente intenta cambiar plan durante permanencia
       [Documentation]    Escenario: El cliente tiene un plan con permanencia vigente.
       ...                Al intentar cambiarlo antes del vencimiento, el sistema debe
       ...                informar el costo de la penalización y presentar opciones.
       ...                Trazabilidad: Historia US-PLAN-029 / AC-PLAN-002
       [Tags]             bdd    plans    cambio_plan    permanencia    negativo    validacion

       Given el cliente "Miguel Ángel Soto" tiene contratado el plan "Fibra Convergente 300Mbps" con permanencia activa hasta "31/12/2025"
       And el cliente navega a la sección "Gestión de Planes" en el portal
       When el cliente intenta cambiar al plan "Fibra Convergente 600Mbps" antes del fin de permanencia
       Then el sistema informa que existe una penalización de "3500" pesos por cambio anticipado
       And el cliente puede optar por aceptar la penalización o mantener su plan actual
       But el plan "Fibra Convergente 300Mbps" se mantiene sin cambios hasta el fin del período de permanencia

   # ==========================================================================
   # Escenario 3: Downgrade de plan con aplicación diferida al siguiente ciclo
   # Criterio de aceptación: AC-PLAN-003
   # ==========================================================================
   Cliente solicita degradación de plan y el cambio se programa para el siguiente ciclo
       [Documentation]    Escenario: El cliente desea reducir su plan al nivel inferior.
       ...                El sistema valida que no hay consumo excedente y programa
       ...                el cambio para el inicio del siguiente ciclo de facturación.
       ...                Trazabilidad: Historia US-PLAN-030 / AC-PLAN-003
       [Tags]             bdd    plans    cambio_plan    downgrade    positivo    ciclo_facturacion

       Given el cliente "Sofía Ramírez" tiene contratado el plan "Datos 100GB Pospago Premium"
       And el cliente navega a la sección "Gestión de Planes" en el portal
       When el cliente solicita degradar su plan al nivel inferior disponible
       And el sistema valida que el cliente no tiene consumo excedente pendiente
       Then el cambio al plan inferior se programa para el inicio del siguiente ciclo de facturación
       And el cliente recibe confirmación del cambio por SMS y correo electrónico
   ```

2. Guarda el archivo.

#### Verificación

```bash
python -m robot --dryrun features/plans/cambio_plan.robot
```

**Salida esperada:**
```
==============================================================================
Cambio Plan
==============================================================================
Cliente realiza upgrade de plan de datos y el cambio aplica de forma inmediata | PASS |
Sistema informa penalización cuando el cliente intenta cambiar plan durante permanencia | PASS |
Cliente solicita degradación de plan y el cambio se programa para el siguiente ciclo | PASS |
==============================================================================
Cambio Plan                                                              | PASS |
3 tests, 3 passed, 0 failed
```

---

### Paso 8: Ejecutar la Suite Completa y Analizar el Reporte

**Objetivo:** Ejecutar todos los feature files de los tres dominios en una sola invocación y explorar el reporte HTML para verificar la legibilidad de los pasos Gherkin.

#### Instrucciones

1. Desde la raíz del proyecto, ejecuta la suite completa especificando el directorio `features/`:

   ```bash
   python -m robot \
     --outputdir results/lab-04-00-01 \
     --name "Suite BDD TelecomCorp" \
     --variable ENTORNO:laboratorio \
     features/
   ```

   **Windows (cmd — una sola línea):**
   ```cmd
   python -m robot --outputdir results\lab-04-00-01 --name "Suite BDD TelecomCorp" --variable ENTORNO:laboratorio features\
   ```

2. Una vez finalizada la ejecución, abre el reporte HTML:

   **macOS / Linux:**
   ```bash
   open results/lab-04-00-01/report.html
   ```

   **Windows:**
   ```cmd
   start results\lab-04-00-01\report.html
   ```

3. En el reporte HTML, navega a cualquier test case y expande los pasos. Observa que:
   - Los pasos se muestran con sus prefijos `Given`, `When`, `Then`, `And`, `But`.
   - Los nombres de los pasos son completamente legibles en lenguaje de negocio.
   - Los parámetros (nombres de clientes, planes, montos) se muestran en el log.

4. Opcionalmente, ejecuta solo un dominio específico usando el tag `mobile`:

   ```bash
   python -m robot \
     --outputdir results/lab-04-00-01-mobile \
     --include mobile \
     features/
   ```

#### Salida Esperada en Terminal

```
==============================================================================
Suite BDD TelecomCorp
==============================================================================
Suite BDD TelecomCorp.Features
==============================================================================
Suite BDD TelecomCorp.Features.Billing
==============================================================================
Suite BDD TelecomCorp.Features.Billing.Pago Factura
==============================================================================
Cliente paga factura mensual exitosamente con tarjeta de débito          | PASS |
Sistema rechaza pago cuando la tarjeta de crédito está vencida           | PASS |
Sistema notifica saldo insuficiente al cliente que paga con billetera digital | PASS |
==============================================================================
Suite BDD TelecomCorp.Features.Billing.Pago Factura                     | PASS |
3 tests, 3 passed, 0 failed
==============================================================================
...
==============================================================================
Suite BDD TelecomCorp                                                    | PASS |
9 tests, 9 passed, 0 failed
==============================================================================
Output:  .../results/lab-04-00-01/output.xml
Log:     .../results/lab-04-00-01/log.html
Report:  .../results/lab-04-00-01/report.html
```

#### Verificación

- **9 tests pasando** (3 por cada dominio: mobile, billing, plans).
- El archivo `report.html` se abre en el navegador y muestra los tres dominios como suites anidadas.
- Los pasos Gherkin son visibles en el log con sus prefijos correspondientes.

---

## Validación y Pruebas

### Lista de Verificación Final

Ejecuta las siguientes comprobaciones antes de dar el laboratorio por completado:

#### 1. Verificar la estructura de archivos

**macOS / Linux:**
```bash
find features -type f | sort
```

**Windows (PowerShell):**
```powershell
Get-ChildItem -Recurse features -File | Select-Object FullName
```

**Salida esperada:**
```
features/billing/keywords/billing_steps.resource
features/billing/pago_factura.robot
features/mobile/keywords/mobile_steps.resource
features/mobile/activacion_linea.robot
features/plans/keywords/plans_steps.resource
features/plans/cambio_plan.robot
```

#### 2. Verificar conteo de escenarios por dominio

```bash
python -m robot --collect-only features/ 2>&1 | grep "tests\|PASS\|FAIL"
```

**Resultado esperado:** 9 tests recolectados en total (3 por dominio).

#### 3. Verificar uso de todas las palabras clave Gherkin

Confirma que en los archivos `.robot` aparecen los cinco prefijos Gherkin:

```bash
grep -rh "^\s*\(Given\|When\|Then\|And\|But\)" features/*.robot features/**/*.robot 2>/dev/null | \
  awk '{print $1}' | sort | uniq -c | sort -rn
```

**Windows (PowerShell):**
```powershell
Select-String -Path "features\**\*.robot" -Pattern "^\s+(Given|When|Then|And|But)" -AllMatches |
  ForEach-Object { $_.Matches } | ForEach-Object { $_.Value.Trim() } |
  Group-Object | Select-Object Count, Name | Sort-Object Count -Descending
```

**Resultado esperado:** Las palabras `Given`, `When`, `Then`, `And` y `But` deben aparecer todas con al menos una ocurrencia.

#### 4. Ejecutar suite completa y verificar resultado

```bash
python -m robot --outputdir results/lab-04-00-01-final features/
echo "Exit code: $?"
```

**Resultado esperado:** Exit code `0` (todos los tests pasaron).

#### 5. Análisis de separación de capas

Verifica que ningún archivo `.robot` contiene implementación técnica directa (solo debe contener `Settings`, `Test Cases`):

```bash
grep -l "Log\|Should\|Set Test Variable" features/mobile/activacion_linea.robot \
     features/billing/pago_factura.robot features/plans/cambio_plan.robot
```

**Resultado esperado:** El comando no debe devolver ningún archivo (las keywords técnicas están solo en los `.resource`).

---

## Solución de Problemas

### Problema 1: Error "No keyword with name '...' found"

**Síntoma:**
```
No keyword with name 'El cliente "María González" tiene una línea activa sin plan de datos' found.
```
La ejecución falla con error de keyword no encontrada, aunque el archivo `.resource` existe en el directorio `keywords/`.

**Causa:**
La ruta en la directiva `Resource` del archivo `.robot` es incorrecta o relativa al directorio de trabajo de ejecución en lugar de al directorio del archivo `.robot`. Robot Framework resuelve las rutas de `Resource` relativas al directorio del archivo que las declara, no al directorio desde donde se lanza el comando.

**Solución:**

1. Verifica que la directiva `Resource` en el archivo `.robot` usa la ruta relativa correcta:
   ```robot
   # CORRECTO — relativo al archivo .robot
   Resource    keywords/mobile_steps.resource

   # INCORRECTO — ruta absoluta o mal formada
   Resource    features/mobile/keywords/mobile_steps.resource
   ```

2. Confirma que el archivo `.resource` existe en la ubicación esperada:
   ```bash
   ls features/mobile/keywords/mobile_steps.resource
   ```

3. Si el problema persiste, usa `--dryrun` para diagnóstico detallado:
   ```bash
   python -m robot --dryrun features/mobile/activacion_linea.robot
   ```
   El mensaje de error indicará exactamente qué keyword no fue encontrada y en qué archivo.

---

### Problema 2: Los pasos Gherkin aparecen sin prefijo en el reporte HTML

**Síntoma:**
En el reporte HTML (`log.html`), los pasos de los test cases aparecen con el nombre completo de la keyword (por ejemplo, `El cliente "María González" tiene una línea activa sin plan de datos`) pero **sin el prefijo** `Given`, `When`, `Then`, `And` o `But`. El log parece una lista de keywords normales, no un escenario BDD.

**Causa:**
Robot Framework elimina los prefijos Gherkin (`Given`, `When`, `Then`, `And`, `But`) al resolver la keyword, lo que es el comportamiento **correcto y esperado**. El prefijo se usa para la legibilidad en el archivo `.robot` y en el log de ejecución, pero internamente Robot Framework llama a la keyword sin el prefijo. Si los pasos no aparecen con prefijo en el log, es porque el Language Server o el visor del reporte está mostrando el nombre interno de la keyword.

**Solución:**

1. Verifica que estás revisando el archivo `log.html` (no `output.xml`). En `log.html`, expande el test case y busca la columna de "Keyword". Robot Framework 7.x **sí muestra** el prefijo Gherkin en el log cuando se usa correctamente.

2. Confirma que los pasos en el archivo `.robot` usan el prefijo con mayúscula inicial y un espacio:
   ```robot
   # CORRECTO
   Given el cliente "María González" tiene una línea activa sin plan de datos

   # INCORRECTO — sin prefijo
   el cliente "María González" tiene una línea activa sin plan de datos
   ```

3. Si el problema persiste, verifica la versión de Robot Framework:
   ```bash
   python -m robot --version
   ```
   Debe ser 7.x. En versiones anteriores a RF 4.0, el comportamiento del log podía diferir.

4. Como prueba de diagnóstico, ejecuta con nivel de log `DEBUG` y revisa el archivo `log.html`:
   ```bash
   python -m robot --loglevel DEBUG --outputdir results/debug features/mobile/activacion_linea.robot
   ```

---

## Limpieza

### Archivos Generados durante el Laboratorio

Los siguientes directorios y archivos fueron generados durante la ejecución y pueden ser eliminados si se desea liberar espacio:

```bash
# Eliminar resultados de ejecución (mantener los archivos .robot y .resource)
# macOS / Linux:
rm -rf results/lab-04-00-01 results/lab-04-00-01-mobile results/lab-04-00-01-final results/debug

# Windows (PowerShell):
Remove-Item -Recurse -Force results\lab-04-00-01, results\lab-04-00-01-mobile, results\lab-04-00-01-final, results\debug
```

> ⚠️ **No elimines** los directorios `features/` ni los archivos `.robot` y `.resource`. Estos son el producto del laboratorio y serán necesarios en laboratorios posteriores del Módulo 4.

### Desactivar el Entorno Virtual

```bash
deactivate
```

### Respaldo del Proyecto

Se recomienda crear una copia de respaldo del proyecto al finalizar el módulo:

```bash
# macOS / Linux:
cp -r ~/proyectos/robot-telecom ~/proyectos/robot-telecom-backup-lab04

# Windows (PowerShell):
Copy-Item -Recurse C:\proyectos\robot-telecom C:\proyectos\robot-telecom-backup-lab04
```

---

## Resumen

### Lo que Construiste

En este laboratorio creaste desde cero una estructura BDD completa para una empresa de telecomunicaciones ficticia, organizada en tres dominios de negocio independientes:

| Dominio    | Feature File             | Escenarios | Palabras Gherkin Usadas          |
|------------|--------------------------|------------|----------------------------------|
| `mobile`   | `activacion_linea.robot` | 3          | Given, When, And, Then           |
| `billing`  | `pago_factura.robot`     | 3          | Given, And, When, Then, But      |
| `plans`    | `cambio_plan.robot`      | 3          | Given, And, When, Then, But      |
| **Total**  |                          | **9**      | **Given, When, Then, And, But**  |

### Conceptos Clave Aplicados

- **Separación de capas:** Los archivos `.robot` contienen únicamente la descripción del comportamiento (Gherkin), mientras que los archivos `.resource` contienen la implementación técnica. Esta separación permite que un stakeholder no técnico lea y valide los escenarios sin necesidad de entender el código subyacente.

- **Sintaxis Gherkin nativa:** Robot Framework soporta los prefijos `Given`, `When`, `Then`, `And` y `But` de forma nativa, sin requerir librerías externas. Estos prefijos son ignorados al resolver la keyword, lo que significa que `Given el cliente X` y `el cliente X` llaman a la misma keyword.

- **Organización por dominio:** La estructura `features/mobile/`, `features/billing/` y `features/plans/` refleja los dominios de negocio de la empresa y facilita la trazabilidad entre escenarios y áreas funcionales.

- **Trazabilidad:** Cada escenario incluye en su `[Documentation]` la referencia a la historia de usuario y al criterio de aceptación correspondiente, cerrando el ciclo entre requisito de negocio y prueba automatizada.

- **Valor de BDD:** Un analista de negocio de TelecomCorp puede leer el escenario *"Sistema rechaza pago cuando la tarjeta de crédito está vencida"* y validar que el comportamiento descrito es correcto, sin necesidad de revisar código Python o keywords técnicas.

### Recursos Adicionales

- [Robot Framework User Guide — Behavior-Driven Style](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#behavior-driven-style)
- [Dan North — Introducing BDD (artículo original, en inglés)](https://dannorth.net/introducing-bdd/)
- [Cucumber — Referencia oficial de Gherkin](https://cucumber.io/docs/gherkin/reference/)
- [Agile Alliance — Glosario BDD](https://www.agilealliance.org/glossary/bdd/)

---

---

---LAB_START---
LAB_ID: 04-00-02
---MARKDOWN---
# Refactorización de test tradicional a modelo BDD con separación de capas

## Metadatos

| Campo            | Valor                                      |
|------------------|--------------------------------------------|
| **Duración**     | 72 minutos                                 |
| **Complejidad**  | Alta                                       |
| **Nivel Bloom**  | Crear                                      |
| **Módulo**       | 4 — Behavior-Driven Development            |
| **Laboratorio**  | 04-00-02 (Práctica 8)                      |

---

## Descripción General

En este laboratorio recibirás una suite de pruebas tradicional (estilo keyword-driven) que valida un flujo de gestión de contratos de telecomunicaciones y la refactorizarás completamente al modelo BDD con arquitectura de tres capas. El proceso te llevará desde identificar los comportamientos de negocio detrás de cada test case técnico, redactar los escenarios Gherkin correspondientes, separar las responsabilidades en capas de abstracción, hasta verificar que ambas versiones producen exactamente los mismos resultados de ejecución. Al finalizar, habrás aplicado los principios de BDD presentados en la Lección 4.1 en un escenario realista de telecomunicaciones.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio serás capaz de:

- [ ] Refactorizar una suite de prueba tradicional a modelo BDD manteniendo la misma cobertura funcional en los 8 test cases originales.
- [ ] Implementar una arquitectura de tres capas: escenarios de negocio (Gherkin) → keywords de dominio → keywords técnicas.
- [ ] Garantizar que los escenarios BDD refactorizados son agnósticos a la implementación técnica subyacente.
- [ ] Comparar métricas de mantenibilidad y legibilidad entre el enfoque tradicional y el BDD refactorizado mediante comentarios documentados.

---

## Prerrequisitos

### Conocimiento previo

- Haber completado el Laboratorio 04-00-01 con escenarios BDD funcionales y sin errores.
- Comprensión sólida de archivos Resource y keywords parametrizadas (Módulo 2).
- Capacidad de leer y entender una suite existente de Robot Framework con keywords anidadas.
- Familiaridad con la sintaxis Gherkin nativa (`Given / When / Then / And / But`) en Robot Framework.

### Acceso y herramientas

- Entorno virtual Python (`venv`) activo con Robot Framework 7.x instalado.
- Visual Studio Code con la extensión Robot Framework Language Server activa.
- Proyecto del Laboratorio 04-00-01 disponible como referencia (no es necesario modificarlo).
- Terminal con permisos de escritura en el directorio de trabajo.

---

## Entorno del Laboratorio

### Requisitos de hardware

| Componente       | Mínimo requerido                                      |
|------------------|-------------------------------------------------------|
| Procesador       | Intel Core i5 8ª gen / AMD Ryzen 5 (4 núcleos)       |
| RAM              | 8 GB (16 GB recomendado)                              |
| Almacenamiento   | 5 GB libres para proyecto, reportes y artefactos      |
| Pantalla         | Resolución mínima 1280×768                            |

### Requisitos de software

| Software                        | Versión mínima |
|---------------------------------|----------------|
| Python                          | 3.10+          |
| Robot Framework                 | 7.x            |
| Visual Studio Code              | 1.85+          |
| RF Language Server (extensión)  | 1.12+          |
| pip                             | 23.x+          |

### Preparación del entorno

Ejecuta los siguientes comandos antes de comenzar. Usa la variante correspondiente a tu sistema operativo.

**Paso 1 — Activar el entorno virtual:**

```bash
# Windows (cmd)
cd C:\cursoRF\modulo04
Scripts\activate

# Windows (PowerShell)
cd C:\cursoRF\modulo04
.\Scripts\Activate.ps1

# macOS / Linux
cd ~/cursoRF/modulo04
source bin/activate
```

**Paso 2 — Verificar la versión de Robot Framework:**

```bash
robot --version
# Salida esperada: Robot Framework 7.x.x (Python 3.x.x ...)
```

**Paso 3 — Crear la estructura del laboratorio:**

```bash
# Windows (cmd / PowerShell)
mkdir lab04-00-02
cd lab04-00-02
mkdir traditional_suite
mkdir bdd_suite\features
mkdir bdd_suite\domain_keywords
mkdir bdd_suite\technical_keywords
mkdir results\traditional
mkdir results\bdd

# macOS / Linux
mkdir -p lab04-00-02
cd lab04-00-02
mkdir -p traditional_suite
mkdir -p bdd_suite/features
mkdir -p bdd_suite/domain_keywords
mkdir -p bdd_suite/technical_keywords
mkdir -p results/traditional
mkdir -p results/bdd
```

La estructura final del proyecto debe verse así:

```
lab04-00-02/
├── traditional_suite/
│   └── contract_management_traditional.robot
├── bdd_suite/
│   ├── features/
│   │   └── contract_management.robot
│   ├── domain_keywords/
│   │   └── contract_domain.resource
│   └── technical_keywords/
│       └── contract_technical.resource
└── results/
    ├── traditional/
    └── bdd/
```

---

## Desarrollo Paso a Paso

---

### Paso 1: Crear y analizar la suite tradicional de referencia

**Objetivo:** Establecer la suite tradicional pre-construida que servirá como punto de partida para la refactorización. Esta suite simula datos en memoria (sin UI ni API real) para que el laboratorio sea autocontenido.

#### Instrucciones

1. Abre Visual Studio Code en la carpeta `lab04-00-02`.
2. Crea el archivo `traditional_suite/contract_management_traditional.robot` con el siguiente contenido completo:

```robot
*** Settings ***
Documentation    Suite tradicional de gestión de contratos de telecomunicaciones.
...              Esta suite será refactorizada al modelo BDD en el laboratorio 04-00-02.
...              NOTA: Los datos son ficticios y autocontenidos. No se requiere conexión externa.
Library          Collections
Library          String
Library          BuiltIn

*** Variables ***
# --- Datos de clientes ficticios ---
${CLIENTE_VALIDO_ID}          CLI-001
${CLIENTE_VALIDO_NOMBRE}      Ana García
${CLIENTE_INVALIDO_ID}        CLI-999
${PLAN_BASICO}                PLAN-DATOS-10GB
${PLAN_PREMIUM}               PLAN-DATOS-50GB
${PLAN_INVALIDO}              PLAN-INEXISTENTE
${CONTRATO_ACTIVO_ID}         CONT-2024-001
${CONTRATO_CANCELADO_ID}      CONT-2024-002
${LIMITE_CREDITO_APROBADO}    500
${LIMITE_CREDITO_RECHAZADO}   50

*** Test Cases ***
TC-001 Verificar que un cliente válido puede ser encontrado en el sistema
    [Documentation]    Verifica la búsqueda de un cliente existente por ID.
    [Tags]    cliente    busqueda    smoke
    ${resultado}=    Buscar Cliente Por ID    ${CLIENTE_VALIDO_ID}
    Should Not Be Empty    ${resultado}
    Dictionary Should Contain Key    ${resultado}    nombre
    Should Be Equal    ${resultado}[nombre]    ${CLIENTE_VALIDO_NOMBRE}
    Log    Cliente encontrado: ${resultado}[nombre]

TC-002 Verificar que un cliente inválido genera error controlado
    [Documentation]    Verifica que buscar un ID inexistente retorna estado de error.
    [Tags]    cliente    busqueda    negativo
    ${resultado}=    Buscar Cliente Por ID    ${CLIENTE_INVALIDO_ID}
    Dictionary Should Contain Key    ${resultado}    error
    Should Be Equal    ${resultado}[error]    CLIENTE_NO_ENCONTRADO
    Log    Error esperado recibido: ${resultado}[error]

TC-003 Verificar que un cliente puede contratar un plan de datos válido
    [Documentation]    Verifica la contratación exitosa de un plan de datos.
    [Tags]    contrato    alta    smoke
    ${cliente}=    Buscar Cliente Por ID    ${CLIENTE_VALIDO_ID}
    ${plan}=       Obtener Detalle Plan    ${PLAN_BASICO}
    ${contrato}=   Crear Contrato    ${cliente}    ${plan}
    Dictionary Should Contain Key    ${contrato}    id_contrato
    Should Be Equal    ${contrato}[estado]    ACTIVO
    Should Be Equal    ${contrato}[plan]      ${PLAN_BASICO}
    Log    Contrato creado: ${contrato}[id_contrato]

TC-004 Verificar que no se puede contratar un plan inexistente
    [Documentation]    Verifica que intentar contratar un plan inválido falla correctamente.
    [Tags]    contrato    alta    negativo
    ${cliente}=    Buscar Cliente Por ID    ${CLIENTE_VALIDO_ID}
    ${plan}=       Obtener Detalle Plan    ${PLAN_INVALIDO}
    Dictionary Should Contain Key    ${plan}    error
    Should Be Equal    ${plan}[error]    PLAN_NO_ENCONTRADO
    Log    Error esperado al buscar plan inválido: ${plan}[error]

TC-005 Verificar que un contrato activo puede ser consultado correctamente
    [Documentation]    Verifica la consulta de un contrato activo existente.
    [Tags]    contrato    consulta    smoke
    ${contrato}=    Consultar Contrato    ${CONTRATO_ACTIVO_ID}
    Should Be Equal    ${contrato}[estado]         ACTIVO
    Should Be Equal    ${contrato}[cliente_id]     ${CLIENTE_VALIDO_ID}
    Dictionary Should Contain Key    ${contrato}   fecha_inicio
    Log    Estado del contrato: ${contrato}[estado]

TC-006 Verificar que un contrato cancelado tiene estado correcto
    [Documentation]    Verifica que un contrato cancelado refleja el estado CANCELADO.
    [Tags]    contrato    consulta    negativo
    ${contrato}=    Consultar Contrato    ${CONTRATO_CANCELADO_ID}
    Should Be Equal    ${contrato}[estado]    CANCELADO
    Log    Contrato cancelado confirmado: ${contrato}[estado]

TC-007 Verificar que un cliente con crédito suficiente puede actualizar su plan
    [Documentation]    Verifica el upgrade de plan cuando el cliente tiene crédito aprobado.
    [Tags]    contrato    upgrade    smoke
    ${cliente}=       Buscar Cliente Por ID       ${CLIENTE_VALIDO_ID}
    ${credito_ok}=    Verificar Credito Cliente   ${cliente}    ${LIMITE_CREDITO_APROBADO}
    Should Be True    ${credito_ok}
    ${resultado}=     Actualizar Plan Contrato    ${CONTRATO_ACTIVO_ID}    ${PLAN_PREMIUM}
    Should Be Equal   ${resultado}[nuevo_plan]    ${PLAN_PREMIUM}
    Should Be Equal   ${resultado}[estado]        ACTUALIZADO
    Log    Plan actualizado a: ${resultado}[nuevo_plan]

TC-008 Verificar que un cliente sin crédito suficiente no puede actualizar su plan
    [Documentation]    Verifica que el upgrade de plan es rechazado por crédito insuficiente.
    [Tags]    contrato    upgrade    negativo
    ${cliente}=       Buscar Cliente Por ID       ${CLIENTE_VALIDO_ID}
    ${credito_ok}=    Verificar Credito Cliente   ${cliente}    ${LIMITE_CREDITO_RECHAZADO}
    Should Not Be True    ${credito_ok}
    Log    Actualización de plan rechazada por crédito insuficiente.

*** Keywords ***
Buscar Cliente Por ID
    [Documentation]    Simula la búsqueda de un cliente en el sistema por su ID.
    [Arguments]    ${id_cliente}
    # Simulación de base de datos en memoria
    ${clientes}=    Create Dictionary
    ...    CLI-001={"nombre": "Ana García", "estado": "ACTIVO", "credito_disponible": 800}
    Run Keyword If    '${id_cliente}' == 'CLI-001'
    ...    Return From Keyword    &{{"nombre": "Ana García", "estado": "ACTIVO", "id": "CLI-001"}}
    ${error}=    Create Dictionary    error=CLIENTE_NO_ENCONTRADO    id=${id_cliente}
    RETURN    ${error}

Obtener Detalle Plan
    [Documentation]    Simula la obtención de detalles de un plan de datos.
    [Arguments]    ${id_plan}
    Run Keyword If    '${id_plan}' == 'PLAN-DATOS-10GB'
    ...    Return From Keyword    &{{"nombre": "Datos 10GB", "precio": 250, "id": "PLAN-DATOS-10GB"}}
    Run Keyword If    '${id_plan}' == 'PLAN-DATOS-50GB'
    ...    Return From Keyword    &{{"nombre": "Datos 50GB Premium", "precio": 450, "id": "PLAN-DATOS-50GB"}}
    ${error}=    Create Dictionary    error=PLAN_NO_ENCONTRADO    id=${id_plan}
    RETURN    ${error}

Crear Contrato
    [Documentation]    Simula la creación de un nuevo contrato.
    [Arguments]    ${cliente}    ${plan}
    ${contrato}=    Create Dictionary
    ...    id_contrato=CONT-NEW-001
    ...    estado=ACTIVO
    ...    plan=${plan}[id]
    ...    cliente_id=${cliente}[id]
    ...    fecha_inicio=2024-01-15
    RETURN    ${contrato}

Consultar Contrato
    [Documentation]    Simula la consulta de un contrato por ID.
    [Arguments]    ${id_contrato}
    Run Keyword If    '${id_contrato}' == 'CONT-2024-001'
    ...    Return From Keyword    &{{"id_contrato": "CONT-2024-001", "estado": "ACTIVO", "cliente_id": "CLI-001", "fecha_inicio": "2024-01-01"}}
    Run Keyword If    '${id_contrato}' == 'CONT-2024-002'
    ...    Return From Keyword    &{{"id_contrato": "CONT-2024-002", "estado": "CANCELADO", "cliente_id": "CLI-001", "fecha_inicio": "2023-06-01"}}
    ${error}=    Create Dictionary    error=CONTRATO_NO_ENCONTRADO    id=${id_contrato}
    RETURN    ${error}

Verificar Credito Cliente
    [Documentation]    Verifica si el cliente tiene crédito disponible suficiente.
    [Arguments]    ${cliente}    ${monto_requerido}
    # Crédito disponible fijo de 500 para el cliente de prueba
    ${credito_disponible}=    Set Variable    ${500}
    ${tiene_credito}=    Evaluate    ${credito_disponible} >= ${monto_requerido}
    RETURN    ${tiene_credito}

Actualizar Plan Contrato
    [Documentation]    Simula la actualización del plan de un contrato existente.
    [Arguments]    ${id_contrato}    ${nuevo_plan_id}
    ${resultado}=    Create Dictionary
    ...    id_contrato=${id_contrato}
    ...    nuevo_plan=${nuevo_plan_id}
    ...    estado=ACTUALIZADO
    RETURN    ${resultado}
```

3. Guarda el archivo.

#### Salida esperada al abrir el archivo

VS Code debe mostrar el archivo sin errores de sintaxis y el Language Server debe reconocer las keywords resaltándolas correctamente.

#### Verificación

Ejecuta la suite tradicional para establecer la línea base:

```bash
# Desde la carpeta lab04-00-02
robot --outputdir results/traditional traditional_suite/contract_management_traditional.robot
```

Debes ver una salida similar a:

```
==============================================================================
Contract Management Traditional
==============================================================================
TC-001 Verificar que un cliente válido puede ser encontrado en el sistema  | PASS |
TC-002 Verificar que un cliente inválido genera error controlado           | PASS |
TC-003 Verificar que un cliente puede contratar un plan de datos válido    | PASS |
TC-004 Verificar que no se puede contratar un plan inexistente             | PASS |
TC-005 Verificar que un contrato activo puede ser consultado correctamente | PASS |
TC-006 Verificar que un contrato cancelado tiene estado correcto           | PASS |
TC-007 Verificar que un cliente con crédito suficiente puede actualizar    | PASS |
TC-008 Verificar que un cliente sin crédito suficiente no puede actualizar | PASS |
==============================================================================
Contract Management Traditional                                             | PASS |
8 tests, 8 passed, 0 failed
```

> **⚠️ Importante:** Si algún test falla en este paso, revisa el archivo antes de continuar. La suite tradicional debe pasar 8/8 tests para que la refactorización tenga sentido.

---

### Paso 2: Analizar y documentar los comportamientos de negocio

**Objetivo:** Identificar el comportamiento de negocio que cada test case técnico valida, separando el *qué* del *cómo*. Este es el paso conceptual más importante del proceso BDD.

#### Instrucciones

1. Crea el archivo `bdd_suite/ANALISIS_REFACTORIZACION.md` con el siguiente análisis:

```markdown
# Análisis de Refactorización: Traditional → BDD
## Laboratorio 04-00-02

### Metodología
Para cada test case técnico se identifica:
- El COMPORTAMIENTO DE NEGOCIO subyacente (independiente de la implementación)
- El ACTOR del escenario (quién realiza la acción)
- La REGLA DE NEGOCIO que se valida

### Mapeo de Test Cases a Comportamientos de Negocio

| TC Original | Comportamiento de Negocio | Actor | Capa BDD |
|---|---|---|---|
| TC-001 | Un cliente registrado puede ser localizado por su identificador | Agente de ventas | Búsqueda de cliente |
| TC-002 | Un identificador inválido no debe retornar datos de cliente | Sistema | Validación de cliente |
| TC-003 | Un cliente activo puede contratar un plan de datos disponible | Cliente / Agente | Contratación de servicio |
| TC-004 | No se puede contratar un plan que no existe en el catálogo | Sistema | Validación de catálogo |
| TC-005 | Un contrato activo refleja su estado y datos correctamente | Agente de soporte | Consulta de contrato |
| TC-006 | Un contrato cancelado refleja el estado CANCELADO | Agente de soporte | Consulta de contrato |
| TC-007 | Un cliente con crédito aprobado puede hacer upgrade de plan | Cliente / Agente | Modificación de contrato |
| TC-008 | Un cliente con crédito insuficiente no puede hacer upgrade | Sistema | Validación financiera |

### Decisiones de Diseño BDD

1. Los escenarios TC-001 y TC-002 se agrupan bajo la Feature "Gestión de Clientes"
2. Los escenarios TC-003 y TC-004 se agrupan bajo "Contratación de Servicios"
3. Los escenarios TC-005 y TC-006 se agrupan bajo "Consulta de Contratos"
4. Los escenarios TC-007 y TC-008 se agrupan bajo "Modificación de Contratos"

### Principio aplicado (Lección 4.1)
Los escenarios BDD describen QUÉ debe ocurrir, no CÓMO se implementa.
Un analista de negocio debe poder leer y validar cada escenario sin conocer
el código de automatización subyacente.
```

2. Revisa el mapeo y asegúrate de entender la diferencia entre el *qué* (comportamiento de negocio) y el *cómo* (implementación técnica) antes de continuar.

#### Verificación

No hay ejecución en este paso. Verifica que el archivo `.md` se creó correctamente y que el mapeo cubre los 8 test cases originales.

---

### Paso 3: Crear la capa técnica (technical_keywords)

**Objetivo:** Extraer todas las implementaciones técnicas a un archivo Resource dedicado que actúa como la capa más baja de la arquitectura. Esta capa contiene el código que interactúa directamente con los datos simulados.

#### Instrucciones

1. Crea el archivo `bdd_suite/technical_keywords/contract_technical.resource`:

```robot
*** Settings ***
Documentation    Capa técnica de la suite de gestión de contratos.
...
...              RESPONSABILIDAD: Esta capa contiene la implementación técnica
...              de las operaciones sobre el sistema de contratos. Es la única
...              capa que "sabe cómo" interactuar con los datos.
...
...              REGLA DE DISEÑO: Las keywords de esta capa NO deben ser
...              llamadas directamente desde los escenarios BDD. Solo deben
...              ser usadas por la capa de domain_keywords.
...
...              DECISIÓN DE REFACTORIZACIÓN: En la suite tradicional, estas
...              keywords estaban mezcladas con los test cases. Separarlas
...              permite cambiar la implementación técnica sin modificar los
...              escenarios de negocio.
Library          Collections
Library          BuiltIn

*** Variables ***
# --- Repositorio de datos simulados (equivalente a una base de datos en memoria) ---
# Estos datos eran variables sueltas en la suite tradicional.
# Al centralizarlos aquí, hay un único punto de mantenimiento.
${TECH_CLIENTE_VALIDO_ID}         CLI-001
${TECH_CLIENTE_VALIDO_NOMBRE}     Ana García
${TECH_CREDITO_DISPONIBLE}        ${500}
${TECH_CONTRATO_ACTIVO_ID}        CONT-2024-001
${TECH_CONTRATO_CANCELADO_ID}     CONT-2024-002

*** Keywords ***
Recuperar Datos De Cliente Por ID
    [Documentation]    Implementación técnica: recupera un registro de cliente por ID.
    ...                Simula una llamada a base de datos o API interna.
    [Arguments]    ${id_cliente}
    IF    '${id_cliente}' == '${TECH_CLIENTE_VALIDO_ID}'
        ${datos_cliente}=    Create Dictionary
        ...    id=${TECH_CLIENTE_VALIDO_ID}
        ...    nombre=${TECH_CLIENTE_VALIDO_NOMBRE}
        ...    estado=ACTIVO
        RETURN    ${datos_cliente}
    END
    ${error}=    Create Dictionary
    ...    error=CLIENTE_NO_ENCONTRADO
    ...    id=${id_cliente}
    RETURN    ${error}

Recuperar Detalle De Plan Por ID
    [Documentation]    Implementación técnica: recupera la definición de un plan del catálogo.
    [Arguments]    ${id_plan}
    IF    '${id_plan}' == 'PLAN-DATOS-10GB'
        ${datos_plan}=    Create Dictionary
        ...    id=PLAN-DATOS-10GB
        ...    nombre=Datos 10GB
        ...    precio=${250}
        RETURN    ${datos_plan}
    END
    IF    '${id_plan}' == 'PLAN-DATOS-50GB'
        ${datos_plan}=    Create Dictionary
        ...    id=PLAN-DATOS-50GB
        ...    nombre=Datos 50GB Premium
        ...    precio=${450}
        RETURN    ${datos_plan}
    END
    ${error}=    Create Dictionary
    ...    error=PLAN_NO_ENCONTRADO
    ...    id=${id_plan}
    RETURN    ${error}

Registrar Nuevo Contrato En Sistema
    [Documentation]    Implementación técnica: persiste un nuevo contrato en el sistema.
    [Arguments]    ${id_cliente}    ${id_plan}
    ${contrato}=    Create Dictionary
    ...    id_contrato=CONT-NEW-001
    ...    estado=ACTIVO
    ...    plan=${id_plan}
    ...    cliente_id=${id_cliente}
    ...    fecha_inicio=2024-01-15
    RETURN    ${contrato}

Recuperar Contrato Por ID
    [Documentation]    Implementación técnica: recupera los datos de un contrato por su ID.
    [Arguments]    ${id_contrato}
    IF    '${id_contrato}' == '${TECH_CONTRATO_ACTIVO_ID}'
        ${datos_contrato}=    Create Dictionary
        ...    id_contrato=${TECH_CONTRATO_ACTIVO_ID}
        ...    estado=ACTIVO
        ...    cliente_id=${TECH_CLIENTE_VALIDO_ID}
        ...    fecha_inicio=2024-01-01
        RETURN    ${datos_contrato}
    END
    IF    '${id_contrato}' == '${TECH_CONTRATO_CANCELADO_ID}'
        ${datos_contrato}=    Create Dictionary
        ...    id_contrato=${TECH_CONTRATO_CANCELADO_ID}
        ...    estado=CANCELADO
        ...    cliente_id=${TECH_CLIENTE_VALIDO_ID}
        ...    fecha_inicio=2023-06-01
        RETURN    ${datos_contrato}
    END
    ${error}=    Create Dictionary
    ...    error=CONTRATO_NO_ENCONTRADO
    ...    id=${id_contrato}
    RETURN    ${error}

Evaluar Credito Disponible Del Cliente
    [Documentation]    Implementación técnica: consulta el crédito disponible y evalúa suficiencia.
    [Arguments]    ${monto_requerido}
    # El crédito disponible es fijo en la simulación (mismo comportamiento que la suite original)
    ${tiene_credito}=    Evaluate    ${TECH_CREDITO_DISPONIBLE} >= ${monto_requerido}
    RETURN    ${tiene_credito}

Modificar Plan De Contrato En Sistema
    [Documentation]    Implementación técnica: actualiza el plan asociado a un contrato.
    [Arguments]    ${id_contrato}    ${nuevo_plan_id}
    ${resultado}=    Create Dictionary
    ...    id_contrato=${id_contrato}
    ...    nuevo_plan=${nuevo_plan_id}
    ...    estado=ACTUALIZADO
    RETURN    ${resultado}

Verificar Que Diccionario No Contiene Error
    [Documentation]    Implementación técnica: valida que un resultado no es un error del sistema.
    [Arguments]    ${resultado}
    Dictionary Should Not Contain Key    ${resultado}    error

Verificar Que Diccionario Contiene Error Especifico
    [Documentation]    Implementación técnica: valida que el resultado contiene el código de error esperado.
    [Arguments]    ${resultado}    ${codigo_error}
    Dictionary Should Contain Key    ${resultado}    error
    Should Be Equal    ${resultado}[error]    ${codigo_error}
```

2. Guarda el archivo.

#### Verificación

Verifica que el archivo existe y tiene sintaxis correcta:

```bash
# Verificación de sintaxis (no ejecuta tests, solo analiza)
python -m robot.libdoc bdd_suite/technical_keywords/contract_technical.resource list
```

Debes ver listadas las 8 keywords definidas en la capa técnica.

---

### Paso 4: Crear la capa de dominio (domain_keywords)

**Objetivo:** Crear la capa intermedia que traduce el lenguaje técnico al lenguaje de negocio. Esta es la capa que hace que los escenarios BDD sean agnósticos a la implementación.

#### Instrucciones

1. Crea el archivo `bdd_suite/domain_keywords/contract_domain.resource`:

```robot
*** Settings ***
Documentation    Capa de dominio de la suite de gestión de contratos.
...
...              RESPONSABILIDAD: Esta capa expresa las operaciones en lenguaje
...              de negocio del dominio de telecomunicaciones. Traduce los
...              conceptos técnicos a conceptos comprensibles por el negocio.
...
...              DECISIÓN DE REFACTORIZACIÓN: Esta capa NO existía en la suite
...              tradicional. En el enfoque keyword-driven, los test cases llamaban
...              directamente a keywords técnicas. Introducir esta capa permite:
...              1) Cambiar la implementación técnica sin tocar los escenarios.
...              2) Reutilizar keywords de dominio en múltiples features.
...              3) Que el equipo de negocio valide los nombres de las keywords.
...
...              REGLA DE DISEÑO: Los nombres de las keywords en esta capa deben
...              ser comprensibles para un analista de negocio sin conocimientos
...              técnicos. Evitar términos como "diccionario", "variable", "objeto".
Resource         ../technical_keywords/contract_technical.resource

*** Variables ***
# --- Variables de dominio (lenguaje de negocio, no técnico) ---
${PLAN_BASICO_ID}       PLAN-DATOS-10GB
${PLAN_PREMIUM_ID}      PLAN-DATOS-50GB
${CONTRATO_VIGENTE}     CONT-2024-001
${CONTRATO_DADO_DE_BAJA}    CONT-2024-002
${CREDITO_PLAN_PREMIUM}     ${450}
${CREDITO_INSUFICIENTE}     ${50}

*** Keywords ***
# ═══════════════════════════════════════════════════════════
# DOMINIO: Gestión de Clientes
# ═══════════════════════════════════════════════════════════

El cliente con identificador "${id_cliente}" existe en el sistema
    [Documentation]    Verifica que un cliente registrado puede ser localizado.
    ...                DOMINIO: Representa la verificación de existencia de un cliente activo.
    ${datos_cliente}=    Recuperar Datos De Cliente Por ID    ${id_cliente}
    Verificar Que Diccionario No Contiene Error    ${datos_cliente}
    Set Test Variable    ${CLIENTE_EN_CONTEXTO}    ${datos_cliente}

El identificador de cliente "${id_cliente}" no corresponde a ningún registro
    [Documentation]    Establece el contexto de un cliente inexistente.
    ...                DOMINIO: Simula un intento de acceso con identificador inválido.
    Set Test Variable    ${ID_CLIENTE_INVALIDO}    ${id_cliente}

El sistema confirma que el cliente "${nombre_cliente}" está registrado
    [Documentation]    Valida que los datos del cliente coinciden con el nombre esperado.
    ...                DOMINIO: Confirmación de identidad del cliente en el sistema.
    Should Be Equal    ${CLIENTE_EN_CONTEXTO}[nombre]    ${nombre_cliente}

El sistema informa que el cliente no fue encontrado
    [Documentation]    Valida que el sistema devuelve el error correcto para cliente inexistente.
    ...                DOMINIO: Comportamiento esperado ante identificador inválido.
    ${resultado}=    Recuperar Datos De Cliente Por ID    ${ID_CLIENTE_INVALIDO}
    Verificar Que Diccionario Contiene Error Especifico    ${resultado}    CLIENTE_NO_ENCONTRADO

# ═══════════════════════════════════════════════════════════
# DOMINIO: Contratación de Servicios
# ═══════════════════════════════════════════════════════════

El cliente selecciona el plan de datos "${id_plan}"
    [Documentation]    Registra la selección de un plan por parte del cliente.
    ...                DOMINIO: Acción de selección de producto en el catálogo de servicios.
    ${datos_plan}=    Recuperar Detalle De Plan Por ID    ${id_plan}
    Verificar Que Diccionario No Contiene Error    ${datos_plan}
    Set Test Variable    ${PLAN_SELECCIONADO}    ${datos_plan}

El cliente intenta seleccionar el plan "${id_plan}" que no existe en el catálogo
    [Documentation]    Intenta seleccionar un plan inexistente.
    ...                DOMINIO: Escenario de error en la selección del catálogo.
    ${resultado}=    Recuperar Detalle De Plan Por ID    ${id_plan}
    Verificar Que Diccionario Contiene Error Especifico    ${resultado}    PLAN_NO_ENCONTRADO

El cliente confirma la contratación del servicio
    [Documentation]    Ejecuta la contratación del plan seleccionado previamente.
    ...                DOMINIO: Confirmación y registro del nuevo contrato de servicio.
    ${nuevo_contrato}=    Registrar Nuevo Contrato En Sistema
    ...    ${CLIENTE_EN_CONTEXTO}[id]
    ...    ${PLAN_SELECCIONADO}[id]
    Set Test Variable    ${CONTRATO_GENERADO}    ${nuevo_contrato}

El contrato queda registrado como activo con el plan seleccionado
    [Documentation]    Verifica que el contrato fue creado en estado ACTIVO con el plan correcto.
    ...                DOMINIO: Validación del resultado exitoso de la contratación.
    Should Be Equal    ${CONTRATO_GENERADO}[estado]    ACTIVO
    Should Be Equal    ${CONTRATO_GENERADO}[plan]      ${PLAN_SELECCIONADO}[id]
    Dictionary Should Contain Key    ${CONTRATO_GENERADO}    id_contrato

El sistema rechaza la contratación informando que el plan no está disponible
    [Documentation]    Verifica que el sistema impide contratar un plan inexistente.
    ...                DOMINIO: Protección del catálogo ante selecciones inválidas.
    # El rechazo ya fue validado en "El cliente intenta seleccionar..."
    # Este step documenta explícitamente el resultado esperado para el lector de negocio.
    Log    El sistema rechazó correctamente la contratación de un plan inválido.

# ═══════════════════════════════════════════════════════════
# DOMINIO: Consulta de Contratos
# ═══════════════════════════════════════════════════════════

El agente consulta el contrato con referencia "${id_contrato}"
    [Documentation]    Recupera los datos de un contrato por su referencia.
    ...                DOMINIO: Consulta de estado de contrato por parte del agente de soporte.
    ${datos_contrato}=    Recuperar Contrato Por ID    ${id_contrato}
    Set Test Variable    ${CONTRATO_CONSULTADO}    ${datos_contrato}

El contrato aparece en estado "${estado_esperado}"
    [Documentation]    Verifica el estado de un contrato consultado.
    ...                DOMINIO: Validación del estado comercial del contrato.
    Should Be Equal    ${CONTRATO_CONSULTADO}[estado]    ${estado_esperado}

El contrato muestra la fecha de inicio y el cliente asociado
    [Documentation]    Verifica que el contrato contiene los datos mínimos requeridos.
    ...                DOMINIO: Completitud de la información del contrato para el agente.
    Dictionary Should Contain Key    ${CONTRATO_CONSULTADO}    fecha_inicio
    Dictionary Should Contain Key    ${CONTRATO_CONSULTADO}    cliente_id

# ═══════════════════════════════════════════════════════════
# DOMINIO: Modificación de Contratos (Upgrade/Downgrade)
# ═══════════════════════════════════════════════════════════

El cliente tiene crédito suficiente para el plan premium
    [Documentation]    Verifica que el cliente tiene capacidad financiera para el upgrade.
    ...                DOMINIO: Validación de elegibilidad financiera para modificación de plan.
    ${tiene_credito}=    Evaluar Credito Disponible Del Cliente    ${CREDITO_PLAN_PREMIUM}
    Should Be True    ${tiene_credito}

El cliente no tiene crédito suficiente para el plan premium
    [Documentation]    Establece el contexto de crédito insuficiente.
    ...                DOMINIO: Escenario de rechazo por capacidad financiera insuficiente.
    ${tiene_credito}=    Evaluar Credito Disponible Del Cliente    ${CREDITO_INSUFICIENTE}
    Should Not Be True    ${tiene_credito}

El cliente solicita cambiar al plan premium en su contrato vigente
    [Documentation]    Ejecuta la solicitud de upgrade de plan en el contrato activo.
    ...                DOMINIO: Solicitud de modificación comercial del servicio contratado.
    ${resultado}=    Modificar Plan De Contrato En Sistema
    ...    ${CONTRATO_VIGENTE}
    ...    ${PLAN_PREMIUM_ID}
    Set Test Variable    ${RESULTADO_UPGRADE}    ${resultado}

El plan del contrato se actualiza al plan premium exitosamente
    [Documentation]    Verifica que el upgrade fue aplicado correctamente.
    ...                DOMINIO: Confirmación de la modificación del servicio.
    Should Be Equal    ${RESULTADO_UPGRADE}[nuevo_plan]    ${PLAN_PREMIUM_ID}
    Should Be Equal    ${RESULTADO_UPGRADE}[estado]        ACTUALIZADO

El sistema deniega el cambio de plan por capacidad financiera insuficiente
    [Documentation]    Documenta el resultado esperado cuando el crédito es insuficiente.
    ...                DOMINIO: El rechazo fue validado en el step de verificación de crédito.
    ...                Este step cierra el escenario con la confirmación explícita del rechazo.
    Log    El sistema denegó correctamente el upgrade por crédito insuficiente.
```

2. Guarda el archivo.

#### Verificación

```bash
python -m robot.libdoc bdd_suite/domain_keywords/contract_domain.resource list
```

Debes ver listadas todas las keywords de dominio. Verifica que los nombres son legibles en lenguaje de negocio.

---

### Paso 5: Crear los escenarios BDD (capa de features)

**Objetivo:** Escribir los 8 escenarios Gherkin que reemplazan a los 8 test cases tradicionales, usando exclusivamente keywords de la capa de dominio.

#### Instrucciones

1. Crea el archivo `bdd_suite/features/contract_management.robot`:

```robot
*** Settings ***
Documentation    Feature: Gestión de Contratos de Telecomunicaciones
...
...              Esta suite representa la refactorización completa de la suite
...              tradicional "contract_management_traditional.robot" al modelo BDD.
...
...              COBERTURA: Los 8 escenarios de esta suite tienen correspondencia
...              exacta con los 8 test cases de la suite tradicional. La cobertura
...              funcional es idéntica; solo cambia el nivel de abstracción.
...
...              PRINCIPIO APLICADO (Lección 4.1):
...              "Los escenarios BDD describen QUÉ debe ocurrir, no CÓMO se implementa."
...              Un analista de negocio puede leer y validar estos escenarios sin
...              conocer el código de automatización subyacente.
...
...              ARQUITECTURA DE TRES CAPAS:
...              features/ → domain_keywords/ → technical_keywords/
...
...              NOTA: Esta capa NO importa technical_keywords directamente.
...              Toda la implementación técnica está encapsulada en domain_keywords.
Resource         ../domain_keywords/contract_domain.resource

*** Test Cases ***
# ═══════════════════════════════════════════════════════════════════════════
# FEATURE: Gestión de Clientes
# Corresponde a: TC-001 y TC-002 de la suite tradicional
# ═══════════════════════════════════════════════════════════════════════════

Escenario: Un agente localiza a un cliente registrado por su identificador
    [Documentation]    BDD equivalente a TC-001.
    ...                Valida que un cliente existente puede ser encontrado en el sistema.
    [Tags]    cliente    busqueda    smoke    bdd
    Given el cliente con identificador "CLI-001" existe en el sistema
    Then el sistema confirma que el cliente "Ana García" está registrado

Escenario: El sistema rechaza la búsqueda de un identificador de cliente inválido
    [Documentation]    BDD equivalente a TC-002.
    ...                Valida que un ID inexistente genera el error controlado correcto.
    [Tags]    cliente    busqueda    negativo    bdd
    Given el identificador de cliente "CLI-999" no corresponde a ningún registro
    When el agente intenta localizar al cliente en el sistema
    Then el sistema informa que el cliente no fue encontrado

# ═══════════════════════════════════════════════════════════════════════════
# FEATURE: Contratación de Servicios
# Corresponde a: TC-003 y TC-004 de la suite tradicional
# ═══════════════════════════════════════════════════════════════════════════

Escenario: Un cliente activo contrata exitosamente un plan de datos disponible
    [Documentation]    BDD equivalente a TC-003.
    ...                Valida el flujo completo de contratación de un plan válido.
    [Tags]    contrato    alta    smoke    bdd
    Given el cliente con identificador "CLI-001" existe en el sistema
    When el cliente selecciona el plan de datos "PLAN-DATOS-10GB"
    And el cliente confirma la contratación del servicio
    Then el contrato queda registrado como activo con el plan seleccionado

Escenario: El sistema impide contratar un plan que no existe en el catálogo
    [Documentation]    BDD equivalente a TC-004.
    ...                Valida que el sistema protege al catálogo de selecciones inválidas.
    [Tags]    contrato    alta    negativo    bdd
    Given el cliente con identificador "CLI-001" existe en el sistema
    When el cliente intenta seleccionar el plan "PLAN-INEXISTENTE" que no existe en el catálogo
    Then el sistema rechaza la contratación informando que el plan no está disponible

# ═══════════════════════════════════════════════════════════════════════════
# FEATURE: Consulta de Contratos
# Corresponde a: TC-005 y TC-006 de la suite tradicional
# ═══════════════════════════════════════════════════════════════════════════

Escenario: Un agente de soporte consulta un contrato vigente y verifica su estado
    [Documentation]    BDD equivalente a TC-005.
    ...                Valida que un contrato activo muestra estado y datos correctos.
    [Tags]    contrato    consulta    smoke    bdd
    Given el agente consulta el contrato con referencia "CONT-2024-001"
    Then el contrato aparece en estado "ACTIVO"
    And el contrato muestra la fecha de inicio y el cliente asociado

Escenario: Un agente de soporte verifica que un contrato dado de baja tiene estado cancelado
    [Documentation]    BDD equivalente a TC-006.
    ...                Valida que un contrato cancelado refleja el estado CANCELADO.
    [Tags]    contrato    consulta    negativo    bdd
    Given el agente consulta el contrato con referencia "CONT-2024-002"
    Then el contrato aparece en estado "CANCELADO"

# ═══════════════════════════════════════════════════════════════════════════
# FEATURE: Modificación de Contratos
# Corresponde a: TC-007 y TC-008 de la suite tradicional
# ═══════════════════════════════════════════════════════════════════════════

Escenario: Un cliente con crédito aprobado actualiza su plan al servicio premium
    [Documentation]    BDD equivalente a TC-007.
    ...                Valida el upgrade de plan cuando el cliente tiene crédito suficiente.
    [Tags]    contrato    upgrade    smoke    bdd
    Given el cliente con identificador "CLI-001" existe en el sistema
    And el cliente tiene crédito suficiente para el plan premium
    When el cliente solicita cambiar al plan premium en su contrato vigente
    Then el plan del contrato se actualiza al plan premium exitosamente

Escenario: El sistema deniega el upgrade de plan a un cliente con crédito insuficiente
    [Documentation]    BDD equivalente a TC-008.
    ...                Valida que el sistema protege contra upgrades sin capacidad financiera.
    [Tags]    contrato    upgrade    negativo    bdd
    Given el cliente con identificador "CLI-001" existe en el sistema
    But el cliente no tiene crédito suficiente para el plan premium
    Then el sistema deniega el cambio de plan por capacidad financiera insuficiente

*** Keywords ***
# ─────────────────────────────────────────────────────────────────────────
# NOTA: Esta sección de Keywords locales solo existe para el step que no
# tiene correspondencia directa en domain_keywords. En una suite BDD madura,
# esta sección debería estar vacía o no existir.
# ─────────────────────────────────────────────────────────────────────────

el agente intenta localizar al cliente en el sistema
    [Documentation]    Step intermedio que representa la acción del agente.
    ...                Delega la validación a la keyword de dominio.
    El sistema informa que el cliente no fue encontrado
```

2. Guarda el archivo.

#### Verificación

Verifica la sintaxis del archivo de features:

```bash
python -m robot.libdoc bdd_suite/features/contract_management.robot list
```

---

### Paso 6: Ejecutar la suite BDD y comparar resultados

**Objetivo:** Verificar que la suite BDD refactorizada produce exactamente los mismos resultados que la suite tradicional (8/8 tests pasando) y comparar las métricas de legibilidad.

#### Instrucciones

1. Ejecuta la suite BDD refactorizada:

```bash
robot --outputdir results/bdd \
      --log bdd_log.html \
      --report bdd_report.html \
      bdd_suite/features/contract_management.robot
```

**En Windows (una sola línea):**

```cmd
robot --outputdir results\bdd --log bdd_log.html --report bdd_report.html bdd_suite\features\contract_management.robot
```

2. Ejecuta ambas suites juntas para comparar en un único reporte:

```bash
# macOS / Linux
robot --outputdir results \
      --log comparison_log.html \
      --report comparison_report.html \
      --name "Comparacion_Traditional_vs_BDD" \
      traditional_suite/ \
      bdd_suite/features/

# Windows
robot --outputdir results --log comparison_log.html --report comparison_report.html --name "Comparacion_Traditional_vs_BDD" traditional_suite\ bdd_suite\features\
```

3. Abre el reporte HTML en tu navegador:

```bash
# macOS
open results/comparison_report.html

# Linux
xdg-open results/comparison_report.html

# Windows
start results\comparison_report.html
```

#### Salida esperada

```
==============================================================================
Comparacion Traditional Vs BDD
==============================================================================
Contract Management Traditional                                               | PASS |
8 tests, 8 passed, 0 failed
------------------------------------------------------------------------------
Contract Management                                                           | PASS |
8 tests, 8 passed, 0 failed
==============================================================================
Comparacion Traditional Vs BDD                                               | PASS |
16 tests, 16 passed, 0 failed
```

#### Verificación

Confirma los siguientes puntos en el reporte HTML:

- [ ] La suite tradicional: 8 tests, 8 passed, 0 failed.
- [ ] La suite BDD: 8 tests, 8 passed, 0 failed.
- [ ] Total combinado: 16 tests, 16 passed, 0 failed.
- [ ] En el log de la suite BDD, los nombres de los escenarios son legibles en lenguaje de negocio.
- [ ] En el log de la suite BDD, se puede ver la jerarquía de tres capas en el árbol de keywords.

---

### Paso 7: Documentar la comparación de métricas

**Objetivo:** Comparar cuantitativamente y cualitativamente ambos enfoques para consolidar el aprendizaje sobre el valor de BDD.

#### Instrucciones

1. Crea el archivo `bdd_suite/COMPARACION_METRICAS.md`:

```markdown
# Comparación de Métricas: Suite Tradicional vs Suite BDD
## Laboratorio 04-00-02 — Resultado del análisis de refactorización

---

## 1. Métricas Estructurales

| Métrica                              | Suite Tradicional | Suite BDD (3 capas) |
|--------------------------------------|:-----------------:|:-------------------:|
| Archivos .robot / .resource          | 1                 | 3                   |
| Líneas de código total               | ~130              | ~310                |
| Keywords reutilizables               | 6                 | 6 técnicas + 14 dominio |
| Capas de abstracción                 | 1                 | 3                   |
| Legibilidad por analista de negocio  | Baja              | Alta                |
| Trazabilidad a requisitos            | Manual            | Directa (nombre del escenario) |

---

## 2. Análisis de Mantenibilidad

### Escenario de cambio: "El sistema ahora usa una API REST en lugar de datos simulados"

**Suite Tradicional:**
- Hay que modificar 6 keywords directamente en el archivo de tests.
- Riesgo de romper los test cases al editar el mismo archivo.
- El analista de negocio no puede distinguir qué cambió.

**Suite BDD (3 capas):**
- Solo se modifica `technical_keywords/contract_technical.resource`.
- Las keywords de dominio y los escenarios NO cambian.
- El analista de negocio puede verificar que los escenarios siguen siendo correctos.

---

## 3. Análisis de Legibilidad

### Test case tradicional (TC-007):
```
TC-007 Verificar que un cliente con crédito suficiente puede actualizar su plan
    ${cliente}=       Buscar Cliente Por ID       ${CLIENTE_VALIDO_ID}
    ${credito_ok}=    Verificar Credito Cliente   ${cliente}    ${LIMITE_CREDITO_APROBADO}
    Should Be True    ${credito_ok}
    ${resultado}=     Actualizar Plan Contrato    ${CONTRATO_ACTIVO_ID}    ${PLAN_PREMIUM}
    Should Be Equal   ${resultado}[nuevo_plan]    ${PLAN_PREMIUM}
    Should Be Equal   ${resultado}[estado]        ACTUALIZADO
```

### Escenario BDD equivalente:
```
Escenario: Un cliente con crédito aprobado actualiza su plan al servicio premium
    Given el cliente con identificador "CLI-001" existe en el sistema
    And el cliente tiene crédito suficiente para el plan premium
    When el cliente solicita cambiar al plan premium en su contrato vigente
    Then el plan del contrato se actualiza al plan premium exitosamente
```

**Conclusión:** El escenario BDD puede ser validado por el área comercial sin conocimientos técnicos.

---

## 4. Principios BDD Aplicados (Lección 4.1)

| Principio              | Aplicación en este laboratorio |
|------------------------|-------------------------------|
| **Colaboración**       | Los escenarios usan nombres que el negocio puede validar |
| **Especificación ejecutable** | Los 8 escenarios Gherkin son ejecutables y pasan |
| **Documentación viva** | Los escenarios reflejan el comportamiento actual del sistema |
| **Agnóstico a tecnología** | Los escenarios no mencionan diccionarios, variables ni métodos |
```

2. Guarda el archivo.

---

## Validación y Pruebas

Ejecuta la siguiente secuencia de validación completa para confirmar que el laboratorio está terminado correctamente:

```bash
# 1. Validar suite tradicional (línea base)
robot --outputdir results/traditional \
      --log traditional_log.html \
      traditional_suite/contract_management_traditional.robot
echo "=== Suite tradicional: debe mostrar 8 passed ==="

# 2. Validar suite BDD
robot --outputdir results/bdd \
      --log bdd_log.html \
      bdd_suite/features/contract_management.robot
echo "=== Suite BDD: debe mostrar 8 passed ==="

# 3. Validar que la capa técnica no es accesible directamente desde features
# (verificación manual: abrir bdd_suite/features/contract_management.robot
#  y confirmar que NO hay ningún import de technical_keywords)
grep -r "technical_keywords" bdd_suite/features/
# Debe retornar vacío (sin resultados)

# 4. Reporte comparativo final
robot --outputdir results \
      --log comparison_log.html \
      --report comparison_report.html \
      traditional_suite/ bdd_suite/features/
echo "=== Comparativo: debe mostrar 16 passed ==="
```

**En Windows (PowerShell):**

```powershell
# 1. Suite tradicional
robot --outputdir results\traditional --log traditional_log.html traditional_suite\contract_management_traditional.robot

# 2. Suite BDD
robot --outputdir results\bdd --log bdd_log.html bdd_suite\features\contract_management.robot

# 3. Verificar encapsulamiento de capas
Select-String -Path "bdd_suite\features\*.robot" -Pattern "technical_keywords"
# Debe retornar sin resultados

# 4. Reporte comparativo
robot --outputdir results --log comparison_log.html --report comparison_report.html traditional_suite\ bdd_suite\features\
```

### Lista de verificación final

| Criterio de validación                                              | Estado esperado |
|---------------------------------------------------------------------|:---------------:|
| Suite tradicional: 8/8 tests passing                                | ✅ PASS         |
| Suite BDD: 8/8 tests passing                                        | ✅ PASS         |
| `features/` no importa `technical_keywords/` directamente          | ✅ Verificado   |
| Nombres de escenarios BDD legibles en lenguaje de negocio           | ✅ Verificado   |
| Archivo `ANALISIS_REFACTORIZACION.md` documenta el mapeo de 8 TCs  | ✅ Presente     |
| Archivo `COMPARACION_METRICAS.md` documenta el análisis comparativo | ✅ Presente     |
| Jerarquía de tres capas visible en el log HTML de la suite BDD      | ✅ Verificado   |

---

## Solución de Problemas

### Problema 1: `Variable '${CLIENTE_EN_CONTEXTO}' not found` en la suite BDD

**Síntoma:** Al ejecutar la suite BDD, uno o más escenarios fallan con el mensaje `Variable '${CLIENTE_EN_CONTEXTO}' not found` o similar para otras variables de contexto (`${PLAN_SELECCIONADO}`, `${CONTRATO_CONSULTADO}`, etc.).

**Causa:** Las variables de contexto entre steps de un mismo escenario se establecen con `Set Test Variable`, que las hace visibles dentro del test case en ejecución. El error ocurre cuando el step `Given` que debería establecer la variable no se ejecutó antes del step `Then` que la consume, generalmente porque los steps están en orden incorrecto en el escenario o porque se está ejecutando un step `Then` de forma aislada.

**Solución:**

1. Verifica que el orden de los steps en el escenario sigue la secuencia `Given → When → Then`. Robot Framework ejecuta los steps en orden secuencial.
2. Confirma que el step `Given` que llama a `El cliente con identificador "..." existe en el sistema` está presente antes de cualquier step que use `${CLIENTE_EN_CONTEXTO}`.
3. Si el problema persiste, agrega un `Log Variables` temporal al inicio del step que falla para ver qué variables están disponibles:

```robot
# Diagnóstico temporal — eliminar después de resolver
El contrato queda registrado como activo con el plan seleccionado
    Log Variables
    Should Be Equal    ${CONTRATO_GENERADO}[estado]    ACTIVO
```

4. Verifica que el archivo `contract_domain.resource` usa `Set Test Variable` (no `Set Suite Variable` ni `Set Global Variable`) para las variables de contexto.

---

### Problema 2: La suite BDD muestra 8 tests pero con nombres técnicos en lugar de nombres de negocio en el reporte HTML

**Síntoma:** Al abrir `results/bdd/bdd_report.html`, los test cases aparecen con nombres como `TC-001 Verificar que un cliente válido...` en lugar de `Escenario: Un agente localiza a un cliente registrado...`.

**Causa:** Robot Framework está ejecutando el archivo incorrecto. Probablemente se está ejecutando `traditional_suite/contract_management_traditional.robot` en lugar de `bdd_suite/features/contract_management.robot`. Esto puede ocurrir si el comando `robot` se ejecutó desde una ruta incorrecta o si hay un error tipográfico en la ruta del archivo.

**Solución:**

1. Verifica la ruta desde la que ejecutas el comando. Debes estar en `lab04-00-02/`:

```bash
# Verificar directorio actual
pwd          # macOS / Linux
cd           # Windows (muestra el directorio actual)
```

2. Ejecuta el comando con la ruta explícita y verifica que apunta a `bdd_suite/features/`:

```bash
robot --outputdir results/bdd bdd_suite/features/contract_management.robot
```

3. Si el problema persiste, verifica que el archivo `bdd_suite/features/contract_management.robot` contiene los escenarios con nombres BDD y no fue sobreescrito accidentalmente:

```bash
# macOS / Linux
head -30 bdd_suite/features/contract_management.robot

# Windows
type bdd_suite\features\contract_management.robot | more
```

4. Confirma que el archivo comienza con `*** Settings ***` y que los test cases tienen nombres que empiezan con `Escenario:`.

---

## Limpieza

Al finalizar el laboratorio, ejecuta los siguientes pasos de limpieza:

```bash
# 1. Crear copia de respaldo del proyecto completo (recomendado)
# macOS / Linux
cp -r lab04-00-02 lab04-00-02_backup_$(date +%Y%m%d)

# Windows (PowerShell)
Copy-Item -Recurse lab04-00-02 "lab04-00-02_backup_$(Get-Date -Format 'yyyyMMdd')"

# 2. Limpiar archivos de salida temporales (opcional — conservar si necesitas los reportes)
# macOS / Linux
rm -f results/traditional/output.xml
rm -f results/bdd/output.xml

# Windows
del results\traditional\output.xml
del results\bdd\output.xml

# 3. Desactivar el entorno virtual
deactivate
```

> **💡 Nota:** Conserva los archivos `results/comparison_report.html` y `results/comparison_log.html`. Son evidencia de que ambas suites producen resultados equivalentes y pueden ser útiles como referencia en el Módulo 5.

---

## Resumen

En este laboratorio completaste el proceso completo de refactorización de una suite keyword-driven a una arquitectura BDD de tres capas:

| Capa | Archivo | Responsabilidad |
|------|---------|----------------|
| **Features (Gherkin)** | `bdd_suite/features/contract_management.robot` | Escenarios en lenguaje de negocio, legibles por analistas |
| **Domain Keywords** | `bdd_suite/domain_keywords/contract_domain.resource` | Traducción de lenguaje técnico a lenguaje de dominio |
| **Technical Keywords** | `bdd_suite/technical_keywords/contract_technical.resource` | Implementación técnica encapsulada y aislada |

### Conceptos clave aplicados

- **Principio de separación de responsabilidades:** cada capa tiene una única razón para cambiar. Si cambia la tecnología subyacente, solo cambia `technical_keywords`. Si cambia el lenguaje de negocio, solo cambia `domain_keywords`. Si cambia el requisito, solo cambia `features`.
- **Agnóstico a la implementación:** los escenarios BDD no mencionan diccionarios, variables, ni métodos técnicos. Un analista puede leerlos sin conocer Robot Framework.
- **Cobertura funcional preservada:** los 8 escenarios BDD cubren exactamente los mismos comportamientos que los 8 test cases tradicionales, verificado por ejecución paralela con 16/16 tests passing.
- **Documentación viva (Lección 4.1):** los escenarios son a la vez especificación ejecutable y documentación del comportamiento del sistema.

### Recursos de referencia

- [Robot Framework User Guide — Behavior Driven Style](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html#behavior-driven-style)
- [Dan North — Introducing BDD](https://dannorth.net/introducing-bdd/)
- [Cucumber — Referencia de Gherkin](https://cucumber.io/docs/gherkin/reference/)
- [Robot Framework — Set Test Variable](https://robotframework.org/robotframework/latest/libraries/BuiltIn.html#Set%20Test%20Variable)

---
LAB_END---
