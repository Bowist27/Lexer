# 🎯 ¿Qué te pide el profesor? — Explicación paso a paso

## El panorama general: ¿Qué es este proyecto?

Tu profesor te está pidiendo que construyas un **compilador/intérprete** para un **mini-lenguaje de programación tipo Logo (Turtle Graphics)**. Es como crear tu propio "mini-Python" que entiende comandos para dibujar en pantalla.

> [!IMPORTANT]
> No es un proyecto desde cero. El profesor te da una **base de código** (el "esqueleto") y tú tienes que **completarlo y extenderlo**. Piensa en ello como un rompecabezas donde ya tienes las orillas y debes llenar el centro.

---

## ¿Cómo se conecta con tus apuntes?

Tu materia ha seguido esta progresión lógica:

| Semana | Tema en tus apuntes | Cómo aplica al proyecto |
|:---|:---|:---|
| **Semana 1** | Análisis Sintáctico, BNF, derivaciones, árboles | Entender la **gramática** (`grammar.txt`) y cómo el Parser la implementa |
| **Semana 2** | Factorización, FIRST, FOLLOW, tablas predictivas | La gramática del profe ya está adaptada para **Top-Down** (sin recursión izquierda). Los conjuntos FIRST que ves en `Parser.py` son exactamente lo que aprendiste |
| **Semana 3** | Autómatas LR, tablas SLR | Contexto teórico. El parser del profe usa **Recursive Descent** (Top-Down), no Bottom-Up, pero entender ambos enfoques te da la base |

---

## Los 7 archivos del profesor explicados uno por uno

### 1. `grammar.txt` — La "receta" del lenguaje
**¿Qué es?** La gramática BNF que define qué programas son válidos en este lenguaje.

**Estructura del lenguaje (lo que tu compilador debe aceptar):**
```
<program> ::= <declaration-sequence> <statement-sequence>
```

Un programa válido tiene:
- **Declaraciones** al inicio (usando `VAR` para crear variables)
- **Sentencias** después (asignaciones con `:=` o impresiones con `PRINT`)

> [!NOTE]
> La gramática está incompleta en el archivo (se corta en `<expression>`). Esto es intencional — **tú debes deducir el resto** viendo cómo el Parser lo implementa, o el profesor te lo completará.

---

### 2. `Type.py` — Los tipos de datos
**¿Qué es?** Un enum ultra simple que define los tipos de datos que maneja el lenguaje.

```python
class Type(IntEnum):
    NUMBER = 0    # Números (enteros/decimales)
    BOOLEAN = 1   # Verdadero/Falso (#T / #F)
```

**Tu rol:** Este archivo probablemente ya está completo. Solo son 2 tipos.

---

### 3. `Lexer.py` — El analizador léxico (tokenizador)
**¿Qué es?** Lee el archivo de código fuente carácter por carácter y lo convierte en **tokens** (pedazos con significado).

**Ejemplo de lo que hace:**
```
Entrada:  "VAR x, y"
Salida:   [Token(VAR), Token(ID,"x"), Token(','), Token(ID,"y")]
```

**Componentes clave:**
- **`Tag` (enum):** Define TODOS los tipos de token posibles:
  - Operadores: `>=`, `<=`, `<>`, `:=`
  - Palabras reservadas: `VAR`, `FORWARD`, `BACKWARD`, `LEFT`, `RIGHT`, `PRINT`, `WHILE`, `IF`, etc.
  - Valores: `ID` (identificadores), `NUMBER`, `STRING`, `TRUE` (#T), `FALSE` (#F)
  
- **`Token` (clase):** Un par de (tipo, valor). Ejemplo: `Token(Tag.ID, "miVariable")`

- **`Lexer` (clase):** Lee el archivo con doble buffer y tiene el método `scan()` que retorna el siguiente token.

> [!IMPORTANT]
> Fíjate en los Tags — este lenguaje tiene comandos de **Turtle Graphics**: `FORWARD`, `BACKWARD`, `LEFT`, `RIGHT`, `SETX`, `SETY`, `CIRCLE`, `ARC`, `PENUP`, `PENDOWN`, `COLOR`, `PENWIDTH`, `HOME`. Esto te dice que el lenguaje final es para **dibujar con una tortuga** (como Logo).

**Tu rol:** El Lexer probablemente está mayormente completo (el profe lo lleva trabajando un mes). Pero revisa bien el método `scan()` — puede que necesites agregar el reconocimiento de algunos tokens.

---

### 4. `Parser.py` — El analizador sintáctico (¡EL CORAZÓN DEL PROYECTO!)
**¿Qué es?** Toma la secuencia de tokens del Lexer y verifica que sigan las reglas de la gramática. Usa la técnica de **Recursive Descent** (Descenso Recursivo).

**¿Cómo funciona el Descenso Recursivo?** Cada regla de la gramática se convierte en una **función de Python**:
- `<program>` → función `program()`
- `<expression>` → función `expression()`
- `<statement>` → función `statement()`

**Conexión directa con tus apuntes:**
- Los conjuntos `self.firstPrimaryExpression`, `self.firstUnaryExpression`, etc. son literalmente los **conjuntos FIRST** que calculaste en Semana 2.
- El método `check(tag)` es el equivalente al **Match** de tu tabla predictiva de pila (Semana 2, Algoritmo 4.34).

**Lo que ves en el código:**
```python
# Estos son los FIRST sets que calculaste en tus apuntes
self.firstPrimaryExpression = set((Tag.ID, Tag.NUMBER, Tag.TRUE, Tag.FALSE, ord('(')))
self.firstUnaryExpression = self.firstPrimaryExpression.union(set((ord('-'), ord('!'))))
self.firstMultiplicativeExpression = self.firstUnaryExpression
self.firstAdditiveExpression = self.firstMultiplicativeExpression
```

> [!WARNING]
> El archivo del profesor está **incompleto** (se corta después de la función `check`). Aquí está la parte principal de tu trabajo: **implementar las funciones de parsing para cada regla de la gramática**.

**Tu rol — lo que probablemente debes programar:**
1. `program()` — Parsea `<declaration-sequence>` seguido de `<statement-sequence>`
2. `declarationSequence()` — Parsea declaraciones `VAR id, id, id`
3. `statementSequence()` — Parsea una secuencia de sentencias
4. `statement()` — Decide si es asignación o PRINT (o cualquier otro comando)
5. `assignmentStatement()` — Parsea `id := expression`
6. `textStatement()` — Parsea `PRINT(expression)`
7. `expression()` / `additiveExpression()` / `multiplicativeExpression()` / `unaryExpression()` / `primaryExpression()` — La jerarquía de expresiones matemáticas

---

### 5. `SymbolTable.py` — La tabla de símbolos
**¿Qué es?** Guarda las variables declaradas y sus valores. Es como un diccionario que el compilador consulta cuando ve una variable.

```python
class SymbolTable:
    table = {}        # Diccionario variable → valor
    previous = None   # Enlace al scope (ámbito) anterior
```

**Conceptos clave:**
- `lookup(variable)` busca si una variable existe
- `previous` permite tener **scopes anidados** (como variables dentro de un `while` vs fuera)

**Tu rol:** Probablemente necesitas completar:
- Un método `insert(variable, value)` para agregar variables
- Manejar la lógica de scopes encadenados (buscar en el scope actual, si no está, buscar en el anterior)

---

### 6. `Translator.py` — El intérprete / traductor (evaluación)
**¿Qué es?** Define los **nodos del árbol de sintaxis abstracta (AST)** y cómo se evalúan. Es donde el código pasa de "entender la estructura" a "ejecutar las instrucciones".

**Jerarquía de nodos:**
```
Node (base)
├── Numeric (nodos que retornan números)
│   └── Number (un literal numérico)
├── Logic (nodos que retornan booleanos)
└── Void (nodos que no retornan valor, como PRINT)
```

**Tu rol:** Este es el segundo bloque grande de trabajo. Debes crear clases para:
- `Variable` — Evalúa buscando el valor en la SymbolTable
- `BinaryOp` / `Add` / `Sub` / `Mul` / `Div` — Operaciones aritméticas
- `UnaryMinus` — Negación numérica (-x)
- `Assignment` — Asigna valor a una variable
- `Print` — Imprime un valor
- Y posiblemente: `While`, `If`, `Comparison`, etc.

---

### 7. `main.py` — El punto de entrada
```python
from Lexer import *
from Parser import *

if __name__ == '__main__':
    parser = Parser("test_cases/bad/input04.txt")
```

**Lo que revela:** El profe prueba con archivos `.txt` que contienen programas escritos en el mini-lenguaje. Hay carpetas `test_cases/bad/` (programas con errores que deben ser rechazados) y probablemente `test_cases/good/` (programas válidos).

---

## 📋 Lo que debes hacer paso a paso

### Paso 1: Entender la gramática completa
- Lee `grammar.txt` y compléta mentalmente lo que falta (la parte de expresiones)
- La jerarquía de expresiones sigue exactamente lo de tus apuntes Semana 1 (precedencia de operadores):
  ```
  expression → additiveExpr
  additiveExpr → multiplicativeExpr (('+' | '-') multiplicativeExpr)*
  multiplicativeExpr → unaryExpr (('*' | '/' | MOD) unaryExpr)*
  unaryExpr → ('-' | '!') unaryExpr | primaryExpr
  primaryExpr → ID | NUMBER | TRUE | FALSE | '(' expression ')'
  ```

### Paso 2: Completar el Lexer (si está incompleto)
- Verificar que el método `scan()` reconoce todos los tokens necesarios
- Crear archivos de prueba simples y probar que tokeniza correctamente

### Paso 3: Implementar el Parser (Descenso Recursivo)
- Crear una función por cada regla de la gramática
- Usar los conjuntos FIRST para decidir qué producción aplicar
- Usar `check(tag)` para consumir (hacer Match) tokens esperados
- Reportar errores con `error()`

### Paso 4: Completar la Tabla de Símbolos
- Agregar `insert()` para guardar variables
- Verificar que `lookup()` busca en scopes encadenados

### Paso 5: Implementar el Translator (AST + evaluación)
- Crear clases para cada tipo de nodo
- Implementar `eval(env)` en cada una
- Conectar el Parser con el Translator (que el Parser cree nodos del AST mientras parsea)

### Paso 6: Probar con casos del profesor
- Crear tus propios archivos de prueba en `test_cases/`
- Programas correctos que deben ejecutar sin error
- Programas incorrectos que deben lanzar errores claros

---

## 🧠 Resumen visual del flujo completo

```
Archivo fuente (.txt)          "VAR x\nx := 3 + 5\nPRINT(x)"
        │
        ▼
   ┌─────────┐
   │  LEXER  │   Análisis Léxico
   └────┬────┘
        │         [VAR, ID("x"), ID("x"), ASSIGN, NUM(3), '+', NUM(5), PRINT, '(', ID("x"), ')']
        ▼
   ┌─────────┐
   │ PARSER  │   Análisis Sintáctico (Descenso Recursivo)
   └────┬────┘
        │         Árbol de Sintaxis Abstracta (AST)
        ▼
   ┌────────────┐
   │ TRANSLATOR │   Evaluación / Ejecución
   └────────────┘
        │
        ▼
     Resultado:  8
```

> [!TIP]
> **Consejo de estudio:** No intentes hacer todo de golpe. Empieza por entender cada pieza por separado. Primero juega con el Lexer solo (dale archivos y ve qué tokens produce). Luego el Parser (solo verificar estructura, sin evaluar). Al final conecta todo con el Translator.
