# Convenciones de Código VFP

> VFP no es case sensitive ni fuertemente tipado. Las convenciones siguientes son de estilo y legibilidad.

## Casing de keywords

- Keywords en **Capital**: `Function`, `Endfunc`, `Define Class`, `Enddefine`, `Local`, `Return`, `Try`, `Catch`, `Finally`, `Endtry`, `If`, `Endif`, `This`, etc.
- Siempre usar `Function`/`Endfunc` — nunca `Procedure`/`Endproc`, salvo indicación expresa.

---

## Declaración de funciones

Salvo indicación expresa, toda función se declara así:

```foxpro
*-----------------------------------------------------------------------------------------
Function NombreFuncion( tcParam1 As String, tnParam2 As Integer ) As TipoRetorno
    ...
Endfunc
```

- Separador `*---...---` antes de cada función
- Nombre en PascalCase
- Parámetros tipados: `As String`, `As Integer`, `As Boolean`, `As Object`, `As Void`
- Tipo de retorno explícito

---

## Estructura de clase

```foxpro
*==============================================================================
* Clase: NombreClase
* Propósito: [descripción]
*==============================================================================
Define Class NombreClase As ParentClass

    cPropiedad = ""
    nPropiedad = 0

    Function Init(tcParam1, tnParam2) As Void
        This.cPropiedad = Evl(tcParam1, "")
        This.nPropiedad = Evl(tnParam2, 0)
        Return Dodefault()
    Endfunc

    *-----------------------------------------------------------------------------------------
    Function MetodoPublico( tcInput As String ) As Boolean
        Local llExito, loError
        llExito = .F.
        Try
            * lógica principal
            llExito = .T.
        Catch To loError
            goServicios.Errores.LevantarExcepcion(loError)
        Finally
            * liberar recursos
        Endtry
        Return llExito
    Endfunc

    Function Destroy() As Void
        This.oObjeto = Null
        Return Dodefault()
    Endfunc

Enddefine
```

> `Init` puede declararse con o sin parámetros tipados. `Destroy` no tiene parámetros.

---

## Nomenclatura húngara

```foxpro
* Variables: prefijo en minúscula (tipo) + resto en Capital
lcCaracter    && Local Character
lnNumero      && Local Numeric
llLogico      && Local Logical
loObjeto      && Local Object
laArray       && Local Array

tcParam       && parámetro Character
tnParam       && parámetro Numeric
tlParam       && parámetro Logical
toParam       && parámetro Object

This.cPropiedad = ""    && Character
This.nPropiedad = 0     && Numeric
This.lPropiedad = .F.   && Logical
This.oPropiedad = Null  && Object
```

---

## Servicios globales e instanciación

### Instanciación de objetos

No usar `Createobject()` directamente — usar los helpers del sistema:

```foxpro
loObjeto    = _Screen.zoo.CrearObjeto("NombreClase")
loEntidad   = _Screen.zoo.InstanciarEntidad("NombreEntidad")
loComponente = _Screen.zoo.InstanciarComponente("NombreComponente")

* Formularios
goFormularios.Mostrar("NombreFormulario")
goFormularios.Procesar("NombreFormulario")
```

### Servicios disponibles

| Servicio | Propósito |
|---|---|
| `goServicios.Errores` | Manejo centralizado de errores |
| `goServicios.Datos` | Acceso a datos / ejecución SQL |
| `goParametros` | Parámetros de configuración del sistema |
| `goMensajes` | Mensajes al usuario |
| `_Screen.zoo` | Contexto central de la aplicación |

---

## Manejo de errores — goServicios.Errores

El sistema usa `goServicios.Errores` como servicio global centralizado:

```foxpro
* Levantar excepción con objeto error
Catch To loError
    goServicios.Errores.LevantarExcepcion(loError)

* Levantar excepción con mensaje y código
goServicios.Errores.LevantarExcepcionTexto("El dato no existe.", 9001)

* Verificar error específico (ej: conflicto de concurrencia por timestamp)
llEsTimestamp = (This.oManejadorErrores.HuboErrorUP( ;
    goServicios.Errores.ObtenerCodigoErrorParaValidacionTimestamp()) > 0)
```

> No usar `This.LogError()` genérico — usar siempre `goServicios.Errores`.

---

## Transacciones

```foxpro
Function Grabar() As Boolean
    Local llExito As Boolean, loError As Object
    llExito = .F.
    Try
        Begin Transaction
            This.oEntidad.Grabar()
            This.ActualizarRelacionados()
        End Transaction
        llExito = .T.
    Catch To loError
        If Txnlevel() > 0
            Rollback
        Endif
        If This.EsErrorDeConcurrencia(loError)
            This.ManejarErrorConcurrencia()
        Else
            goServicios.Errores.LevantarExcepcion(loError)
        Endif
    Endtry
    Return llExito
Endfunc
```

---

## Reglas generales

- Preferir SQL sobre `Scan/Endscan`
- Funciones < 50 líneas, una responsabilidad por función
- Liberar recursos en `Destroy()`
- No variables globales (`Public`/`Private`)
- No rutas hardcodeadas
- No código comentado (usar Git)
- No magic numbers (usar constantes)
