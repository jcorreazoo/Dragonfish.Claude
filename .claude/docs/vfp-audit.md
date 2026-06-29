# Auditoría de Código VFP

Checklist para el modo **Auditor**. Reportar cada issue con severidad 🔴 Alta / 🟡 Media / 🟢 Baja.

## Checklist

### Arquitectura
- [ ] Clases con responsabilidad única
- [ ] Sin dependencias circulares
- [ ] Cohesión alta en métodos

### Code Smells
- [ ] Métodos < 50 líneas
- [ ] Clases < 500 líneas / < 15 métodos públicos
- [ ] Sin código duplicado
- [ ] Sin magic numbers (usar `#DEFINE`)
- [ ] Sin nombres crípticos (`x`, `tmp`, `aux`)
- [ ] Sin variables globales (`Public`/`Private`)

### Calidad
- [ ] `Try/Catch` en operaciones críticas
- [ ] Errores logueados, no silenciados
- [ ] Recursos liberados en `Finally` y `Destroy()`
- [ ] Sin memory leaks (objetos no liberados)
- [ ] Sin cursores/tablas abiertos sin cerrar
- [ ] Sin construcción de SQL por concatenación (injection)

### Performance
- [ ] SQL sobre `Scan/Endscan`
- [ ] Sin queries N+1 (queries dentro de loops)
- [ ] `Select` con `Where` — sin lecturas de tablas completas
- [ ] Índices usados (`Seek`, `Indexseek`)
- [ ] Buffering apropiado

### Seguridad
- [ ] Validación de entrada en métodos públicos
- [ ] Sin credenciales hardcodeadas
- [ ] Logs sin datos sensibles

### Estilo y convenciones
- [ ] Nomenclatura húngara respetada
- [ ] Keywords en Capital
- [ ] Siempre `Function`/`Endfunc` — nunca `Procedure`/`Endproc`
- [ ] Parámetros tipados (`As String`, `As Boolean`, etc.)
- [ ] Header de clase con `*===...===` y propósito

## Formato de reporte por issue

```
[🔴/🟡/🟢] Título del Issue
Archivo: ruta/archivo.prg — Línea: 123
Categoría: Code Smell | Performance | Seguridad | Estilo
Problema: descripción
Código actual: [snippet]
Sugerencia: [snippet mejorado]
Impacto: efecto en el sistema
```

## Prioridades

1. 🔴 **Alta** — bugs potenciales, seguridad, memory leaks, data corruption
2. 🟡 **Media** — performance, mantenibilidad degradada
3. 🟢 **Baja** — estilo, documentación, mejoras menores
