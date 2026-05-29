# Apuntes de Compiladores: Gramáticas, Factorización, FIRST y FOLLOW

## 1. Análisis Léxico vs Sintáctico y Ambigüedad
* **Expresiones Regulares:** Se utilizan en el análisis léxico.
* **Ambigüedad:** Ocurre cuando existen más de 2 caminos (árboles de derivación) para llegar a la misma expresión. Existen analizadores que ayudan a detectar esto.
* **Problema del "Dangling Else" (Else-colgante):** Es un problema clásico de ambigüedad en el que el compilador no sabe a qué `if` pertenece un `else` cuando hay `if`s anidados sin llaves.

---

## 2. Factorización por la Izquierda (Left Factoring)
Consiste en **transformar una gramática compleja** en una que el compilador pueda procesar de forma rápida y determinista (útil para analizadores *Top-Down*).
* **Similitud matemática:** Funciona igual que extraer un factor común en álgebra.
    * *Ejemplo:* $ab + ac \xrightarrow{\text{factorización}} a(b+c)$

**Fórmula de Factorización:**
Si tenemos el problema de que varias producciones empiezan con el mismo prefijo:
$$A \rightarrow \alpha\beta_1 \mid \alpha\beta_2 \mid \dots \mid \alpha\beta_n \mid \gamma$$
Se factoriza extrayendo $\alpha$ (el prefijo común) y creando un nuevo no terminal $A'$:
$$A \rightarrow \alpha A' \mid \gamma$$
$$A' \rightarrow \beta_1 \mid \beta_2 \mid \dots \mid \beta_n$$

**Glosario de la fórmula:**
| Símbolo | Significado |
| :--- | :--- |
| **$A$** | Instrucción (No Terminal) que el compilador intenta procesar. |
| **$\alpha$** | **Prefijo Común:** Secuencia de palabras o símbolos idéntica en varias opciones. |
| **$\beta_1, \beta_2$** | Diferentes finales posibles que le siguen al prefijo común. |
| **$\gamma$** | Cualquier otra opción (producción) que NO empieza con el prefijo $\alpha$. |

> *El problema que resuelve:* El compilador no puede adivinar qué camino tomar si ambos empiezan igual, "tiene que ser una u otra".

---

## 3. Ejercicios de Factorización y Recursividad (Ejercicio 4.3.1)

### Variante 1: $S \rightarrow SS+ \mid SS* \mid a$
* **a) Factorización por la izquierda:**
    El prefijo común es $SS$.
    $S \rightarrow SSS' \mid a$
    $S' \rightarrow + \mid *$
* **b) ¿Es apta para Top-Down (Descendente)?**
    *No es apta*. Aunque se solucionó la ambigüedad del prefijo, sigue teniendo el problema de **recursión por la izquierda** ($S$ produce algo que empieza con $S$).
* **c) Eliminar la recursión por la izquierda:**
    A partir de la gramática factorizada ($S \rightarrow SSS' \mid a$), eliminamos la recursividad:
    $S \rightarrow aS''$
    $S'' \rightarrow SS'S'' \mid \epsilon$
    $S' \rightarrow + \mid *$
* **d) ¿Es apta ahora para Top-Down?**
    *Sí*, porque ya se resolvió tanto el prefijo común como la recursividad por la izquierda.

### Variante 2: $S \rightarrow 0S1 \mid 01$
* **a) Factorización por la izquierda:**
    El prefijo común es $0$.
    $S \rightarrow 0S'$
    $S' \rightarrow S1 \mid 1$
* **b) ¿Es apta para Top-Down?**
    *Sí*, porque solo hay una forma única de derivar; ya no hay conflictos.
* **c) Eliminar recursión por la izquierda:**
    *No se puede/no es necesario hacer*, porque la gramática factorizada no tiene recursividad por la izquierda.

### Variante 3: $S \rightarrow S(S)S \mid \epsilon$
* **a) Factorización por la izquierda:**
    *No se factoriza*, porque no hay producciones diferentes que compartan un mismo prefijo con quien hacer la comparación.
* **c) Eliminar recursión por la izquierda:**
    Aplicando la fórmula para eliminar recursión directa ($A \rightarrow A\alpha \mid \beta$):
    $S \rightarrow S'$  (Nota: como $\beta = \epsilon$, $\epsilon S' = S'$)
    $S' \rightarrow (S)SS' \mid \epsilon$

---

## 4. Generación de Conjuntos FIRST y FOLLOW
Evaluación para saber si una gramática es **LL(1)**.

### Gramática de ejemplo (Estructura de Clases tipo Java):
1.  $C \rightarrow P\ F\ \text{class}\ \text{id}\ X\ Y$ (Símbolo Inicial)
2.  $P \rightarrow \text{public} \mid \epsilon$
3.  $F \rightarrow \text{final} \mid \epsilon$
4.  $X \rightarrow \text{extends}\ \text{id} \mid \epsilon$
5.  $Y \rightarrow \text{implements}\ I \mid \epsilon$
6.  $I \rightarrow \text{id}\ J$
7.  $J \rightarrow ,\ I \mid \epsilon$

### Conjuntos FIRST
El conjunto FIRST incluye el primer símbolo terminal que puede derivarse de un No Terminal.
* **$\text{First}(J)$** = $\{ \text{,} \ , \epsilon \}$
* **$\text{First}(I)$** = $\{ \text{id} \}$
* **$\text{First}(Y)$** = $\{ \text{implements}, \epsilon \}$
* **$\text{First}(X)$** = $\{ \text{extends}, \epsilon \}$
* **$\text{First}(F)$** = $\{ \text{final}, \epsilon \}$
* **$\text{First}(P)$** = $\{ \text{public}, \epsilon \}$
* **$\text{First}(C)$** = $\{ \text{public}, \text{final}, \text{class} \}$
    *(Como $P$ puede ser $\epsilon$, pasa a evaluar $F$; como $F$ puede ser $\epsilon$, evalúa obligatoriamente el terminal `class`).*

### Conjuntos FOLLOW
**Reglas Básicas de FOLLOW:**
1.  Al símbolo inicial se le agrega el símbolo de fin de cadena ( **\$** ).
2.  Si tienes $A \rightarrow \alpha B \beta$, todo lo que esté en $\text{FIRST}(\beta)$ (excepto $\epsilon$) pasa a $\text{FOLLOW}(B)$.
3.  Si tienes $A \rightarrow \alpha B$ (o si $\beta$ puede derivar en $\epsilon$), todo lo de $\text{FOLLOW}(A)$ pasa a $\text{FOLLOW}(B)$.

**Resultados:**
* **$\text{Follow}(C)$** = $\{ \$ \}$ *(Símbolo inicial)*
* **$\text{Follow}(P)$** = $\{ \text{final}, \text{class} \}$ *(Lo que le sigue a P es F. FIRST(F) = `final`. Como F puede ser $\epsilon$, también hereda el siguiente símbolo: `class`)*
* **$\text{Follow}(F)$** = $\{ \text{class} \}$
* **$\text{Follow}(X)$** = $\{ \text{implements}, \$ \}$ *(Lo que le sigue a X es Y. FIRST(Y) = `implements`. Como Y puede ser $\epsilon$, hereda FOLLOW(C))*
* **$\text{Follow}(Y)$** = $\{ \$ \}$ *(Está al final de C, hereda FOLLOW(C))*
* **$\text{Follow}(I)$** = $\{ \$ \}$ *(Está al final de Y, y en la recursión con J)*
* **$\text{Follow}(J)$** = $\{ \$ \}$ *(Está al final de I, hereda FOLLOW(I))*
