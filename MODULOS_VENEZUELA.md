# Guía de Implementación Odoo 19 — Venezuela

**Repositorio:** odoo-venezuela (rama 19.0)  
**Módulos locales:** 33  
**Mantenedor:** Binaural Dev  
**Última revisión:** 2026-06-02

> Este documento es la guía operativa de implementación para empresas venezolanas.
> Se usa en cada proyecto: desde el levantamiento hasta la homologación con el SENIAT.

---

## Tabla de Contenido

1. [Levantamiento Previo](#1-levantamiento-previo)
2. [Perfiles de Empresa](#2-perfiles-de-empresa)
3. [Apps Odoo Nativas Requeridas](#3-apps-odoo-nativas-requeridas)
4. [Instalación por Perfil](#4-instalación-por-perfil)
5. [Configuración Post-Instalación](#5-configuración-post-instalación)
6. [Checklist de Homologación SENIAT](#6-checklist-de-homologación-seniat)
7. [Go-Live Checklist](#7-go-live-checklist)
8. [Referencia de Módulos por Capa](#8-referencia-de-módulos-por-capa)
9. [Árbol de Dependencias](#9-árbol-de-dependencias)
10. [Cobertura de la Localización](#10-cobertura-de-la-localización)
11. [Checklist de Pruebas por Módulo](#11-checklist-de-pruebas-por-módulo)

---

## 1. Levantamiento Previo

Recopilar esta información del cliente **antes** de iniciar la instalación. Determina el perfil y los módulos a instalar.

### 1.1 Datos Fiscales de la Empresa

| Dato | Valor | Impacto |
|------|-------|---------|
| RIF completo (ej. J-12345678-9) | | Configuración empresa, aparece en todos los documentos |
| Razón social exacta (igual al RIF) | | Facturas, retenciones, guías |
| Dirección fiscal (igual al SENIAT) | | Facturas, comprobantes |
| Tipo de contribuyente | Ordinario / Especial / Exento | Define si retiene IVA/ISLR |
| ¿Es agente de retención de IVA? | Sí / No | `l10n_ve_payment_extension` obligatorio |
| ¿Es agente de retención de ISLR? | Sí / No | `l10n_ve_payment_extension` obligatorio |
| ¿Retiene impuesto municipal? | Sí / No | `l10n_ve_payment_extension` + actividad económica |
| Resolución de contribuyente especial (número) | | Aparece en comprobantes de retención |
| Actividad económica principal (código y descripción) | | Retención municipal |
| Valor actual de la Unidad Tributaria (UT) | | `l10n_ve_accountant` — cálculo ISLR |

### 1.2 Operaciones en Moneda Extranjera

| Pregunta | Sí / No | Impacto |
|----------|---------|---------|
| ¿Factura en USD u otra divisa? | | `l10n_ve_rate` + `l10n_ve_igtf` obligatorios |
| ¿Recibe pagos en divisas? | | `l10n_ve_igtf` — cargo del 3% IGTF |
| ¿Realiza anticipos en divisas? | | Configuración cuentas puente IGTF |
| ¿Necesita sincronización automática BCV? | | `l10n_ve_currency_rate_live` (opcional) |
| Tasa de cambio vigente al inicio | | Configuración inicial en Odoo |

### 1.3 Emisión de Facturas

| Pregunta | Opciones | Módulo resultante |
|----------|----------|-------------------|
| ¿Cómo emite facturas actualmente? | Formato libre / Máquina fiscal / Factura digital | Define arquitectura |
| ¿Tiene máquina fiscal TFHKA? | Sí / No | `l10n_ve_iot_mf` si aplica |
| ¿Necesita facturación digital con retenciones automáticas? | Sí / No | `l10n_ve_invoice_digital` |
| Prefijo del número de control (ej. `00`) | | Configuración diario de ventas |
| ¿Maneja notas de débito? | Sí / No | Incluido en `l10n_ve_invoice` |

### 1.4 Operaciones Comerciales

| Pregunta | Sí / No | Módulos adicionales |
|----------|---------|---------------------|
| ¿Maneja inventario físico? | | `l10n_ve_stock` + `l10n_ve_stock_account` |
| ¿Emite guías de despacho? | | `l10n_ve_stock_account` obligatorio |
| ¿Tiene punto de venta (caja)? | | `l10n_ve_pos` + `l10n_ve_pos_igtf` |
| ¿El POS usa máquina fiscal? | | `l10n_ve_pos_mf` |
| ¿Realiza donaciones registrables? | | `l10n_ve_donation` |
| ¿Necesita programa de fidelización? | | `l10n_ve_invoice_loyalty` |
| ¿Maneja listas de precio en divisas? | | `l10n_ve_price_list` |

### 1.5 Requerimientos Adicionales

| Pregunta | Sí / No | Módulo |
|----------|---------|--------|
| ¿Requiere bloqueo de períodos fiscales? | | `l10n_ve_fiscal_lock_days` |
| ¿Requiere auditoría de cambios contables? | | `l10n_ve_auditlog` |
| ¿Necesita cierre formal de año fiscal? | | `l10n_ve_account_fiscalyear_closing` |
| ¿Valida referencias bancarias en pagos? | | `l10n_ve_ref_bank` |
| ¿Usa Odoo Studio para customizaciones? | | `l10n_ve_studio` obligatorio si responde sí |

---

## 2. Perfiles de Empresa

Con base en el levantamiento, clasificar al cliente en uno de estos perfiles. Los perfiles son acumulativos.

### Perfil A — Servicios, Contribuyente Ordinario
**Caso típico:** Firma de consultoría, agencia, despacho jurídico, sin inventario, sin retenciones.

Factura en bolívares o divisas, paga IVA pero no retiene, puede tener operaciones en USD con IGTF.

### Perfil B — Servicios o Comercio, Contribuyente Especial
**Caso típico:** Empresa mediana/grande designada por el SENIAT como agente de retención.

Todo lo del Perfil A, más obligación de retener IVA (75% o 100%), ISLR y/o Municipal sobre pagos a proveedores.

### Perfil C — Comercio con Inventario
**Caso típico:** Distribuidora, importadora, empresa manufacturera.

Todo lo del Perfil B, más control de inventario, guías de despacho legalmente requeridas para movilización de mercancía.

### Perfil D — Con Punto de Venta
**Caso típico:** Retail, restaurante, tienda.

Todo lo del Perfil C (o B si no hay inventario propio), más POS con manejo de múltiples métodos de pago en divisas y VES, IGTF en caja.

### Perfil E — Con Máquina Fiscal TFHKA
**Caso típico:** Empresa obligada por el SENIAT a usar máquina fiscal.

Cualquier perfil anterior, más integración IoT con máquina fiscal TFHKA para emisión de facturas y reportes Z.

---

## 3. Apps Odoo Nativas Requeridas

Instalar estas apps de Odoo **antes** de los módulos venezolanos. Sin ellas las dependencias no resuelven.

| App (nombre en UI) | Módulo técnico | Obligatoria para |
|--------------------|---------------|-----------------|
| Contabilidad | `account` + `account_accountant` | Todos los perfiles |
| Contactos | `contacts` | Todos los perfiles |
| Ventas | `sale` | Perfiles C, D, E |
| Compras | `purchase` | Perfiles C, D, E |
| Inventario | `stock` | Perfiles C, D, E |
| Punto de Venta | `point_of_sale` | Perfiles D, E |
| IoT | `iot` | Solo Perfil E |

> Al instalar **Contabilidad** en Odoo 19 con país Venezuela, el plan de cuentas oficial (`l10n_ve`) se carga automáticamente. No se requiere módulo adicional de plan de cuentas.

---

## 4. Instalación por Perfil

### Perfil A — Servicios, Contribuyente Ordinario

```
l10n_ve_base
l10n_ve_rate
l10n_ve_location
l10n_ve_contact
od_journal_sequence
l10n_ve_accountant
l10n_ve_invoice
l10n_ve_tax_payer
l10n_ve_igtf              ← solo si opera en divisas
l10n_ve_filter_partner    ← recomendado siempre
l10n_ve_suggested_amount  ← recomendado si opera en divisas
```

### Perfil B — Contribuyente Especial (acumulativo sobre A)

```
+ l10n_ve_payment_extension   ← retenciones IVA / ISLR / Municipal
+ l10n_ve_ref_bank            ← si valida referencias bancarias
+ l10n_ve_auditlog            ← recomendado para trazabilidad SENIAT
+ l10n_ve_fiscal_lock_days    ← recomendado para control de períodos
```

### Perfil C — Con Inventario (acumulativo sobre B)

```
+ l10n_ve_stock
+ l10n_ve_sale
+ l10n_ve_purchase
+ l10n_ve_stock_purchase
+ l10n_ve_donation          ← solo si realiza donaciones
+ l10n_ve_stock_account     ← guías de despacho (obligatorio si mueve mercancía)
+ l10n_ve_stock_reports     ← libro de inventario para cierre fiscal
+ l10n_ve_price_list        ← si usa listas de precio en divisas
```

### Perfil D — Con POS (acumulativo sobre B o C)

```
+ l10n_ve_pos
+ l10n_ve_pos_igtf          ← si el POS acepta pagos en divisas
```

### Perfil E — Con Máquina Fiscal TFHKA (acumulativo sobre D)

```
+ l10n_ve_iot_mf
+ l10n_ve_pos_mf            ← si la máquina está en el POS
+ l10n_ve_invoice_digital   ← si emite factura digital con retenciones automáticas
```

### Módulos opcionales (cualquier perfil)

| Módulo | Cuándo agregar |
|--------|---------------|
| `l10n_ve_currency_rate_live` | Si se quiere actualización automática de tasa BCV |
| `account_fiscal_year_closing` + `l10n_ve_account_fiscalyear_closing` | Al final del primer año fiscal |
| `l10n_ve_invoice_loyalty` | Si tienen programa de puntos/fidelización |
| `l10n_ve_studio` | Si el cliente va a usar Odoo Studio |

---

## 5. Configuración Post-Instalación

Pasos a ejecutar **en orden** tras instalar los módulos. Esta es la fase de parametrización.

### 5.1 Empresa

- [ ] **Ajustes → Empresas → [empresa]**
  - Razón social exacta (igual al RIF-J)
  - RIF con prefijo: `J-12345678-9`
  - Dirección fiscal completa (estado, municipio, parroquia)
  - Teléfono, email, logo
  - Marcar **Contribuyente Especial** si aplica
  - Número de resolución de contribuyente especial
- [ ] Verificar que el país es **Venezuela** y el plan de cuentas `l10n_ve` está aplicado

### 5.2 Tipos de Cambio

- [ ] **Contabilidad → Configuración → Divisas** — activar USD
- [ ] Crear la tasa de cambio inicial con el valor BCV del día
- [ ] Si se usa `l10n_ve_currency_rate_live`: configurar el proveedor BCV y activar el cron

### 5.3 Unidad Tributaria

- [ ] **Contabilidad → Configuración → Unidad Tributaria**
- [ ] Crear registro con el valor vigente publicado por el SENIAT
- [ ] Este valor afecta los cálculos de retención ISLR con acumulados

### 5.4 Diarios Contables

Para cada diario (Ventas, Compras, Banco USD, Banco VES, Caja):
- [ ] Configurar **secuencia** con prefijo apropiado (ej. `FAC`, `COMP`, `BNK`)
- [ ] Diario de Ventas: configurar **prefijo de número de control** (ej. `00`)
- [ ] Diarios de banco/caja en USD: marcar **Aplica IGTF** si el cliente lo requiere
- [ ] Diarios bancarios: configurar validación de referencia si se instaló `l10n_ve_ref_bank`

### 5.5 Impuestos

- [ ] Verificar que existen: IVA 16%, IVA 8%, Exento (0%), IVA Importación
- [ ] Configurar las **cuentas contables** de cada impuesto (IVA por pagar, IVA soportado)
- [ ] Si hay IGTF: configurar la **cuenta puente IGTF** en Ajustes → Contabilidad

### 5.6 Retenciones (Perfiles B, C, D, E)

- [ ] **Contabilidad → Configuración → Tipos de Retención** — verificar IVA, ISLR, Municipal
- [ ] Configurar los **conceptos de retención ISLR** (honorarios, servicios técnicos, etc.) con alícuotas
- [ ] Si retiene Municipal: cargar **Actividades Económicas** y sus alícuotas
- [ ] Configurar la **firma del representante legal** para los comprobantes ARCV
- [ ] Secuencias de numeración para cada tipo de retención

### 5.7 Contactos / Proveedores

- [ ] Cargar los proveedores principales con RIF completo y prefijo
- [ ] Marcar correctamente: **Contribuyente Especial**, **Retiene IVA**, **Retiene ISLR**, **Retiene Municipal**
- [ ] Asignar actividad económica a los proveedores que aplique retención municipal

### 5.8 Inventario (Perfiles C, D, E)

- [ ] Configurar almacenes con nombres y ubicaciones correctas
- [ ] Cargar **Razones de Transferencia** (Venta, Traslado, Donación, Muestra, etc.)
- [ ] Configurar secuencia de **Guías de Despacho**
- [ ] Asignar cuentas contables a categorías de productos

### 5.9 Punto de Venta (Perfiles D, E)

- [ ] Configurar POS: almacén, tasa de cambio por defecto, métodos de pago
- [ ] Métodos de pago: crear uno en VES y uno en USD (si aplica IGTF, marcarlo)
- [ ] Configurar impresora de tickets si aplica

### 5.10 Máquina Fiscal (Perfil E)

- [ ] **IoT → Dispositivos** — registrar máquina fiscal con IP y token TFHKA
- [ ] Mapear impuestos IVA a códigos de la máquina fiscal
- [ ] Marcar el diario de ventas como **Con Máquina Fiscal**
- [ ] Hacer prueba de conexión desde Odoo

---

## 6. Checklist de Homologación SENIAT

El SENIAT no tiene un proceso formal de "homologación de software" para todos los casos, pero sí exige que los documentos emitidos cumplan con los requisitos del Artículo 57 de la Ley del IVA y las providencias administrativas vigentes. Este checklist valida que Odoo cumple esos requisitos.

### 6.1 Requisitos de la Factura (Art. 57 Ley IVA + Providencia 0071)

Imprimir una factura de prueba y verificar que contiene:

- [ ] Denominación **"FACTURA"** claramente visible
- [ ] **Número de Control** (correlativo, ej. `No. 00-00000001`)
- [ ] **Número de factura** (secuencia del diario)
- [ ] **Fecha** de emisión
- [ ] **Razón social** del emisor exactamente igual al RIF-SENIAT
- [ ] **RIF del emisor** con prefijo (J, V, E, G)
- [ ] **Dirección fiscal** del emisor
- [ ] **Teléfono** del emisor
- [ ] **Nombre o razón social** del cliente
- [ ] **RIF del cliente**
- [ ] **Dirección** del cliente
- [ ] Descripción del bien o servicio
- [ ] Cantidad, precio unitario, precio total por línea
- [ ] **Base imponible** (monto gravado)
- [ ] **Alícuota del IVA** aplicada (16% o 8%)
- [ ] **Monto del IVA**
- [ ] **Total** de la factura
- [ ] Si opera en divisas: **tasa de cambio** y equivalente en VES
- [ ] Si aplica IGTF: línea de **IGTF 3%** separada

### 6.2 Comprobantes de Retención (Contribuyentes Especiales)

Generar un comprobante de retención de IVA y verificar:

- [ ] Denominación **"COMPROBANTE DE RETENCIÓN"**
- [ ] **Número** del comprobante (secuencia propia)
- [ ] **Fecha** de emisión
- [ ] Datos del **agente de retención** (razón social, RIF, dirección, resolución SENIAT)
- [ ] Datos del **proveedor retenido** (razón social, RIF)
- [ ] **Número de factura** retenida y fecha
- [ ] **Monto total** de la factura
- [ ] **Base imponible** retenida
- [ ] **Alícuota de retención** (75% del IVA para contribuyentes especiales)
- [ ] **Monto retenido**
- [ ] Período fiscal al que corresponde
- [ ] **Firma** del representante legal o persona autorizada (configurable en el módulo)
- [ ] Lo mismo para retención de **ISLR** si aplica

### 6.3 Guía de Despacho (si mueve mercancía)

- [ ] Denominación **"GUÍA DE DESPACHO"**
- [ ] **Número** correlativo
- [ ] Fecha de emisión
- [ ] Datos del **remitente** y **destinatario** con RIF
- [ ] Descripción de los bienes, cantidad y unidad
- [ ] **Razón de la transferencia** (venta, traslado, etc.)
- [ ] Datos del **transportista**
- [ ] Referencia a la **factura** o **orden** que la origina

### 6.4 Validación de RIF

- [ ] Todo cliente y proveedor tiene RIF cargado con prefijo correcto
- [ ] El sistema rechaza guardar un partner sin RIF si se configura como obligatorio
- [ ] Las facturas muestran el RIF del cliente correctamente

### 6.5 Control de Numeración

- [ ] Los números de control son **correlativos sin saltos** por diario
- [ ] Las notas de débito tienen su **propio número de control** separado de las facturas
- [ ] Las notas de crédito tienen su **propio número de control**
- [ ] Cada diario tiene su secuencia independiente (`od_journal_sequence`)

---

## 7. Go-Live Checklist

Verificar antes de que el cliente empiece a operar en producción.

### Datos maestros
- [ ] Empresa configurada con todos los datos fiscales
- [ ] Tasa de cambio del día cargada
- [ ] Valor de la UT actualizado
- [ ] Todos los proveedores habituales cargados con RIF y configuración de retenciones
- [ ] Productos y servicios cargados con impuestos correctos
- [ ] Cuentas bancarias de la empresa configuradas

### Secuencias
- [ ] Número de control de ventas inicia desde el número correcto (continuación del sistema anterior o desde 1)
- [ ] Secuencias de retenciones IVA / ISLR / Municipal inician correctamente
- [ ] Secuencias de guías de despacho (si aplica)

### Accesos y permisos
- [ ] Usuarios configurados con roles apropiados
- [ ] El contador tiene acceso a retenciones y cierre de períodos
- [ ] El almacenista tiene acceso a guías de despacho pero no a contabilidad (si aplica)

### Prueba final
- [ ] Crear una factura de venta completa → confirmar → imprimir → verificar formato
- [ ] Crear un pago de proveedor con retención (si contribuyente especial) → generar ARCV → imprimir
- [ ] Crear un pago en USD → verificar cargo de IGTF (si aplica)
- [ ] Hacer una venta en POS con pago mixto VES/USD (si aplica)
- [ ] Validar que los asientos contables generados son correctos

### Bloqueo de períodos
- [ ] Configurar `l10n_ve_fiscal_lock_days` para bloquear períodos anteriores al inicio de operaciones

---

## 8. Referencia de Módulos por Capa

La numeración de capas refleja el orden estricto de instalación. Un módulo no puede instalarse antes que todos los de capas anteriores.

---

### Capa 0 — Infraestructura Base

Estos dos módulos son pre-requisitos de prácticamente todo el stack. No tienen UI funcional propia — su rol es proveer extensiones técnicas y la lógica de tipos de cambio que el resto de capas consume. Deben instalarse primero, antes que cualquier otra app venezolana.

#### `l10n_ve_base`
Infraestructura técnica mínima. Extiende `ir.module.module` e `ir.ui.view` para que otros módulos de la localización puedan registrar vistas y configuraciones sin conflictos. Agrega configuraciones en `res.config.settings` comunes a toda la localización.

**Dependencias:** `base`, `web`

#### `l10n_ve_rate`
Núcleo del manejo multimoneda venezolano. Extiende `res.currency.rate` para soportar múltiples tipos de tasa (BCV oficial, paralela, etc.). Extiende `res.company` y `res.currency` con lógica de conversión específica. Todo el stack de localización lo usa para calcular equivalencias en VES/USD.

**Dependencias:** `base`, `l10n_ve_base`

> `l10n_ve_rate` va aquí porque su única dependencia venezolana es `l10n_ve_base` y es requerido por `l10n_ve_contact`, `l10n_ve_accountant`, `l10n_ve_stock` y docenas de módulos más. Intentar instalar `l10n_ve_contact` sin él falla inmediatamente.

---

### Capa 1 — Datos Geográficos y Contactos

#### `l10n_ve_location`
Base de datos geográfica completa de Venezuela: ciudades, municipios (23 estados) y parroquias. Define los modelos `res.country.city`, `res.country.municipality` y `res.country.parish`, extiende `res.partner` para enlazarlos. Los datos vienen cargados como CSV/XML en el módulo.

**Dependencias:** `base`, `contacts`

#### `l10n_ve_contact`
Extiende `res.partner` con campos venezolanos críticos:
- **`prefix_vat`**: prefijo del RIF (`V`, `E`, `J`, `G`, `P`)
- **Tipo de contribuyente**: natural, jurídico, gobierno, etc.
- Validación de RIF con lógica de dígito verificador
- Enlace a municipio y parroquia del módulo `l10n_ve_location`
- Extiende `res.company` y `res.config.settings` para información fiscal de la empresa

**Dependencias:** `base`, `contacts`, `account`, `l10n_ve_rate`, `l10n_ve_location`

#### `l10n_ve_currency_rate_live` *(opcional)*
Agrega un proveedor de tasa de cambio que consulta la API del BCV automáticamente. Depende de `currency_rate_live` (módulo OCA).

**Dependencias:** `l10n_ve_rate`, `currency_rate_live`

---

### Capa 2 — Contabilidad Core

#### `l10n_ve_accountant`
El módulo más importante de la localización. Es el motor contable venezolano. Extiende prácticamente todos los modelos de `account`:

- **`account.move`**: agrega `invoice_date_display` (fecha factura en zona horaria VE), `company_currency_rate` (tasa de cambio al momento de la transacción)
- **`account.move.line`**: cálculos en moneda local vs. extranjera
- **`account.payment`**: pagos multimoneda con conversión a VES
- **`account.payment.term`**: condiciones de pago adaptadas
- **`account.tax`**: integración con Unidad Tributaria (UT)
- **`account.journal`**: configuración de diarios venezolanos
- **`res.company`**: datos fiscales de empresa, RIF, contribuyente especial
- **`res.config.settings`**: configuraciones venezolanas centralizadas
- **`tax_unit`**: modelo propio para la **Unidad Tributaria (UT)** con su historial de valores
- **`product.template`**: precios con tasa de cambio embebida
- **`res.partner`**: crédito en moneda extranjera

Incluye reportes:
- Detalles de factura
- Reporte de todos los pagos
- Plantillas de reporte contable

**Dependencias:** `base`, `web`, `account`, `account_reports`, `purchase`, `sale`, `l10n_ve_base`, `l10n_ve_rate`, `l10n_ve_contact`, `account_invoice_pricelist`, `account_invoice_pricelist_sale`

---

### Capa 3 — Facturación y Secuencias

#### `od_journal_sequence`
Módulo de terceros que habilita **numeración independiente por diario**. En Odoo estándar todos los asientos de un tipo comparten secuencia; este módulo crea una secuencia por diario. Requerido por `l10n_ve_invoice` para el control de correlativo. Extiende `account.journal` y `account.move`.

**Dependencias:** `account`

#### `l10n_ve_invoice`
Módulo de facturación venezolana. Implementa:

- **`correlative`** (número de control): campo obligatorio en facturas, numerado por diario según SENIAT
- **`declaration_unique_of_customs`**: número de declaración aduanal para importaciones
- **`invoice_reception_date`**: fecha de recepción de factura de proveedor
- **`account_debit_note`**: integración con notas de débito venezolanas (número de control propio)
- **`account.journal`**: configuración de prefijos de numeración, tipo de impresora fiscal
- **`res.company`** y **`res.config.settings`**: configuración de información de emisión

Reportes:
- **Factura en formato libre** (formato requerido por SENIAT para impresoras no fiscales)
- Acción de reporte sobre `ir.actions.report`

**Dependencias:** `l10n_ve_rate`, `l10n_ve_base`, `l10n_ve_accountant`, `l10n_ve_contact`, `od_journal_sequence`, `account_debit_note`

---

### Capa 4 — Retenciones e Impuestos Especiales

#### `l10n_ve_tax_payer`
Define la clasificación fiscal del `res.partner`:
- Tipo de contribuyente: ordinario, especial, exento
- Retiene IVA: sí/no (y porcentaje)
- Retiene ISLR: sí/no
- Retiene Municipal: sí/no

Estos flags determinan qué retenciones aplican automáticamente en `l10n_ve_payment_extension`.

**Dependencias:** `base`, `l10n_ve_rate`, `l10n_ve_accountant`

#### `l10n_ve_payment_extension`
El módulo de retenciones. Gestiona las tres retenciones obligatorias venezolanas:

**Retención de IVA:**
- Porcentaje según tipo de contribuyente (75% para especiales, etc.)
- Generación de `account.retention` con líneas por factura
- Comprobante de retención imprimible (ARCV)

**Retención de ISLR:**
- Conceptos de retención configurables (honorarios, servicios, etc.)
- Porcentaje base imponible + alícuota
- Acumulación de honorarios para cálculo de UT
- Reporte de ARCV con firmas configurables

**Retención Municipal:**
- Actividades económicas (`economic.activity`) y ramos (`economic.branch`)
- Alícuota por actividad
- Comprobante municipal

Modelos clave:
- `account.retention` — cabecera de la retención
- `account.retention.line` — línea por factura retenida
- `account.withholding.type` — tipos de retención configurables
- `accumulated.fees` — acumulados de honorarios para UT

**Dependencias:** `base`, `account`, `l10n_ve_rate`, `l10n_ve_accountant`, `l10n_ve_invoice`, `l10n_ve_location`, `l10n_ve_contact`, `l10n_ve_tax_payer`, `product`, `stock`

#### `l10n_ve_igtf`
Maneja el **IGTF** (Impuesto a las Grandes Transacciones Financieras, 3% sobre pagos en moneda extranjera):

- **`bi_igtf`** / **`alter_bi_igtf`**: campos en `account.move` para controlar si aplica IGTF y su monto alternativo
- **`is_advance_move`**: marca un asiento como anticipo (prepago)
- Extiende `account.journal` para marcar diarios de caja/banco como sujetos a IGTF
- Extiende `account.payment` para calcular y registrar el IGTF automáticamente al confirmar
- Cuentas puente configurables por empresa
- Soporte para IGTF en pagos anticipados (adelantos a proveedores y clientes)
- Vista `invoice_free_form.xml` con línea de IGTF en factura

**Dependencias:** `base`, `l10n_ve_accountant`, `l10n_ve_invoice`, `l10n_ve_tax_payer`, `l10n_ve_base`

---

### Capa 5 — Inventario y Logística

#### `l10n_ve_filter_partner`
Módulo técnico puro. Define `filter.partner.mixin` para reutilización: filtra `res.partner` mostrando solo clientes en contexto de ventas y solo proveedores en contexto de compras.

**Dependencias:** `web`

#### `l10n_ve_stock`
Inventario base venezolano. Extiende:
- `product.category`: cuentas contables diferenciadas
- `product.template` / `product.product`: precio en moneda extranjera, equivalencia en VES
- `stock.move` / `stock.move.line`: integración con tipos de cambio
- `stock.quant`: valoración en VES
- `stock.picking`: campos venezolanos en transferencias
- `stock.warehouse` / `stock.location`: configuraciones venezolanas

Reportes: etiqueta de empaque, valoración de inventario (con conversión a VES)

**Dependencias:** `stock`, `product`, `l10n_ve_rate`, `stock_delivery`

#### `l10n_ve_stock_account`
Integración inventario ↔ contabilidad. Módulo más complejo de la cadena de suministro:

- **Guía de despacho** (`stock.picking.guide.dispatch`): documento físico legalmente requerido para movilización de mercancía, con número secuencial y campos de transportista
- **`transfer_reason`**: razón de la transferencia (venta, traslado, donación, muestra, etc.)
- **`alert`**: alertas configurables para autoconsumo y diferencias de inventario
- Integración con `account.move`: la guía de despacho genera/vincula la factura
- Cron jobs para reconciliación automática

**Dependencias:** `l10n_ve_stock`, `l10n_ve_invoice`, `l10n_ve_accountant`, `l10n_ve_sale`, `l10n_ve_donation`, `sale_stock`, `web`

#### `l10n_ve_stock_reports`
Genera el **Libro de Inventario** (obligatorio para cierres fiscales en Venezuela). Wizard que exporta movimientos y existencias en un período dado.

**Dependencias:** `stock`, `account`, `sale_stock`

#### `l10n_ve_stock_purchase`
Glue module entre `purchase_stock` y el stack venezolano. Sin modelos propios.

**Dependencias:** `purchase_stock`

---

### Capa 6 — Ventas, Compras y POS

#### `l10n_ve_sale`
Ventas venezolanas:
- `sale.order`: integración con tipos de cambio, filtrado de clientes, enlace a almacén venezolano
- `sale.order.line`: precio en moneda extranjera + equivalente en VES
- `product.pricelist.item`: reglas de precio con moneda extranjera
- Reportes de ventas con campo `invoice_date_display`
- Cron jobs para actualización de precios según tasa

**Dependencias:** `base`, `l10n_ve_base`, `sale`, `l10n_ve_rate`, `l10n_ve_contact`, `l10n_ve_invoice`, `l10n_ve_filter_partner`, `l10n_ve_stock`

#### `l10n_ve_purchase`
Compras venezolanas. Módulo liviano, principalmente configuración de seguridades y vistas.

**Dependencias:** `purchase`, `account`

#### `l10n_ve_price_list`
Complemento de listas de precio. Extiende vistas para mostrar precios en moneda extranjera con lista de precios activa.

**Dependencias:** `account`, `account_invoice_pricelist`, `l10n_ve_sale`

#### `l10n_ve_pos`
Punto de venta venezolano. Extiende prácticamente todo el stack POS:
- `pos.session`: cierre de sesión con tipos de cambio del día
- `pos.config`: configuración venezolana (almacén, contactos, monedas)
- `pos.order` / `pos.order.line`: precios multimoneda, datos venezolanos
- `pos.payment` / `pos.payment.method`: métodos de pago con equivalencia en moneda extranjera
- `res.partner` en POS: búsqueda por RIF
- Reporte de pagos por sesión

**Dependencias:** `base`, `point_of_sale`, `l10n_ve_rate`, `l10n_ve_contact`, `l10n_ve_stock`, `l10n_ve_location`, `l10n_ve_accountant`

#### `l10n_ve_pos_igtf`
Extensión del POS para IGTF. Toda la lógica en assets JS/OWL. Calcula automáticamente el 3% de IGTF en pagos con divisas.

**Dependencias:** `base`, `l10n_ve_pos`, `l10n_ve_igtf`

---

### Capa 7 — Máquinas Fiscales e IoT

#### `l10n_ve_iot_mf`
Integración con máquinas fiscales **TFHKA (The Factory HKA)** a través del sistema IoT de Odoo:
- Define `iot.device` especializado para máquinas fiscales venezolanas
- Al confirmar una factura, envía los datos a la máquina fiscal y obtiene número de secuencia fiscal
- `account.tax`: mapeo de tasas IVA a códigos de la máquina fiscal
- `account.journal`: diario marcado como "con máquina fiscal"
- Soporte para **Reporte Z** (cierre de caja fiscal diario)

**Dependencias:** `iot`, `account`, `web`, `l10n_ve_invoice`, `l10n_ve_tax_payer`, `l10n_ve_stock_account`

#### `l10n_ve_pos_mf`
Integración POS ↔ máquina fiscal:
- Al cerrar una venta, envía ticket a máquina fiscal
- Captura número de secuencia fiscal en la orden POS
- Genera **libro de ventas** con números fiscales

**Dependencias:** `point_of_sale`, `l10n_ve_pos`, `pos_iot`, `l10n_ve_iot_mf`

#### `l10n_ve_invoice_digital`
Facturación digital con retenciones automáticas (integración TFHKA):
- Retenciones generadas automáticamente al confirmar la factura
- Validación de que el almacén tenga máquina fiscal asociada

**Dependencias:** `base`, `account`, `l10n_ve_igtf`, `account_debit_note`, `l10n_ve_invoice`, `l10n_ve_iot_mf`, `l10n_ve_stock_account`, `l10n_ve_payment_extension`, `stock`

---

### Capa 8 — Módulos Opcionales / Extensiones

#### `l10n_ve_fiscal_lock_days`
Implementa bloqueo de períodos fiscales. Wizard para cambiar fecha de bloqueo con registro de auditoría.

**Dependencias:** `base`, `account_accountant`, `l10n_ve_accountant`

#### `account_fiscal_year_closing`
Módulo base genérico (OCA) para cierre de año fiscal. Wizard paso a paso con plantillas de asientos.

**Dependencias:** `account`

#### `l10n_ve_account_fiscalyear_closing`
Especialización venezolana del cierre fiscal. Templates de cierre venezolanos con integración de tipos de cambio.

**Dependencias:** `account_fiscal_year_closing`, `l10n_ve_contact`, `l10n_ve_rate`

#### `l10n_ve_ref_bank`
Validación de referencias bancarias en diarios de pago por prefijo y longitud.

**Dependencias:** `l10n_ve_invoice`

#### `l10n_ve_suggested_amount`
Calcula automáticamente el monto equivalente en VES al registrar un pago en USD.

**Dependencias:** `account`, `l10n_ve_accountant`

#### `l10n_ve_auditlog`
Auditoría de cambios en `account.move` y `account.payment` con log visible en chatter.

**Dependencias:** `l10n_ve_accountant`, `l10n_ve_payment_extension`

#### `l10n_ve_donation`
Donaciones con certificado imprimible, integración con inventario y validación de partner empresa.

**Dependencias:** `l10n_ve_accountant`, `l10n_ve_stock`, `l10n_ve_invoice`, `l10n_ve_sale`

#### `l10n_ve_invoice_loyalty`
Integración de programas de fidelización con facturación venezolana.

**Dependencias:** `l10n_ve_invoice`, `loyalty`

#### `l10n_ve_studio`
Restricciones de Odoo Studio para compatibilidad con la localización.

**Dependencias:** `l10n_ve_base`

#### `l10n_ve_currency_rate_live`
Sincronización automática de tasa de cambio BCV.

**Dependencias:** `l10n_ve_rate`, `currency_rate_live`

---

## 9. Árbol de Dependencias

```
base / web / account / account_accountant
│
├── l10n_ve_base
│   ├── l10n_ve_rate
│   │   ├── l10n_ve_location
│   │   │   └── l10n_ve_contact
│   │   │       └── l10n_ve_accountant  ◄─── núcleo contable
│   │   │           ├── od_journal_sequence
│   │   │           │   └── l10n_ve_invoice
│   │   │           │       ├── l10n_ve_tax_payer
│   │   │           │       │   ├── l10n_ve_payment_extension  ◄─ retenciones
│   │   │           │       │   └── l10n_ve_igtf               ◄─ IGTF
│   │   │           │       ├── l10n_ve_ref_bank
│   │   │           │       └── l10n_ve_invoice_loyalty
│   │   │           ├── l10n_ve_suggested_amount
│   │   │           └── l10n_ve_fiscal_lock_days
│   │   ├── l10n_ve_stock
│   │   │   ├── l10n_ve_sale
│   │   │   │   └── l10n_ve_price_list
│   │   │   ├── l10n_ve_donation
│   │   │   └── l10n_ve_stock_account   ◄─ guías de despacho
│   │   │       ├── l10n_ve_iot_mf      ◄─ máquinas fiscales
│   │   │       │   ├── l10n_ve_invoice_digital
│   │   │       │   └── l10n_ve_pos_mf
│   │   │       └── l10n_ve_stock_reports
│   │   └── l10n_ve_currency_rate_live
│   └── l10n_ve_studio
│
├── l10n_ve_filter_partner    ◄─ mixin técnico
│
├── l10n_ve_pos               ◄─ POS
│   ├── l10n_ve_pos_igtf
│   └── l10n_ve_pos_mf
│
├── l10n_ve_purchase
├── l10n_ve_stock_purchase
│
├── account_fiscal_year_closing
│   └── l10n_ve_account_fiscalyear_closing
│
└── l10n_ve_auditlog
```

---

## 10. Cobertura de la Localización

### Cubre completamente

| Área | Módulo(s) |
|------|-----------|
| Plan de cuentas venezolano | Oficial Odoo 19 (`l10n_ve` core) |
| Validación de RIF | `l10n_ve_contact` |
| Geografía (estados/municipios/parroquias) | `l10n_ve_location` |
| Tipos de cambio BCV | `l10n_ve_rate`, `l10n_ve_currency_rate_live` |
| Unidad Tributaria (UT) | `l10n_ve_accountant` |
| Factura con número de control | `l10n_ve_invoice` |
| Notas de débito venezolanas | `l10n_ve_invoice` |
| Retención de IVA | `l10n_ve_payment_extension` |
| Retención de ISLR | `l10n_ve_payment_extension` |
| Retención Municipal | `l10n_ve_payment_extension` |
| IGTF (3% transacciones financieras) | `l10n_ve_igtf`, `l10n_ve_pos_igtf` |
| Comprobantes de retención (ARCV) | `l10n_ve_payment_extension` |
| Anticipos clientes/proveedores | `l10n_ve_igtf` |
| Guía de despacho | `l10n_ve_stock_account` |
| Cierre fiscal anual | `l10n_ve_account_fiscalyear_closing` |
| Bloqueo de períodos | `l10n_ve_fiscal_lock_days` |
| Libro de inventario | `l10n_ve_stock_reports` |
| Máquinas fiscales TFHKA | `l10n_ve_iot_mf`, `l10n_ve_pos_mf` |
| POS multimoneda | `l10n_ve_pos` |
| Donaciones con certificado | `l10n_ve_donation` |
| Auditoría contable | `l10n_ve_auditlog` |

### No cubre (requiere desarrollo adicional o módulos externos)

| Área | Detalle |
|------|---------|
| **Declaración mensual de IVA (Forma 30)** | No hay exportación automática al formato SENIAT |
| **ARC en formato XML SENIAT** | Los comprobantes se generan en PDF pero no en XML |
| **Nómina venezolana** | No hay módulo de nómina local (IVSS, FAOV, LCT, utilidades, etc.) |
| **Libro de compras y ventas** | No se observó reporte de libro CV estándar SENIAT |
| **Declaración de impuesto municipal** | Existe retención pero no módulo de declaración/pago municipal |
| **Contabilidad pública / gobierno** | La localización es para empresas privadas |

---

## 11. Checklist de Pruebas por Módulo

Las pruebas están ordenadas igual que el orden de instalación.

---

### Plan de cuentas oficial Odoo 19 (`l10n_ve` core)

> Se configura durante el asistente de creación de empresa, no es un módulo separado.

- [ ] Al crear/configurar la empresa, seleccionar **Venezuela** como país
- [ ] Verificar que el plan de cuentas venezolano se aplica automáticamente (activos, pasivos, patrimonio, ingresos, gastos)
- [ ] Ir a **Contabilidad → Configuración → Impuestos** — deben existir: IVA 16%, IVA 8%, Exento, Cero
- [ ] Ir a **Contabilidad → Configuración → Diarios** — deben existir al menos: Ventas, Compras, Banco, Caja

---

### `l10n_ve_base`

> Módulo técnico — no tiene UI propia. Verificar que la instalación no produzca errores.

- [ ] El módulo instala sin errores en el log del servidor
- [ ] En **Ajustes → Técnico → Vistas** existen vistas con módulo `l10n_ve_base`
- [ ] En **Ajustes** aparece alguna sección con configuraciones de Venezuela

---

### `l10n_ve_rate`

- [ ] Ir a **Contabilidad → Configuración → Divisas** — USD debe aparecer activa
- [ ] Abrir USD y crear una tasa de cambio manual (ej. 1 USD = 36 VES)
- [ ] Verificar que el campo de tasa queda guardado correctamente
- [ ] En **Ajustes → Contabilidad** debe aparecer sección de tipo de cambio Venezuela
- [ ] Confirmar que VES (Bolívar Soberano) está disponible como divisa

---

### `l10n_ve_location`

- [ ] Ir a **Contactos → Configuración → Municipios** — debe listar los 335 municipios de Venezuela
- [ ] Ir a **Contactos → Configuración → Parroquias** — debe listar las parroquias
- [ ] Ir a **Contactos → Configuración → Ciudades** — debe listar ciudades venezolanas
- [ ] Crear un contacto y verificar que los campos Municipio/Parroquia están disponibles y filtran por estado

---

### `l10n_ve_contact`

- [ ] Crear un contacto de tipo Empresa y verificar que aparece el campo **Prefijo RIF** (V, E, J, G, P)
- [ ] Ingresar un RIF con formato correcto (ej. `J-12345678-9`) — debe aceptarlo
- [ ] Ingresar un RIF con formato incorrecto — debe mostrar error de validación
- [ ] Verificar que el campo **Tipo de Contribuyente** está disponible
- [ ] En **Ajustes → Empresa** verificar que el RIF de la empresa se puede configurar con prefijo
- [ ] Crear un contacto persona natural con prefijo `V` y uno jurídico con `J`

---

### `od_journal_sequence`

- [ ] Ir a **Contabilidad → Configuración → Diarios**, abrir el diario de Ventas
- [ ] Verificar que existe configuración de **Secuencia** con prefijo y siguiente número
- [ ] Crear una factura y confirmar — el número debe seguir la secuencia del diario
- [ ] Crear una factura en otro diario — debe tener numeración independiente
- [ ] Cambiar el prefijo de secuencia de un diario y confirmar nueva factura — debe usar el nuevo prefijo

---

### `l10n_ve_accountant`

- [ ] Ir a **Contabilidad → Configuración → Unidad Tributaria** — debe existir el modelo
- [ ] Crear un valor de UT y guardar
- [ ] Crear una factura de cliente en USD — verificar que aparece el campo **Tasa de Cambio**
- [ ] El campo `Fecha de Factura (Display)` debe mostrarse en zona horaria Venezuela
- [ ] Crear un pago en USD — verificar que calcula el equivalente en VES automáticamente
- [ ] En **Ajustes → Contabilidad** verificar las configuraciones venezolanas (contribuyente especial, RIF)

---

### `l10n_ve_invoice`

- [ ] Crear una factura de cliente, confirmarla — debe aparecer el **Número de Control** correlativo
- [ ] Crear una segunda factura — el número de control debe incrementar
- [ ] Configurar un prefijo de control diferente en el diario — nueva factura debe usarlo
- [ ] Crear una factura de proveedor — verificar campo **Fecha de Recepción de Factura**
- [ ] En una factura de importación, verificar campo **Declaración Única de Aduanas**
- [ ] Imprimir la factura con el reporte **Formato Libre** — verificar todos los campos requeridos por SENIAT (sección 6.1)
- [ ] Crear una Nota de Débito — debe generar su propio número de control

---

### `l10n_ve_tax_payer`

- [ ] Abrir un contacto proveedor — verificar campos: **Retiene IVA**, **Porcentaje IVA**, **Retiene ISLR**, **Retiene Municipal**
- [ ] Marcar un proveedor como Contribuyente Especial con retención IVA 75%
- [ ] Crear otro proveedor como ordinario sin retenciones — verificar diferencia en flags

---

### `l10n_ve_payment_extension`

**Retención IVA:**
- [ ] Configurar proveedor como Contribuyente Especial (retiene IVA 75%)
- [ ] Crear factura de ese proveedor con IVA → al pagar, crear la retención
- [ ] Verificar cálculo: base IVA × 75% = monto retenido
- [ ] Imprimir el **ARCV de IVA** — verificar campos requeridos (sección 6.2)
- [ ] Confirmar la retención y verificar asiento contable

**Retención ISLR:**
- [ ] Ir a **Contabilidad → Configuración → Conceptos de Retención ISLR** — deben existir conceptos
- [ ] Crear retención ISLR sobre factura de servicio profesional
- [ ] Verificar cálculo según alícuota del concepto
- [ ] Imprimir ARCV de ISLR

**Retención Municipal:**
- [ ] Ir a **Contabilidad → Configuración → Actividades Económicas** — verificar que existen
- [ ] Asignar actividad al proveedor, crear retención municipal y verificar cálculo

---

### `l10n_ve_igtf`

- [ ] En el diario de Banco USD marcar **Aplica IGTF**
- [ ] Configurar la **Cuenta Puente IGTF** en Ajustes
- [ ] Crear factura en USD y registrar pago por ese diario — verificar cargo automático del 3%
- [ ] Revisar asiento contable del pago — debe tener línea adicional de IGTF
- [ ] Crear anticipo de cliente en USD — debe marcarse como `is_advance_move`
- [ ] En la factura impresa verificar que aparece la línea de IGTF

---

### `l10n_ve_filter_partner`

- [ ] Al crear factura de cliente, el campo Cliente solo muestra contactos marcados como clientes
- [ ] Al crear factura de proveedor, el campo Proveedor solo muestra proveedores

---

### `l10n_ve_stock`

- [ ] Verificar que los productos tienen campo de **Precio en Moneda Extranjera**
- [ ] Ingresar precio en USD — verificar equivalente en VES según tasa
- [ ] Ir a **Inventario → Reportes → Valoración** — debe mostrar valores en VES
- [ ] Crear transferencia interna — verificar campos venezolanos presentes

---

### `l10n_ve_sale`

- [ ] Crear orden de venta — el selector de cliente usa filtro venezolano
- [ ] Agregar línea con producto en USD — verificar precio en USD y equivalente VES
- [ ] Confirmar orden y crear factura — verificar que hereda número de control y datos venezolanos
- [ ] Imprimir reporte de la orden

---

### `l10n_ve_purchase`

- [ ] Crear orden de compra y confirmarla
- [ ] Crear factura de proveedor desde la orden — verificar fecha de recepción

---

### `l10n_ve_stock_purchase`

- [ ] Confirmar orden de compra → debe crear recibo en almacén automáticamente
- [ ] Validar recibo → movimiento usa campos venezolanos

---

### `l10n_ve_stock_account`

- [ ] Ir a **Inventario → Configuración → Razones de Transferencia** — deben existir razones
- [ ] Crear transferencia, asignarle una razón, validarla — debe generarse **Guía de Despacho**
- [ ] Abrir la guía: verificar número secuencial, campos de transportista, origen y destino
- [ ] Imprimir la guía — verificar campos requeridos (sección 6.3)
- [ ] Verificar que la factura generada tiene número de control correcto

---

### `l10n_ve_stock_reports`

- [ ] Ir a **Inventario → Reportes → Libro de Inventario**
- [ ] Seleccionar un período y ejecutar — debe mostrar movimientos con saldo por producto

---

### `l10n_ve_pos`

- [ ] Configurar POS con tasa de cambio del día
- [ ] Abrir sesión de POS
- [ ] Buscar un cliente por **RIF** desde el POS
- [ ] Crear venta con pago en USD — verificar equivalente en VES
- [ ] Cerrar sesión — verificar resumen por tipo de pago y moneda
- [ ] Generar **Reporte de Pagos** de la sesión

---

### `l10n_ve_pos_igtf`

- [ ] Crear venta en POS con pago en USD — debe aparecer cálculo del IGTF (3%) automáticamente
- [ ] Verificar que el total cobrado incluye el IGTF
- [ ] En la orden cerrada, el IGTF debe estar registrado como línea separada

---

### `l10n_ve_stock_reports`

- [ ] Ejecutar el **Libro de Inventario** para un período
- [ ] Verificar que incluye movimientos de entrada/salida y saldo final en VES

---

### `account_fiscal_year_closing`

- [ ] Ir a **Contabilidad → Contabilidad → Cierres de Año Fiscal**
- [ ] Verificar que existen plantillas de cierre precargadas
- [ ] Crear cierre para el período anterior y ejecutar el paso "Preparar"

---

### `l10n_ve_account_fiscalyear_closing`

- [ ] Verificar campos venezolanos en el cierre (RIF empresa, tasa de cambio de cierre)
- [ ] Ejecutar cierre completo (preparar → confirmar → validar)
- [ ] Verificar que los asientos generados tienen la tasa de cambio correcta

---

### `l10n_ve_fiscal_lock_days`

- [ ] Configurar fecha de bloqueo de facturas en Ajustes
- [ ] Intentar crear factura con fecha anterior al bloqueo — debe mostrar error
- [ ] Cambiar fecha de bloqueo con el wizard — debe requerir justificación

---

### `l10n_ve_ref_bank`

- [ ] Configurar diario bancario con prefijo y longitud de referencia
- [ ] Registrar pago con referencia válida — debe aceptarla
- [ ] Registrar pago con referencia de longitud incorrecta — debe mostrar error

---

### `l10n_ve_suggested_amount`

- [ ] Crear factura en USD con saldo pendiente
- [ ] Ir a **Registrar Pago** — verificar campo **Monto Sugerido** en VES
- [ ] Cambiar la tasa — el monto sugerido debe recalcularse
- [ ] Confirmar el pago — la factura debe quedar saldada

---

### `l10n_ve_auditlog`

- [ ] Crear y confirmar una factura, modificar un campo editable
- [ ] Verificar que el chatter muestra el tracking del cambio con valor anterior y nuevo
- [ ] Verificar que los cambios en `account.payment` también quedan registrados

---

### `l10n_ve_donation`

- [ ] Verificar configuración de cuenta contable de donación en Ajustes
- [ ] Crear orden de venta marcada como **Donación** — solo debe aceptar clientes empresa
- [ ] Confirmar orden, validar despacho, crear factura
- [ ] Imprimir **Certificado de Donación** — verificar campos

---

### `l10n_ve_price_list`

- [ ] Crear lista de precio en USD y asignarla a un cliente
- [ ] Crear orden de venta para ese cliente — precios en USD correctamente mostrados
- [ ] Verificar que la factura generada refleja la lista de precio

---

### `l10n_ve_currency_rate_live` *(opcional)*

- [ ] Ir a **Contabilidad → Configuración → Divisas**
- [ ] Hacer clic en **Actualizar Tasas** — debe consultar el BCV y actualizar USD
- [ ] Verificar que la tasa actualizada es razonable (tasa BCV oficial del día)
- [ ] Configurar el cron de actualización automática

---

### `l10n_ve_iot_mf` *(requiere hardware TFHKA)*

- [ ] Configurar IP y token de máquina fiscal en Ajustes
- [ ] Verificar que aparece como dispositivo IoT
- [ ] Marcar el diario de ventas como **Con Máquina Fiscal**
- [ ] Mapear impuestos IVA a códigos de la máquina
- [ ] Confirmar una factura — debe recibir número de secuencia fiscal
- [ ] Ejecutar **Reporte Z** al final del día

---

### `l10n_ve_invoice_digital` *(requiere `l10n_ve_iot_mf`)*

- [ ] Activar **Retenciones Automáticas** en Ajustes
- [ ] Confirmar factura para proveedor con retención configurada — retenciones deben generarse automáticamente
- [ ] Verificar que el almacén tiene máquina fiscal asociada (si no, debe aparecer alerta)

---

### `l10n_ve_pos_mf` *(requiere `l10n_ve_iot_mf` + `l10n_ve_pos`)*

- [ ] Asignar máquina fiscal al POS en configuración
- [ ] Realizar venta — al cerrar debe enviar a máquina fiscal y recibir número fiscal
- [ ] Verificar número de secuencia fiscal en la orden POS
- [ ] Ir a **Reportes → Libro de Ventas Fiscal** — debe mostrar ventas con números fiscales

---

### `l10n_ve_invoice_loyalty` *(opcional)*

- [ ] Crear programa de fidelización con puntos
- [ ] Asignar programa a un cliente
- [ ] Crear factura para ese cliente — debe aparecer sección para aplicar rewards
- [ ] Aplicar reward y confirmar — debe registrarse en el programa

---

*Documento de implementación generado desde análisis de código fuente del repositorio `odoo-venezuela` rama `19.0`.  
Actualizar en cada cambio significativo de módulos o regulaciones.*
