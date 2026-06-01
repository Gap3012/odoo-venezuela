# Módulos Odoo Venezuela — Mapeo Completo

**Repositorio:** odoo-venezuela (rama 19.0)  
**Fecha de análisis:** 2026-06-01  
**Total de módulos:** 35

---

## Tabla de Contenido

1. [Ecosistema General](#1-ecosistema-general)
2. [Orden de Instalación Recomendado](#2-orden-de-instalación-recomendado)
3. [Módulos por Capa](#3-módulos-por-capa)
   - [Capa 1 — Infraestructura Base](#capa-1--infraestructura-base)
   - [Capa 2 — Datos Geográficos y Contactos](#capa-2--datos-geográficos-y-contactos)
   - [Capa 3 — Tipos de Cambio y Plan de Cuentas](#capa-3--tipos-de-cambio-y-plan-de-cuentas)
   - [Capa 4 — Contabilidad Core](#capa-4--contabilidad-core)
   - [Capa 5 — Facturación y Secuencias](#capa-5--facturación-y-secuencias)
   - [Capa 6 — Retenciones e Impuestos Especiales](#capa-6--retenciones-e-impuestos-especiales)
   - [Capa 7 — Inventario y Logística](#capa-7--inventario-y-logística)
   - [Capa 8 — Ventas, Compras y POS](#capa-8--ventas-compras-y-pos)
   - [Capa 9 — Máquinas Fiscales e IoT](#capa-9--máquinas-fiscales-e-iot)
   - [Capa 10 — Módulos Opcionales / Extensiones](#capa-10--módulos-opcionales--extensiones)
4. [Fichas Detalladas de Cada Módulo](#4-fichas-detalladas-de-cada-módulo)
5. [Árbol de Dependencias](#5-árbol-de-dependencias)
6. [Qué cubre y qué no cubre esta localización](#6-qué-cubre-y-qué-no-cubre-esta-localización)

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

### Capa 1 — Infraestructura Base

#### `l10n_ve_base`
Infraestructura técnica mínima. Extiende `ir.module.module` e `ir.ui.view` para que otros módulos de la localización puedan registrar vistas y configuraciones sin conflictos. Agrega configuraciones en `res.config.settings` comunes a toda la localización.

---

### Capa 2 — Datos Geográficos y Contactos

#### `l10n_ve_location`
Base de datos geográfica completa de Venezuela: ciudades, municipios (23 estados) y parroquias. Define los modelos `res.country.city`, `res.country.municipality` y `res.country.parish`, extiende `res.partner` para enlazarlos. Los datos vienen cargados como CSV/XML en el módulo.

#### `l10n_ve_contact`
Extiende `res.partner` con campos venezolanos críticos:
- **`prefix_vat`**: prefijo del RIF (`V`, `E`, `J`, `G`, `P`)
- **Tipo de contribuyente**: natural, jurídico, gobierno, etc.
- Validación de RIF con lógica de dígito verificador
- Enlace a municipio y parroquia del módulo `l10n_ve_location`
- Extiende `res.company` y `res.config.settings` para información fiscal de la empresa

---

### Capa 3 — Tipos de Cambio y Plan de Cuentas

#### `l10n_ve_rate`
Núcleo del manejo multimoneda venezolano. Extiende `res.currency.rate` para soportar múltiples tipos de tasa (BCV oficial, paralela, etc.). Extiende `res.company` y `res.currency` con lógica de conversión específica. Todo el stack de localización lo usa para calcular equivalencias en VES/USD.

#### `l10n_ve_currency_rate_live`
Opcional. Agrega un proveedor de tasa de cambio que consulta la API del BCV automáticamente, sincronizando la tasa oficial sin intervención manual. Depende de `currency_rate_live` (módulo OCA).

#### `l10n_binaural`
Plan de cuentas completo para **empresas de servicio venezolanas**. Contiene:
- Árbol de cuentas contables (activos, pasivos, patrimonio, ingresos, gastos)
- Diarios contables preconfigurados
- Impuestos IVA (16%, exento, cero) y retenciones básicas
- `product.template` para productos de servicio
- Datos de configuración inicial

#### `l10n_ve_binaural`
Plan de cuentas alternativo. Define el modelo `template_ve.py` con la plantilla oficial de Odoo para Venezuela, más datos de demostración en `demo_company.xml`. Es más liviano que `l10n_binaural` y útil para empresas que parten de cero con Odoo.

---

### Capa 4 — Contabilidad Core

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

### Capa 5 — Facturación y Secuencias

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

### Capa 6 — Retenciones e Impuestos Especiales

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

### Capa 7 — Inventario y Logística

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

### Capa 8 — Ventas, Compras y POS

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

### Capa 9 — Máquinas Fiscales e IoT

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

### Capa 10 — Módulos Opcionales / Extensiones

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

*Documento generado por análisis directo del código fuente del repositorio `odoo-venezuela` rama `19.0`.*
