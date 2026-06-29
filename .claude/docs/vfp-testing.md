# Testing VFP — FxuTestCase

## Estructura de un test

```foxpro
Define Class ztestNombreModulo As FxuTestCase OF FxuTestCase.prg

    oSUT = Null

    Function Setup
        This.oSUT = Createobject("ClaseATestear")
    Endfunc

    Function TearDown
        This.oSUT = Null
    Endfunc

    Function zTestU_Metodo_DebeResultado_CuandoCondicion
        Local lcInput, lcEsperado, lcResultado
        lcInput    = "valor"                          && Arrange
        lcResultado = This.oSUT.Procesar(lcInput)     && Act
        This.AssertEquals("msg", lcEsperado, lcResultado)  && Assert
    Endfunc

Enddefine
```

---

## Nomenclatura de tests

```
zTestU_[Método]_Debe[Comportamiento]_Cuando[Condición]
```

- `zTestU_` — test unitario (default, salvo indicación expresa)
- `zTestI_` — test de integración (solo cuando se indique explícitamente)

Ejemplos:
- `zTestU_ProcesarVenta_DebeRetornarTrue_CuandoClienteTieneCredito`
- `zTestU_ValidarEmail_DebeGenerarError_CuandoEmailEsInvalido`

---

## Assertions disponibles

```foxpro
This.AssertEquals("msg", esperado, actual)
This.AssertTrue("msg", expr)
This.AssertTrue("msg", !expr)      && negación — AssertFalse NO existe en VFP
This.AssertNull("msg", var)
This.AssertNotNull("msg", var)
This.AssertContains("msg", "subcadena", lcTexto)
This.AssertType(variable, "C", "msg")   && C=Character, N=Numeric, L=Logical
This.AssertThrows("NombreMetodo", This.oSUT, "msg")  && verifica que el método lanza error
```

---

## Mocks

### Datos mock
```foxpro
Use ClasesMock In 0 Shared
Insert Into ClasesMock (id, nombre, tipo) Values (999, "Test", "Mock")
* Limpiar en TearDown:
Delete From ClasesMock Where tipo = "Mock"
Pack
```

### Clase mock
```foxpro
Define Class RepositorioMock As Custom
    Dimension aDatos[1, 2]
    nCount = 0

    Function AgregarDato(tnId As Integer, tcValor As String) As Void
        This.nCount = This.nCount + 1
        Dimension This.aDatos[This.nCount, 2]
        This.aDatos[This.nCount, 1] = tnId
        This.aDatos[This.nCount, 2] = tcValor
    Endfunc
Enddefine
```

---

## Edge cases a cubrir siempre

- Parámetro `Null`
- String vacío `""`
- Cero `0` y negativos
- Números muy grandes
- Fechas inválidas
- Arrays vacíos
- Tipos incorrectos

---

## Organización

```
Organic.Tests/
├── main.prg
├── ClasesMock.dbf
├── clasesdeprueba/       && helpers y utilidades
├── Tests/
│   ├── ztestVentas.prg
│   └── ...
└── _dovfp_excluidos/     && tests deshabilitados
```

---

## Objetivos

| Métrica | Valor |
|---|---|
| Tiempo por test | < 1 segundo (límite 2s) |
| Cobertura mínima | 50% |
| Cobertura target | 70% |
| Cobertura ideal | 85% |

Prioridad de cobertura: lógica de negocio crítica → validaciones → cálculos financieros → integraciones → UI.

---

## Testing paralelo

`dovfp test` paraleliza a nivel de clase. Clases con el mismo `@test_collection` se serializan; todos los workers comparten una única DB SQL Server.

### Reglas obligatorias

- **Códigos únicos**: usar `Sys(2015)` para IDs de test — nunca códigos fijos que puedan colisionar entre workers
- **Cleanup scoped**: `Delete From Tabla Where id = lcIdPropio` — nunca `Delete From Tabla` global
- **`@test_collection`**: si dos clases comparten tabla o fixture, asignarles la misma collection para serializarlas
- **Tiempo**: anclar a `goLibrerias.ObtenerFechaHora()` — no usar fechas fijas en datos de test
- **Independencia**: ningún test puede depender del orden de ejecución de otro
- **No asumir velocidad**: bajo carga un test puede tardar varias veces más de lo normal

### Etiquetas disponibles

```foxpro
@test_collection:NombreGrupo   && serializa clases que comparten tabla/fixture
@skip:motivo                   && deshabilita un test — siempre documentar el motivo
```

### Validación

Correr varias corridas consecutivas antes de considerar un test estable (el equipo usa 4 corridas verdes).

---

## Reglas

- Un concepto por test, tests independientes
- Siempre Arrange-Act-Assert
- Cleanup en `TearDown`
- No dependencias entre tests
- No tests sin assertions
- No dejar tests comentados
