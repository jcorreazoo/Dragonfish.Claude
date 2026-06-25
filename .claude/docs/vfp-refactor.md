# Patrones de Refactorización VFP

Para el modo **Refactor**. Aplicar siempre de forma incremental: cambio pequeño → compilar → test.

## Checklist previo

- [ ] ¿Existen tests? Si no, crearlos primero
- [ ] ¿El código compila y funciona? Verificar
- [ ] Commitear antes de refactorizar

---

## Catálogo

### 1. Extract Method
**Cuándo:** método > 50 líneas o lógica que tiene nombre propio.

```foxpro
* Antes: un método largo
* Después: delegar a métodos con nombre
Function ProcesarVenta() As Boolean
    If !This.ValidarCliente()
        Return .F.
    Endif
    Local lnTotal As Number
    lnTotal = This.CalcularTotal()
    Return This.Guardar(lnTotal)
Endfunc
```

---

### 2. Replace Magic Number with Constant
```foxpro
* Antes:  If This.nEstado = 3
#Define ESTADO_PROCESANDO 3
* Después: If This.nEstado = ESTADO_PROCESANDO
```

---

### 3. Introduce Parameter Object
**Cuándo:** método con > 3 parámetros.

```foxpro
* Antes:  Function CrearFactura(tcCliente, tcCUIT, tnTotal, tdFecha)
Define Class DatosFactura As Custom
    cCliente = "" & cCUIT = "" & nTotal = 0 & dFecha = {}
Enddefine
* Después: Function CrearFactura(toDatos As DatosFactura)
```

---

### 4. Decompose Conditional
**Cuándo:** condición compleja difícil de leer.

```foxpro
* Antes:  If (nEdad >= 18 And nEdad <= 65) And (nIngreso >= 20000 Or lTrabajo)
* Después:
If This.EsEdadValida() And This.TieneSolvencia()
Protected Function EsEdadValida() As Boolean
    Return Between(This.nEdad, 18, 65)
Endfunc
```

---

### 5. Replace Conditional with Polymorphism
**Cuándo:** `Do Case` basado en tipo de objeto.

```foxpro
* Antes: Do Case / Case tcTipo = "IVA" / Case tcTipo = "IIBB" / Endcase
* Después: clase base + subclases por tipo
Define Class CalculadorIVA As CalculadorImpuesto
    Function Calcular(tnMonto As Number) As Number
        Return tnMonto * 0.21
    Endfunc
Enddefine
```

---

### 6. Dependency Injection (para testabilidad)
**Cuándo:** clase instancia dependencias internamente, imposibilitando mocks.

```foxpro
* Antes: loAcceso = Createobject("AccesoDatosSQL") dentro del método
Define Class ServicioFacturacion As Custom
    oAccesoDatos = Null
    Procedure Init(toAccesoDatos As Object)
        This.oAccesoDatos = toAccesoDatos  && inyectado desde afuera
    Endproc
Enddefine
* En tests: Createobject("ServicioFacturacion", Createobject("AccesoDatosMock"))
```

---

### 7. Replace Error Code with Exception
```foxpro
* Antes: Return -1 / Return -2 con ON ERROR
* Después:
Function AbrirArchivo(tcRuta As String) As Boolean
    Try
        If !File(tcRuta)
            Error "Archivo no existe: " + tcRuta
        Endif
        Use (tcRuta) In 0
        Return .T.
    Catch To loEx
        This.LogError("AbrirArchivo", loEx)
        Throw
    Endtry
Endfunc
```

---

## Prioridades de refactorización

1. **Alta** — código crítico, frecuentemente modificado, con bugs recurrentes
2. **Media** — código complejo pero estable
3. **Baja** — código legacy que funciona y rara vez se toca
