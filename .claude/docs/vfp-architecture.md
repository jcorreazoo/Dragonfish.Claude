# Arquitectura del Sistema

## Qué es

Sistema de **punto de venta** con módulos: Ventas, Compras, Inventario, Producción, Ecommerce, Contabilidad. Transversal: control de stock e impuestos (IVA, retenciones, ingresos brutos).

---

## Distribución

- Sistema distribuido: cada proyecto y cada DLL tiene su propio repositorio.
- Existen DLLs .NET para funcionalidades específicas (interfaces, promociones, etc.).
- **Dirección estratégica:** migrar lógica de VFP a .NET progresivamente. VFP queda como capa de presentación y orquestación.

> Antes de implementar algo nuevo en VFP, evaluar si corresponde a una DLL .NET.

---

## Proyectos principales

```
Dragonfish  →  Feline  →  Drawing  →  Core
(específico)                          (base)
```
`→` significa "hereda de". Pueden existir saltos (ej: Dragonfish → Core directamente) — al rastrear una clase base, buscar en **todos** los proyectos. La herencia directa de cada clase está declarada en el header de su `.prg`.

| Proyecto | Rol |
|---|---|
| `Organic.Core` | Base del sistema, clases fundamentales |
| `Organic.Drawing` | Componentes reutilizables |
| `Organic.Feline` | Lógica de negocio compartida |
| `Organic.Dragonfish` | Proyecto principal — este repo |

---

## Regla de intervención

El código nuevo siempre debe quedar en **Dragonfish**. Si un método en Feline o Core no es sobreescribible, primero refactorizarlo en el proyecto base para habilitarlo, luego sobreescribirlo en Dragonfish. No modificar lógica de negocio en proyectos base si puede resolverse por sobreescritura.

---

## Nomenclatura de archivos `.prg`

### Entidades

El sistema está organizado en **Entidades**: clases que agrupan por convención otras subclases vinculadas internamente (no por herencia).

| Prefijo / Palabra | Rol |
|---|---|
| `ADN*.prg` | Punto de entrada que declara e inicializa la entidad |
| `*Entidad*` / `*Ent*` | Clase principal de la entidad |
| `*ABM*` | Formulario de Alta, Baja y Modificación |
| `*Kontroler*` | Vincula el formulario con la entidad (binding visual) |
| `*Detalle*` | Representa el detalle de la entidad. Visualmente es un conjunto de textboxes agrupados con apariencia de grilla, no una grilla real |
| `*Item*` | Representa cada fila/ítem dentro del detalle |
| `*Componente*` | Contiene métodos auxiliares de la entidad; su método `Grabar()` se ejecuta dentro de la misma transacción al impactar datos en SQL, para actualizar tablas no directamente relacionadas con la entidad |
| `*Colaborador*` | Clase auxiliar reutilizable, instanciada por separado. Agrupa métodos relacionados con un tema específico y puede ser usada por distintas entidades o subentidades para evitar duplicidad de código |
| `*REST*` | Funcionalidad de la entidad expuesta vía API |
| `*AD_SQLServer*` | Sentencias INSERT / UPDATE / DELETE contra SQL Server |

> No todas las entidades tienen formulario `ABM` asociado.

### Prefijos que indican herencia

| Prefijo | Hereda de |
|---|---|
| `colorytalle_NombreClase` | Clase `NombreClase` en `Organic.Feline` |

Ejemplo: `colorytalle_componentestock` → hereda de `componentestock` en `Organic.Feline`.

---

### Archivos generados — NO EDITAR

| Prefijo | Origen |
|---|---|
| `DIN_*` | Generado automáticamente por el sistema vía ADN |
| `Mock-*` | Generado automáticamente por el sistema vía ADN |

Si algo está mal en estos archivos, **corregir el generador (ADN)**, nunca el archivo generado.

---

## LoadReference — herencia desde VCX

Cuando una clase hereda de una clase visual (`.vcx`) que vive en otro proyecto compilado (`.app`), se requiere una línea `LoadReference` antes del `Define Class`:

```foxpro
_screen._instanceFactory.LoadReference('NombreClase.vcx', "Organic.Core.app")

Define Class MiFormulario As NombreClase Of NombreClase.vcx
    ...
Enddefine
```

| Situación | Acción |
|---|---|
| VCX en proyecto externo (.app) | Agregar `LoadReference` con la app correcta |
| VCX local (en este repo) | **Sin** `LoadReference` |
| App incorrecta | Corregir — buscar en `packages/Exe/*.symbols` |

Apps disponibles: `Organic.Core.app`, `Organic.Drawing.app`, `Organic.Feline.app`.

---

## Convenciones de formularios, cursores y aliases

> (pendiente)
