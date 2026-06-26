# Organic.Dragonfish — Contexto para Claude Code

@.claude/docs/vfp-architecture.md
@.claude/docs/vfp-conventions.md
@.claude/docs/vfp-testing.md
@.claude/docs/vfp-build.md
@.claude/docs/vfp-audit.md
@.claude/docs/vfp-refactor.md

## El Proyecto

Aplicación empresarial **Visual FoxPro 9** compilada con **DOVFP** (.NET 6), desarrollada en VS Code. Código de negocio en `Organic.BusinessLogic/CENTRALSS/`.

### Estructura de proyectos

| Carpeta | Tipo | Regla |
|---|---|---|
| `Organic.BusinessLogic/` | Código principal | Editable |
| `Organic.Generated/` | Código auto-generado | **NO EDITAR** |
| `Organic.Tests/` | Tests unitarios | Editable |
| `Organic.Hooks/` | Extensiones | Editable |
| `Organic.Mocks/` | Clases mock | Editable |

---

## Testing

Framework: **FxuTestCase**. Tests en `Organic.Tests/`. Ver `.claude/docs/vfp-testing.md`.

---

## Build

Ver `.claude/docs/vfp-build.md`.

---

## Encoding de archivos

Los archivos `.prg` están en **Windows-1252**, definido en [.vscode/settings.json](.vscode/settings.json). El tool `Edit` los reescribe como UTF-8 corrompiendo los caracteres especiales.

**Regla:** para editar archivos `.prg` usar siempre PowerShell con encoding explícito:

```powershell
$enc = [System.Text.Encoding]::GetEncoding(1252)
$content = [System.IO.File]::ReadAllText($file, $enc)
# ... modificar $content ...
[System.IO.File]::WriteAllText($file, $newContent, $enc)
```

---

## Reglas del Workspace

- **NO** crear archivos de reporte: `*-REPORT.md`, `*-SUMMARY.md`, `*-ANALYSIS.md`
- **NO** crear archivos temporales: `.tmp`, `.bak`, `debug-*`
- **NO** editar `Organic.Generated/Generados/`
- **NO** commitear `bin/`, `obj/`, `packages/`

---

## Modos de Trabajo (equivalentes a los Agents de Copilot)

Pedime explícitamente que trabaje en uno de estos modos:

| Modo | Cuándo usarlo |
|---|---|
| **Developer** | Implementar funcionalidades, código de negocio |
| **Test Engineer** | Crear o mejorar tests unitarios |
| **Auditor** | Revisar calidad, detectar code smells |
| **Refactor** | Modernizar código legacy |

### Flujo recomendado

Developer → Test Engineer → Auditor → Refactor → (validar con Test Engineer)

