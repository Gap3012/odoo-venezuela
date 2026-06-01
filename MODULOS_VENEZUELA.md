# Módulos Odoo Venezuela — Mapeo Completo

**Repositorio:** odoo-venezuela (rama 19.0)  
**Fecha de análisis:** 2026-06-01  
**Total de módulos:** 35

---

## Tabla de Contenido

1. [Ecosistema General](#1-ecosistema-general)
2. [Orden de Instalación Recomendado](#2-orden-de-instalación-recomendado)
3. [Módulos por Capa](#3-módulos-por-capa)
   - [Capa 0 — Infraestructura Base](#capa-0--infraestructura-base)
   - [Capa 1 — Datos Geográficos y Contactos](#capa-1--datos-geográficos-y-contactos)
   - [Capa 2 — Plan de Cuentas](#capa-2--plan-de-cuentas)
   - [Capa 3 — Contabilidad Core](#capa-3--contabilidad-core)
   - [Capa 4 — Facturación y Secuencias](#capa-4--facturación-y-secuencias)
   - [Capa 5 — Retenciones e Impuestos Especiales](#capa-5--retenciones-e-impuestos-especiales)
   - [Capa 6 — Inventario y Logística](#capa-6--inventario-y-logística)
   - [Capa 7 — Ventas, Compras y POS](#capa-7--ventas-compras-y-pos)
   - [Capa 8 — Máquinas Fiscales e IoT](#capa-8--máquinas-fiscales-e-iot)
   - [Capa 9 — Módulos Opcionales / Extensiones](#capa-9--módulos-opcionales--extensiones)
4. [Fichas Detalladas de Cada Módulo](#4-fichas-detalladas-de-cada-módulo)
5. [Árbol de Dependencias](#5-árbol-de-dependencias)
6. [Qué cubre y qué no cubre esta localización](#6-qué-cubre-y-qué-no-cubre-esta-localización)
7. [Checklist de Pruebas por Módulo](#7-checklist-de-pruebas-por-módulo)

---

## 1. Ecosistema General

Esta localización es una implementación **completa y profunda** de Odoo para Venezuela. No es un simple plan de cuentas: abarca desde la validación de RIF hasta la integración con máquinas fiscales TFHKA (The Factory HKA), pasando por retenciones de ISLR/IVA, IGTF, guías de remisión y cierre fiscal.

### Responsables técnicos
Los módulos con prefijo `l10n_ve_*` son desarrollados por **Binaural Dev**, mientras que `od_journal_sequence` es un módulo de terceros incorporado al stack.

### Arquitectura en capas

```
[Odoo Core]
    ↓
[l10n_ve_base]  ←  infraestructura técnica
    ↓
[l10n_ve_rate]  ←  tipos de cambio
    ↓
[l10n_ve_location] + [l10n_ve_contact]  ←  geografía + RIF
    ↓
[l10n_binaural / l10n_ve_binaural]  ←  plan de cuentas
    ↓
[l10n_ve_accountant]  ←  contabilidad core
    ↓
[l10n_ve_invoice]  ←  facturación SENIAT
    ↓
[l10n_ve_payment_extension]  ←  retenciones
[l10n_ve_igtf]               ←  IGTF
    ↓
[l10n_ve_stock] → [l10n_ve_stock_account]  ←  inventario
    ↓
[l10n_ve_sale] + [l10n_ve_purchase]  ←  ventas / compras
    ↓
[l10n_ve_pos] + extensiones  ←  punto de venta
[l10n_ve_iot_mf]             ←  máquinas fiscales
```

---

## 2. Orden de Instalación Recomendado

El orden está determinado por el grafo de dependencias de los `__manifest__.py`. Se listan en grupos que pueden instalarse en el orden dado.

### Instalación mínima (solo contabilidad básica)

| # | Módulo | Razón |
|---|--------|-------|
| 1 | `l10n_ve_base` | Base técnica de toda la localización |
| 2 | `l10n_ve_rate` | Tipos de cambio (requerido por casi todo) |
| 3 | `l10n_ve_location` | Datos geográficos de Venezuela |
| 4 | `l10n_ve_contact` | RIF, tipos de contribuyente, municipio |
| 5 | `l10n_binaural` | Plan de cuentas para empresas de servicio |
| 6 | `od_journal_sequence` | Numeración por diario (requerido por facturación) |
| 7 | `l10n_ve_accountant` | Motor contable venezolano + Unidad Tributaria |
| 8 | `l10n_ve_invoice` | Facturación, número de control, SENIAT |
| 9 | `l10n_ve_tax_payer` | Clasificación de contribuyentes |
| 10 | `l10n_ve_payment_extension` | Retenciones IVA / ISLR / Municipal |
| 11 | `l10n_ve_igtf` | IGTF (3% grandes transacciones financieras) |

### Instalación con inventario

Continuar desde la mínima y agregar:

| # | Módulo | Razón |
|---|--------|-------|
| 12 | `l10n_ve_filter_partner` | Mixin técnico para filtros de clientes |
| 13 | `l10n_ve_stock` | Inventario venezolano |
| 14 | `l10n_ve_sale` | Ventas venezolanas |
| 15 | `l10n_ve_purchase` | Compras venezolanas |
| 16 | `l10n_ve_stock_purchase` | Integración compras ↔ inventario |
| 17 | `l10n_ve_donation` | Donaciones (si aplica) |
| 18 | `l10n_ve_stock_account` | Integración inventario ↔ contabilidad, guías de remisión |

### Instalación completa (con POS y reportes)

Continuar desde inventario y agregar:

| # | Módulo | Razón |
|---|--------|-------|
| 19 | `l10n_ve_pos` | Punto de venta |
| 20 | `l10n_ve_pos_igtf` | IGTF en POS |
| 21 | `l10n_ve_stock_reports` | Libro de inventario |
| 22 | `account_fiscal_year_closing` | Base genérica de cierre fiscal |
| 23 | `l10n_ve_account_fiscalyear_closing` | Cierre fiscal venezolano |
| 24 | `l10n_ve_fiscal_lock_days` | Bloqueo de períodos |
| 25 | `l10n_ve_ref_bank` | Validación de referencias bancarias |
| 26 | `l10n_ve_suggested_amount` | Monto sugerido en pagos multimoneda |
| 27 | `l10n_ve_auditlog` | Auditoría de cambios |
| 28 | `l10n_ve_price_list` | Listas de precio multimoneda |

### Módulos opcionales (solo si se necesita)

| Módulo | Cuándo instalar |
|--------|----------------|
| `l10n_ve_iot_mf` | Si se tiene máquina fiscal TFHKA |
| `l10n_ve_invoice_digital` | Si se emite factura digital con retenciones automáticas |
| `l10n_ve_pos_mf` | Si el POS está conectado a máquina fiscal |
| `l10n_ve_currency_rate_live` | Si se quiere sincronización automática BCV |
| `l10n_ve_invoice_loyalty` | Si se tiene programa de fidelización |
| `l10n_ve_studio` | Si se usa Odoo Studio (limita customizaciones) |
| `l10n_ve_binaural` | Alternativa al plan de cuentas `l10n_binaural` |

> **Nota:** `l10n_binaural` y `l10n_ve_binaural` son mutuamente excluyentes — ambos proveen plan de cuentas. `l10n_binaural` es el plan completo para empresas de servicios; `l10n_ve_binaural` es el oficial de Odoo adaptado.

---

## 3. Módulos por Capa

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

---

### Capa 2 — Plan de Cuentas

#### `l10n_binaural`
Plan de cuentas completo para **empresas de servicio venezolanas**. Contiene:
- Árbol de cuentas contables (activos, pasivos, patrimonio, ingresos, gastos)
- Diarios contables preconfigurados
- Impuestos IVA (16%, exento, cero) y retenciones básicas
- `product.template` para productos de servicio
- Datos de configuración inicial

**Dependencias:** `base`, `account`, `account_accountant`, `stock`, `sale`, `contacts`

#### `l10n_ve_binaural`
Plan de cuentas alternativo. Define el modelo `template_ve.py` con la plantilla oficial de Odoo para Venezuela, más datos de demostración en `demo_company.xml`. Es más liviano que `l10n_binaural` y útil para empresas que parten de cero con Odoo.

**Dependencias:** `account`

> `l10n_binaural` y `l10n_ve_binaural` son mutuamente excluyentes — instalar solo uno.

#### `l10n_ve_currency_rate_live` *(opcional en esta capa)*
Agrega un proveedor de tasa de cambio que consulta la API del BCV automáticamente. Depende de `currency_rate_live` (módulo OCA).

**Dependencias:** `l10n_ve_rate`, `currency_rate_live`

---

### Capa 3 — Contabilidad Core

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

---

### Capa 4 — Facturación y Secuencias

#### `od_journal_sequence`
Módulo de terceros que habilita **numeración independiente por diario**. En Odoo estándar todos los asientos de un tipo comparten secuencia; este módulo crea una secuencia por diario. Requerido por `l10n_ve_invoice` para el control de correlativo. Extiende `account.journal` y `account.move`.

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

---

### Capa 5 — Retenciones e Impuestos Especiales

#### `l10n_ve_tax_payer`
Define la clasificación fiscal del `res.partner`:
- Tipo de contribuyente: ordinario, especial, exento
- Retiene IVA: sí/no (y porcentaje)
- Retiene ISLR: sí/no
- Retiene Municipal: sí/no
Estos flags determinan qué retenciones aplican automáticamente en `l10n_ve_payment_extension`.

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

#### `l10n_ve_igtf`
Maneja el **IGTF** (Impuesto a las Grandes Transacciones Financieras, 3% sobre pagos en moneda extranjera):

- **`bi_igtf`** / **`alter_bi_igtf`**: campos en `account.move` para controlar si aplica IGTF y su monto alternativo
- **`is_advance_move`**: marca un asiento como anticipo (prepago)
- Extiende `account.journal` para marcar diarios de caja/banco como sujetos a IGTF
- Extiende `account.payment` para calcular y registrar el IGTF automáticamente al confirmar
- Cuentas puente configurables por empresa
- Soporte para IGTF en pagos anticipados (adelantos a proveedores y clientes)
- Vista `invoice_free_form.xml` con línea de IGTF en factura

---

### Capa 6 — Inventario y Logística

#### `l10n_ve_stock`
Inventario base venezolano. Extiende:
- `product.category`: cuentas contables diferenciadas
- `product.template` / `product.product`: precio en moneda extranjera, equivalencia en VES
- `stock.move` / `stock.move.line`: integración con tipos de cambio
- `stock.quant`: valoración en VES
- `stock.picking`: campos venezolanos en transferencias
- `stock.warehouse` / `stock.location`: configuraciones venezolanas

Reportes:
- Etiqueta de empaque
- Valoración de inventario (con conversión a VES)

#### `l10n_ve_stock_account`
Integración inventario ↔ contabilidad. Es el módulo más complejo de la cadena de suministro:

- **Guía de despacho** (`stock.picking.guide.dispatch`): documento físico requerido legalmente para movilización de mercancía en Venezuela, con número secuencial y campos de transportista
- **`transfer_reason`**: razón de la transferencia (venta, traslado, donación, muestra, etc.)
- **`alert`**: alertas configurables para autoconsumo y diferencias de inventario
- Integración con `account.move`: la guía de despacho genera/vincula la factura
- Integración con donaciones
- Cron jobs para reconciliación automática

#### `l10n_ve_stock_reports`
Genera el **Libro de Inventario** (obligatorio para cierres fiscales en Venezuela). Wizard que exporta movimientos y existencias en un período dado, con formato apto para presentar ante el SENIAT.

#### `l10n_ve_stock_purchase`
Glue module entre `purchase_stock` (Odoo) y el stack venezolano. Sin modelos propios, solo seguridades y configuraciones para que compras y almacén venezolano operen juntos correctamente.

---

### Capa 7 — Ventas, Compras y POS

#### `l10n_ve_filter_partner`
Módulo técnico puro. Define `filter.partner.mixin` para reutilización: filtra `res.partner` mostrando solo clientes en contexto de ventas y solo proveedores en contexto de compras. Evita código duplicado en `l10n_ve_sale`, `l10n_ve_invoice`, etc.

#### `l10n_ve_sale`
Ventas venezolanas:
- `sale.order`: integración con tipos de cambio, filtrado de clientes, enlace a almacén venezolano
- `sale.order.line`: precio en moneda extranjera + equivalente en VES
- `product.pricelist.item`: reglas de precio con moneda extranjera
- Reportes de ventas con campo `invoice_date_display`
- Cron jobs para actualización de precios según tasa

#### `l10n_ve_purchase`
Compras venezolanas. Módulo liviano (sin modelos propios actualmente), principalmente configuración de seguridades y vistas para el proceso de compras local.

#### `l10n_ve_price_list`
Complemento de listas de precio. Extiende las vistas de `account.invoice`, `sale.order` y `product.template` para mostrar correctamente precios en moneda extranjera cuando hay lista de precios activa.

#### `l10n_ve_pos`
Punto de venta venezolano. Extiende prácticamente todo el stack POS:
- `pos.session`: cierre de sesión con tipos de cambio del día
- `pos.config`: configuración venezolana (almacén, contactos, monedas)
- `pos.order` / `pos.order.line`: precios multimoneda, datos venezolanos
- `pos.payment` / `pos.payment.method`: métodos de pago con equivalencia en moneda extranjera
- `res.partner` en POS: búsqueda por RIF
- `account.move`: factura generada desde POS con campos venezolanos
- Reporte de pagos por sesión

#### `l10n_ve_pos_igtf`
Extensión del POS para IGTF. Sin modelos propios (toda la lógica está en assets JS/OWL). Extiende las vistas del POS para mostrar y calcular automáticamente el 3% de IGTF en pagos con divisas o transferencias.

---

### Capa 8 — Máquinas Fiscales e IoT

#### `l10n_ve_iot_mf`
Integración con máquinas fiscales **TFHKA (The Factory HKA)** a través del sistema IoT de Odoo:
- Define `iot.device` especializado para máquinas fiscales venezolanas
- `iot.box`: caja IoT con configuración de puerto/IP
- `account.move`: al confirmar una factura, envía los datos a la máquina fiscal y obtiene número de secuencia fiscal
- `account.tax`: mapeo de tasas IVA a códigos de la máquina fiscal
- `account.journal`: diario marcado como "con máquina fiscal"
- `res.company` / `res.config.settings`: IP y token de la máquina fiscal
- Soporte para **Reporte Z** (cierre de caja fiscal diario)

#### `l10n_ve_pos_mf`
Integración POS ↔ máquina fiscal. Combina `l10n_ve_pos` con `l10n_ve_iot_mf`:
- `pos.order`: al cerrar una venta, envía ticket a máquina fiscal
- Captura número de secuencia fiscal en la orden POS
- Genera **libro de ventas** con números fiscales
- Vistas de configuración para enlazar POS con máquina fiscal específica

#### `l10n_ve_invoice_digital`
Facturación digital con retenciones automáticas (integración TFHKA):
- `account.retention` extendido para generación digital automática
- `stock.picking` extendido para validar que el almacén tenga máquina fiscal asociada antes de facturar
- `res.config.settings`: activar/desactivar retenciones automáticas
- Alertas de retención en proceso de facturación

---

### Capa 9 — Módulos Opcionales / Extensiones

#### `l10n_ve_fiscal_lock_days`
Implementa bloqueo de períodos fiscales:
- `res.company`: campos `fiscalyear_lock_date_invoice` y similares específicos para Venezuela
- `account.move`: validación al crear/modificar asientos en período bloqueado
- `account.change.lock.date`: wizard para cambiar la fecha de bloqueo con registro de auditoría
- `res.config.settings`: UI de configuración de bloqueo

#### `account_fiscal_year_closing`
Módulo base genérico (OCA) para cierre de año fiscal:
- `account.fiscalyear.closing`: encabezado del proceso de cierre
- `account.fiscalyear.closing.template`: plantillas de asientos de cierre (apertura de balance, traslado de resultados, etc.)
- Wizard paso a paso: preparar → revisar → confirmar → validar

#### `l10n_ve_account_fiscalyear_closing`
Especialización venezolana del cierre fiscal:
- Extiende los modelos OCA con campos venezolanos (tipo de cambio, RIF de empresa)
- Templates de cierre precargados según regulaciones venezolanas
- Integración con `l10n_ve_rate` para conversión de saldos

#### `l10n_ve_ref_bank`
Validación de referencias bancarias en diarios de pago:
- Campos para número de referencia bancaria con validación de formato
- Configuración por diario (banco) de prefijos y longitudes aceptadas
- Evita registrar pagos con referencias mal formateadas

#### `l10n_ve_suggested_amount`
Mejora UX para pagos multimoneda:
- Extiende `account.payment.register` con campo `suggested_amount`
- Calcula automáticamente cuánto pagar en VES equivalente al saldo en USD según tasa vigente
- Vista del wizard extendida con el campo sugerido

#### `l10n_ve_auditlog`
Auditoría de cambios:
- Extiende `mail.tracking.value` para registrar cambios en campos críticos de `account.move` y `account.payment`
- Vistas de log de auditoría filtradas por documento
- Útil para trazabilidad ante el SENIAT

#### `l10n_ve_donation`
Gestión de donaciones caritativas/sociales:
- `account.move`: campo `is_donation` que marca el asiento
- Validación de que las donaciones solo usen partners empresa (no personas naturales)
- `sale.order`: ventas marcadas como donación
- `stock.picking` / `stock.move`: movimientos de bienes donados con contabilización automática
- `stock.scrap`: bajas de inventario por donación
- Reporte **Certificado de Donación** imprimible

#### `l10n_ve_invoice_loyalty`
Integración del módulo nativo `loyalty` de Odoo con facturación venezolana:
- Extiende `account.move` para aplicar rewards/puntos de fidelización en facturas
- Módulo liviano que actúa como glue entre `loyalty` y `l10n_ve_invoice`

#### `l10n_ve_studio`
Control de Odoo Studio para la localización:
- Hook `post_init_hook` que desactiva o restringe funcionalidades de Studio que entrarían en conflicto con las vistas y modelos de la localización
- Solo instalar si se va a usar Odoo Studio en un entorno venezolano

---

## 4. Fichas Detalladas de Cada Módulo

| Módulo | Descripción funcional | Dependencias directas |
|--------|----------------------|----------------------|
| `l10n_ve_base` | Infraestructura técnica común de la localización. Extiende vistas y módulos base. | `base`, `web` |
| `l10n_ve_rate` | Tipos de cambio venezolanos (BCV oficial, paralela). Conversiones VES/USD. | `base`, `l10n_ve_base` |
| `l10n_ve_location` | Ciudades, municipios y parroquias de Venezuela (23 estados). | `base`, `contacts` |
| `l10n_ve_contact` | Validación RIF, prefijos (V/E/J/G), tipo contribuyente, domicilio fiscal. | `base`, `contacts`, `account`, `l10n_ve_rate`, `l10n_ve_location` |
| `l10n_binaural` | Plan de cuentas completo para empresas de servicio + impuestos + diarios. | `base`, `account`, `account_accountant`, `stock`, `sale`, `contacts` |
| `l10n_ve_binaural` | Plan de cuentas alternativo (template oficial Odoo Venezuela). | `account` |
| `od_journal_sequence` | Secuencias numéricas independientes por diario contable. | `account` |
| `l10n_ve_accountant` | Motor contable venezolano: UT, multimoneda, pagos, reportes. | `base`, `web`, `account`, `account_reports`, `purchase`, `sale`, `l10n_ve_base`, `l10n_ve_rate`, `l10n_ve_contact`, `account_invoice_pricelist`, `account_invoice_pricelist_sale` |
| `l10n_ve_invoice` | Número de control, correlativo, referencia aduanal, factura libre SENIAT. | `l10n_ve_rate`, `l10n_ve_base`, `l10n_ve_accountant`, `l10n_ve_contact`, `od_journal_sequence`, `account_debit_note` |
| `l10n_ve_tax_payer` | Tipo contribuyente en partner: retiene IVA/ISLR/Municipal. | `base`, `l10n_ve_rate`, `l10n_ve_accountant` |
| `l10n_ve_payment_extension` | Retenciones IVA, ISLR y Municipal. ARCV. Comprobantes. | `base`, `account`, `l10n_ve_rate`, `l10n_ve_accountant`, `l10n_ve_invoice`, `l10n_ve_location`, `l10n_ve_contact`, `l10n_ve_tax_payer`, `product`, `stock` |
| `l10n_ve_igtf` | IGTF 3% en pagos en divisas. Anticipos con cuentas puente. | `base`, `l10n_ve_accountant`, `l10n_ve_invoice`, `l10n_ve_tax_payer`, `l10n_ve_base` |
| `l10n_ve_filter_partner` | Mixin técnico: filtro clientes/proveedores en formularios. | `web` |
| `l10n_ve_stock` | Inventario venezolano: valoración VES, precios en divisas, transferencias. | `stock`, `product`, `l10n_ve_rate`, `stock_delivery` |
| `l10n_ve_sale` | Ventas con tipos de cambio, precios en divisas, reportes. | `base`, `l10n_ve_base`, `sale`, `l10n_ve_rate`, `l10n_ve_contact`, `l10n_ve_invoice`, `l10n_ve_filter_partner`, `l10n_ve_stock` |
| `l10n_ve_purchase` | Compras venezolanas (liviano, principalmente configuraciones). | `purchase`, `account` |
| `l10n_ve_stock_purchase` | Glue: compras ↔ inventario venezolano. | `purchase_stock` |
| `l10n_ve_donation` | Donaciones con certificado imprimible e integración inventario. | `l10n_ve_accountant`, `l10n_ve_stock`, `l10n_ve_invoice`, `l10n_ve_sale` |
| `l10n_ve_stock_account` | Guías de despacho, autoconsumo, integración inventario/facturación. | `l10n_ve_stock`, `l10n_ve_invoice`, `l10n_ve_accountant`, `l10n_ve_sale`, `l10n_ve_donation`, `sale_stock`, `web` |
| `l10n_ve_stock_reports` | Libro de inventario para cierre fiscal. | `stock`, `account`, `sale_stock` |
| `l10n_ve_pos` | POS venezolano multimoneda, tipos de cambio, RIF en POS. | `base`, `point_of_sale`, `l10n_ve_rate`, `l10n_ve_contact`, `l10n_ve_stock`, `l10n_ve_location`, `l10n_ve_accountant` |
| `l10n_ve_pos_igtf` | IGTF en POS para pagos en divisas. | `base`, `l10n_ve_pos`, `l10n_ve_igtf` |
| `l10n_ve_iot_mf` | Máquinas fiscales TFHKA: IoT, envío de facturas, Reporte Z. | `iot`, `account`, `web`, `l10n_ve_invoice`, `l10n_ve_tax_payer`, `l10n_ve_stock_account` |
| `l10n_ve_pos_mf` | POS ↔ máquina fiscal: libro de ventas fiscal. | `point_of_sale`, `l10n_ve_pos`, `pos_iot`, `l10n_ve_iot_mf` |
| `l10n_ve_invoice_digital` | Factura digital con retenciones automáticas (TFHKA). | `base`, `account`, `l10n_ve_igtf`, `account_debit_note`, `l10n_ve_invoice`, `l10n_ve_iot_mf`, `l10n_ve_stock_account`, `l10n_ve_payment_extension`, `stock` |
| `l10n_ve_price_list` | Listas de precio multimoneda, vistas extendidas. | `account`, `account_invoice_pricelist`, `l10n_ve_sale` |
| `l10n_ve_ref_bank` | Validación de referencias bancarias en diarios. | `l10n_ve_invoice` |
| `l10n_ve_suggested_amount` | Monto sugerido en registro de pago multimoneda. | `account`, `l10n_ve_accountant` |
| `l10n_ve_fiscal_lock_days` | Bloqueo de períodos fiscales por fecha. | `base`, `account_accountant`, `l10n_ve_accountant` |
| `account_fiscal_year_closing` | Base genérica OCA para cierre de año fiscal. | `account` |
| `l10n_ve_account_fiscalyear_closing` | Cierre fiscal venezolano con tipos de cambio y RIF. | `account_fiscal_year_closing`, `l10n_ve_contact`, `l10n_ve_rate` |
| `l10n_ve_auditlog` | Auditoría de cambios en movimientos y pagos. | `l10n_ve_accountant`, `l10n_ve_payment_extension` |
| `l10n_ve_invoice_loyalty` | Fidelización en facturación venezolana. | `l10n_ve_invoice`, `loyalty` |
| `l10n_ve_studio` | Restricciones de Studio para la localización. | `l10n_ve_base` |
| `l10n_ve_currency_rate_live` | Sincronización automática tasa BCV. | `l10n_ve_rate`, `currency_rate_live` |

---

## 5. Árbol de Dependencias

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
├── l10n_binaural             ◄─ plan de cuentas (servicio)
├── l10n_ve_binaural          ◄─ plan de cuentas (template Odoo)
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

## 6. Qué cubre y qué no cubre esta localización

### Cubre completamente

| Área | Módulos |
|------|---------|
| Plan de cuentas venezolano | `l10n_binaural`, `l10n_ve_binaural` |
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
| **SENIAT EDI / factura electrónica XML** | La "factura digital" implementada aquí es vía máquina fiscal TFHKA, no el formato XML del SENIAT (si aplica) |
| **Declaración mensual de IVA (Forma 30)** | No hay exportación automática al formato del SENIAT |
| **ARC (Agentes de Retención de ISLR)** | Los comprobantes se generan pero no se exportan en formato XML SENIAT |
| **Contabilidad pública / gobierno** | La localización es para empresas privadas |
| **Nómina venezolana** | No hay módulo de nómina local (IVSS, FAOV, LCT, etc.) |
| **Libro de compras y ventas (BCV/SENIAT)** | No se observó reporte de libro de compras/ventas estándar |
| **Impuesto municipal a las actividades económicas (pago)** | Existe retención pero no módulo de declaración municipal |

---

---

## 7. Checklist de Pruebas por Módulo

Las pruebas están ordenadas igual que el orden de instalación. Cada sección asume que los módulos anteriores ya fueron instalados y probados.

---

### `l10n_ve_base`

> Módulo técnico — no tiene UI propia. Verificar que la instalación no produzca errores.

- [ ] El módulo instala sin errores en el log del servidor
- [ ] En **Ajustes → Técnico → Vistas** existen vistas con módulo `l10n_ve_base`
- [ ] En **Ajustes** aparece alguna sección con configuraciones de Venezuela (aunque estén vacías)

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
- [ ] Crear un contacto y verificar que los campos Municipio/Parroquia están disponibles y filtran correctamente por estado

---

### `l10n_ve_contact`

- [ ] Crear un contacto de tipo Empresa y verificar que aparece el campo **Prefijo RIF** (V, E, J, G, P)
- [ ] Ingresar un RIF con formato correcto (ej. `J-12345678-9`) — debe aceptarlo
- [ ] Ingresar un RIF con formato incorrecto — debe mostrar error de validación
- [ ] Verificar que el campo **Tipo de Contribuyente** está disponible (Ordinario, Especial, Exento, etc.)
- [ ] En **Ajustes → Empresa** verificar que el RIF de la empresa se puede configurar con prefijo
- [ ] Crear un contacto persona natural con prefijo `V` y uno jurídico con `J` — ambos deben guardarse

---

### `l10n_binaural`

- [ ] Ir a **Contabilidad → Configuración → Plan de Cuentas** — debe mostrar el plan de cuentas venezolano completo (activos, pasivos, patrimonio, ingresos, gastos)
- [ ] Verificar que existen cuentas de IVA (ej. IVA por pagar, IVA soportado)
- [ ] Ir a **Contabilidad → Configuración → Impuestos** — deben existir: IVA 16%, IVA 8%, Exento, Cero
- [ ] Ir a **Contabilidad → Configuración → Diarios** — deben existir al menos: Ventas, Compras, Banco, Caja
- [ ] Verificar que la empresa tiene asignado el plan de cuentas en **Ajustes → Empresa**

---

### `od_journal_sequence`

- [ ] Ir a **Contabilidad → Configuración → Diarios**, abrir el diario de Ventas
- [ ] Verificar que existe la pestaña o campo **Secuencia** con configuración de prefijo y siguiente número
- [ ] Crear una factura y confirmar — el número asignado debe seguir la secuencia del diario
- [ ] Crear una factura en otro diario — debe tener numeración independiente al primero
- [ ] Cambiar el prefijo de secuencia de un diario y confirmar una nueva factura — debe usar el nuevo prefijo

---

### `l10n_ve_accountant`

- [ ] Ir a **Contabilidad → Configuración → Unidad Tributaria** — debe existir el modelo con el valor actual de la UT
- [ ] Crear un valor de UT (ej. 0,02 VES) y guardar
- [ ] Crear una factura de cliente en USD — verificar que aparece el campo **Tasa de Cambio** en el encabezado
- [ ] El campo `Fecha de Factura (Display)` debe mostrarse correctamente en zona horaria Venezuela
- [ ] Ir a **Contabilidad → Reportes** — verificar que existen reportes venezolanos disponibles
- [ ] Crear un pago en USD — verificar que el sistema calcula el equivalente en VES automáticamente
- [ ] En **Ajustes → Contabilidad** verificar las configuraciones venezolanas (contribuyente especial, RIF empresa, etc.)
- [ ] Verificar que `res.company` tiene el campo **Contribuyente Especial** y se puede marcar

---

### `l10n_ve_invoice`

- [ ] Crear una factura de cliente, confirmarla — debe aparecer el campo **Número de Control** con correlativo automático
- [ ] Verificar que el número de control sigue la secuencia del diario (ej. `00-00000001`)
- [ ] Crear una segunda factura — el número de control debe ser `00-00000002`
- [ ] Ir al diario de ventas y configurar un prefijo de control diferente — nueva factura debe usar ese prefijo
- [ ] Crear una factura de proveedor — verificar que aparece el campo **Fecha de Recepción de Factura**
- [ ] En una factura de importación, verificar que el campo **Declaración Única de Aduanas** está disponible
- [ ] Imprimir una factura con el reporte **Factura en Formato Libre** — debe mostrar logo, RIF, número de control, datos del cliente y detalle
- [ ] Crear una Nota de Débito desde una factura confirmada — debe generar su propio número de control

---

### `l10n_ve_tax_payer`

- [ ] Abrir un contacto proveedor e ir a la pestaña **Contabilidad** (o sección tributaria)
- [ ] Verificar que existen los campos: **Retiene IVA**, **Porcentaje de Retención IVA**, **Retiene ISLR**, **Retiene Municipal**
- [ ] Marcar un proveedor como **Contribuyente Especial** con retención de IVA al 75% — guardar
- [ ] Verificar que el flag persiste y es visible al reabrir el registro
- [ ] Crear otro proveedor como ordinario sin retenciones — verificar diferencia en flags

---

### `l10n_ve_payment_extension`

**Retención IVA:**
- [ ] Configurar un proveedor como Contribuyente Especial (retiene IVA 75%)
- [ ] Crear una factura de ese proveedor por 100 USD + IVA 16% → base IVA = 16 USD → retención = 12 USD
- [ ] Al registrar el pago, verificar que aparece el botón/sección de **Crear Retención IVA**
- [ ] Crear la retención — debe generarse un `account.retention` con número secuencial
- [ ] Imprimir el **Comprobante de Retención IVA (ARCV)** — debe mostrar datos de empresa, proveedor, factura y monto retenido
- [ ] Confirmar la retención y verificar el asiento contable generado

**Retención ISLR:**
- [ ] Ir a **Contabilidad → Configuración → Conceptos de Retención ISLR** — deben existir conceptos predefinidos (honorarios, servicios, etc.)
- [ ] Marcar un proveedor con **Retiene ISLR**
- [ ] Crear factura de ese proveedor por servicio profesional
- [ ] Al pagar, crear la retención ISLR con el concepto correspondiente — verificar cálculo de alícuota
- [ ] Imprimir el ARCV de ISLR — debe mostrar acumulado de honorarios si aplica

**Retención Municipal:**
- [ ] Ir a **Contabilidad → Configuración → Actividades Económicas** — verificar que existen actividades
- [ ] Asignar una actividad económica al proveedor
- [ ] Crear factura y retención municipal — verificar cálculo según alícuota de la actividad
- [ ] Imprimir comprobante de retención municipal

---

### `l10n_ve_igtf`

- [ ] Ir a **Contabilidad → Configuración → Diarios**, abrir el diario de Banco en USD
- [ ] Verificar que existe el campo **Aplica IGTF** — marcarlo
- [ ] Configurar la **Cuenta Puente IGTF** en **Ajustes → Contabilidad**
- [ ] Crear una factura de cliente en USD y registrar el pago por ese diario bancario USD
- [ ] Verificar que al confirmar el pago, se genera automáticamente el cargo del 3% de IGTF
- [ ] Revisar el asiento contable del pago — debe tener una línea adicional de IGTF
- [ ] Crear un anticipo de cliente: crear pago por adelantado sin factura — debe marcarse como `is_advance_move`
- [ ] Verificar que el anticipo queda disponible para aplicar a futuras facturas
- [ ] En la factura impresa (**Formato Libre**), verificar que aparece la línea de IGTF cuando aplica

---

### `l10n_ve_filter_partner`

> Módulo técnico. Verificar comportamiento en formularios.

- [ ] Al crear una factura de cliente, el campo **Cliente** solo muestra contactos marcados como clientes
- [ ] Al crear una factura de proveedor, el campo **Proveedor** solo muestra contactos marcados como proveedores
- [ ] Crear un contacto que sea solo proveedor — no debe aparecer en el selector de clientes de facturas de venta

---

### `l10n_ve_stock`

- [ ] Ir a **Inventario → Productos** — verificar que los productos tienen campo de **Precio en Moneda Extranjera**
- [ ] Ingresar precio en USD a un producto — verificar que se calcula el equivalente en VES según tasa
- [ ] Crear un ajuste de inventario — verificar que la valoración muestra VES
- [ ] Ir a **Inventario → Reportes → Valoración de Inventario** — debe mostrar valores en VES con nota de tasa de cambio
- [ ] Crear una transferencia interna — verificar que los campos venezolanos están presentes
- [ ] Imprimir etiqueta de empaque desde una transferencia — debe generarse correctamente

---

### `l10n_ve_sale`

- [ ] Crear una orden de venta — verificar que el selector de cliente usa el filtro venezolano (solo clientes)
- [ ] Agregar una línea con producto en USD — verificar que la línea muestra precio en USD y equivalente en VES
- [ ] Cambiar la tasa de cambio y verificar que los precios se actualizan (o el cron lo hace)
- [ ] Confirmar la orden y crear la factura — verificar que hereda número de control y datos venezolanos
- [ ] Imprimir el reporte de la orden de venta — debe mostrar datos venezolanos correctamente
- [ ] Verificar que el campo **Almacén** en la orden usa el almacén venezolano configurado

---

### `l10n_ve_purchase`

- [ ] Crear una orden de compra — verificar que el selector de proveedor funciona correctamente
- [ ] Confirmar la orden y crear la factura de proveedor — verificar que hereda fecha de recepción
- [ ] Verificar que los grupos de seguridad de compras están correctamente configurados

---

### `l10n_ve_stock_purchase`

- [ ] Crear una orden de compra con productos en inventario
- [ ] Confirmar la orden — debe crearse automáticamente un recibo en almacén
- [ ] Validar el recibo — verificar que el movimiento de inventario usa los campos venezolanos
- [ ] Crear la factura desde la orden de compra confirmada — verificar integración completa

---

### `l10n_ve_stock_account`

- [ ] Ir a **Inventario → Configuración → Razones de Transferencia** — deben existir razones predefinidas (Venta, Traslado, Donación, etc.)
- [ ] Crear una transferencia de almacén y asignarle una razón
- [ ] Validar la transferencia — debe generarse o vincularse una **Guía de Despacho**
- [ ] Abrir la guía de despacho — verificar número secuencial, campos de transportista, origen y destino
- [ ] Imprimir la guía de despacho — debe ser un documento imprimible con formato legal
- [ ] Configurar una **Alerta de Autoconsumo** en **Inventario → Configuración → Alertas**
- [ ] Simular un movimiento que dispare la alerta — verificar que la notificación aparece
- [ ] Verificar que la factura generada desde la guía de despacho tiene número de control correcto

---

### `l10n_ve_stock_reports`

- [ ] Ir a **Inventario → Reportes → Libro de Inventario**
- [ ] Seleccionar un período (ej. el mes actual) y ejecutar el reporte
- [ ] Verificar que el reporte muestra movimientos de entrada, salida y saldo por producto
- [ ] Exportar o imprimir el reporte — debe ser apto para presentación fiscal

---

### `l10n_ve_pos`

- [ ] Ir a **Punto de Venta → Configuración → Ajustes** — verificar campos venezolanos (almacén, tasa de cambio)
- [ ] Configurar el POS con la tasa de cambio del día
- [ ] Abrir una sesión de POS
- [ ] Buscar un cliente por **RIF** desde el POS — debe encontrar al contacto venezolano
- [ ] Crear una venta con pago en USD — verificar que el sistema muestra el equivalente en VES
- [ ] Crear una venta con pago en VES — flujo normal
- [ ] Cerrar la sesión de POS — verificar que el cierre muestra resumen por tipo de pago y moneda
- [ ] Generar el **Reporte de Pagos** de la sesión — debe mostrar desgloses venezolanos

---

### `l10n_ve_pos_igtf`

- [ ] Configurar un método de pago en USD en el POS
- [ ] Abrir el POS y crear una venta
- [ ] Seleccionar pago en USD — debe aparecer el cálculo del IGTF (3%) automáticamente
- [ ] Verificar que el total cobrado incluye el IGTF
- [ ] Revisar la orden cerrada — el IGTF debe estar registrado como línea separada en el pago

---

### `l10n_ve_stock_reports`

- [ ] Ir al menú de reportes de inventario
- [ ] Ejecutar el **Libro de Inventario** para un período
- [ ] Verificar que incluye productos con sus movimientos de entrada/salida y saldo final
- [ ] Confirmar que los valores están en VES con la tasa del período

---

### `account_fiscal_year_closing`

- [ ] Ir a **Contabilidad → Contabilidad → Cierres de Año Fiscal**
- [ ] Verificar que existen **Plantillas de Cierre** precargadas
- [ ] Crear un nuevo cierre de año fiscal para el período anterior
- [ ] Ejecutar el paso "Preparar" — debe validar que no hay asientos pendientes de confirmar
- [ ] Revisar los asientos de cierre propuestos — deben incluir traslado de resultados y apertura de balance

---

### `l10n_ve_account_fiscalyear_closing`

- [ ] Abrir el cierre fiscal creado anteriormente — verificar que tiene campos venezolanos (RIF empresa, tasa de cambio de cierre)
- [ ] Verificar que las plantillas de cierre venezolanas están disponibles
- [ ] Ejecutar el cierre completo (preparar → confirmar → validar)
- [ ] Verificar que los asientos generados tienen la tasa de cambio correcta al cierre del período

---

### `l10n_ve_fiscal_lock_days`

- [ ] Ir a **Contabilidad → Configuración → Ajustes** — sección de bloqueo fiscal
- [ ] Configurar una fecha de bloqueo de facturas (ej. bloquear hasta el 31 del mes anterior)
- [ ] Intentar crear y confirmar una factura con fecha anterior al bloqueo — debe mostrar error de período bloqueado
- [ ] Cambiar la fecha de bloqueo usando el wizard **Cambiar Fecha de Bloqueo** — debe requerir justificación
- [ ] Verificar que el cambio de fecha queda registrado en el log de auditoría

---

### `l10n_ve_ref_bank`

- [ ] Ir a **Contabilidad → Configuración → Diarios**, abrir un diario de tipo Banco
- [ ] Verificar que existen campos de configuración de referencia bancaria (prefijo, longitud)
- [ ] Configurar el diario con un prefijo y longitud de referencia (ej. 20 dígitos)
- [ ] Registrar un pago con ese diario usando una referencia válida — debe aceptarla
- [ ] Registrar un pago con referencia de longitud incorrecta — debe mostrar error de validación

---

### `l10n_ve_suggested_amount`

- [ ] Crear una factura de cliente en USD con saldo pendiente
- [ ] Ir a **Registrar Pago** desde la factura
- [ ] En el wizard de pago, verificar que aparece el campo **Monto Sugerido** con el equivalente en VES
- [ ] Cambiar la tasa de cambio en el wizard — el monto sugerido debe recalcularse
- [ ] Confirmar el pago con el monto sugerido — la factura debe quedar saldada

---

### `l10n_ve_auditlog`

- [ ] Crear y confirmar una factura
- [ ] Modificar algún campo editable (ej. nota) y guardar
- [ ] Ir al chatter de la factura — debe mostrar el tracking del cambio con valor anterior y nuevo
- [ ] Registrar un pago y luego intentar modificarlo
- [ ] Ir a **Contabilidad → Técnico → Logs de Auditoría** (si el menú existe) — verificar registros
- [ ] Verificar que los cambios en `account.payment` también quedan registrados

---

### `l10n_ve_donation`

- [ ] Ir a **Ajustes → Contabilidad** — verificar configuración de donaciones (cuenta contable de donación)
- [ ] Crear una orden de venta y marcarla como **Donación**
- [ ] Verificar que solo se puede seleccionar un cliente tipo Empresa (no persona natural)
- [ ] Confirmar la orden y validar el despacho — el movimiento de inventario debe usar la lógica de donación
- [ ] Crear la factura de donación — debe marcarse automáticamente como donación
- [ ] Imprimir el **Certificado de Donación** — debe mostrar datos del donante, descripción de bienes y montos

---

### `l10n_ve_price_list`

- [ ] Ir a **Ventas → Configuración → Listas de Precio** — crear una lista en USD
- [ ] Asignar la lista de precio a un cliente
- [ ] Crear una orden de venta para ese cliente — verificar que los precios se muestran en USD
- [ ] En la factura generada, verificar que la lista de precio y moneda están correctamente reflejadas
- [ ] Abrir un producto — verificar que la vista de lista de precio venezolana muestra precio en moneda extranjera

---

### `l10n_ve_currency_rate_live` *(opcional)*

- [ ] Ir a **Contabilidad → Configuración → Divisas**
- [ ] Hacer clic en **Actualizar Tasas** — el sistema debe consultar el BCV y actualizar la tasa del USD
- [ ] Verificar que la tasa actualizada es razonable (tasa BCV oficial del día)
- [ ] Configurar la actualización automática (cron) y verificar que está activa
- [ ] Revisar el historial de tasas de USD — deben aparecer las actualizaciones automáticas

---

### `l10n_ve_iot_mf` *(opcional — requiere hardware TFHKA)*

- [ ] Configurar la IP y token de la máquina fiscal en **Ajustes → Contabilidad**
- [ ] Ir a **IoT → Dispositivos** — verificar que aparece la máquina fiscal como dispositivo
- [ ] Marcar el diario de ventas como **Con Máquina Fiscal**
- [ ] Mapear los impuestos IVA 16% y 8% a los códigos de la máquina fiscal
- [ ] Crear y confirmar una factura de cliente — el sistema debe enviarla a la máquina fiscal
- [ ] Verificar que la factura recibe el número de secuencia fiscal de la máquina
- [ ] Ejecutar el **Reporte Z** al final del día — la máquina debe emitir el cierre diario

---

### `l10n_ve_invoice_digital` *(opcional — requiere `l10n_ve_iot_mf`)*

- [ ] En **Ajustes → Contabilidad** activar **Retenciones Automáticas en Factura Digital**
- [ ] Crear una factura para un proveedor con retención de IVA configurada
- [ ] Confirmar la factura — las retenciones deben generarse automáticamente sin intervención manual
- [ ] Verificar que el almacén de la factura tiene máquina fiscal asociada (requisito previo)
- [ ] Si el almacén no tiene máquina fiscal, debe aparecer una alerta de advertencia

---

### `l10n_ve_pos_mf` *(opcional — requiere `l10n_ve_iot_mf` + `l10n_ve_pos`)*

- [ ] En configuración del POS, asignar la máquina fiscal al punto de venta
- [ ] Abrir sesión de POS y realizar una venta
- [ ] Al cerrar la venta, verificar que se envía a la máquina fiscal y regresa con número fiscal
- [ ] Verificar que la orden del POS muestra el número de secuencia fiscal
- [ ] Ir a **Reportes → Libro de Ventas Fiscal** — debe mostrar ventas con números fiscales correlativos
- [ ] Ejecutar el Reporte Z del POS al cerrar la sesión

---

### `l10n_ve_invoice_loyalty` *(opcional)*

- [ ] Ir a **eCommerce / Ventas → Programas de Fidelización** — crear un programa con puntos
- [ ] Asignar el programa a un cliente
- [ ] Crear una factura para ese cliente — debe aparecer sección para aplicar puntos/rewards
- [ ] Aplicar un reward — verificar que se genera descuento o línea de ajuste en la factura
- [ ] Confirmar la factura — el reward debe registrarse en el programa de fidelización

---

*Checklist generado en base al análisis de código fuente. Los valores numéricos (porcentajes, montos) son ejemplos ilustrativos — usar los valores reales configurados en producción.*

---

*Documento generado por análisis directo del código fuente del repositorio `odoo-venezuela` rama `19.0`.*
