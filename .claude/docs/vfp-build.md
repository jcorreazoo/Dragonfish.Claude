# Build — DOVFP

DOVFP es el compilador personalizado para VFP 9 que permite builds desde VS Code.

## Comandos esenciales

```bash
dovfp build Organic.Drawing.vfpsln              # solución completa
dovfp build Organic.BusinessLogic/...           # proyecto específico
dovfp build Organic.Drawing.vfpsln -build_debug 2   # Release
dovfp build -build_force 1                      # forzar recompilación completa
dovfp rebuild                                   # limpiar y recompilar en un paso
dovfp run                                       # ejecutar
dovfp run -run_args "'param1', 123, .T."        # con argumentos
dovfp test Organic.Tests/Organic.Tests.vfpproj
dovfp restore                                   # restaurar dependencias
dovfp restore -force_download 1                 # forzar re-descarga de paquetes
dovfp clean                                     # limpiar
```

CI/CD: Azure Pipelines (`azure-pipelines.yml`). Autenticación NuGet via Azure CLI (`az login`).

---

## Cuándo hacer clean

- Después de cambios en `.vfpproj`
- Después de agregar/quitar archivos
- Ante errores extraños de compilación

---

## Troubleshooting

```powershell
# DOVFP no encontrado
dotnet tool list --global
dotnet tool install --global dovfp --add-source ./nupkg
dotnet tool update --global dovfp --add-source ./nupkg

# Error de autenticación NuGet
az login
az account show

# Build falla sin errores claros
dovfp clean
Remove-Item -Recurse obj/
dovfp build
```

---

## Tests fallan en CI pero pasan local

Causa más común: rutas absolutas. Usar siempre rutas relativas:

```foxpro
Local lcRutaBase As String
lcRutaBase = Justpath(Sys(16))  && ruta del programa actual
```

También verificar: configuración regional, dependencias de archivos locales, estado compartido entre tests.

---

## Checklist antes de push

- [ ] `.vfpsln` incluye todos los proyectos activos
- [ ] Cada `.vfpproj` tiene `OutputPath` correcto
- [ ] `MainProgram` apunta al PRG de inicio correcto
- [ ] `NuGet.config` sin credenciales hardcodeadas
- [ ] Tests pasan localmente
- [ ] Build en modo Release exitoso (`-build_debug 2`)

---

## Reglas

- Compilar localmente antes de push
- Ejecutar tests antes de merge
- No commitear `bin/`, `obj/`, `packages/`
- No hardcodear credenciales en `Nuget.config`
- No ignorar warnings de compilación
