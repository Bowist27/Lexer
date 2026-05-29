# Apuntes de Compiladores: Ejercicio Autómata SLR

## 1. Gramática y Producciones
Se presenta la gramática original aumentada con el estado $S'$.

**Gramática Aumentada:**
$S' \rightarrow S$
1. $S \rightarrow AB$
2. $S \rightarrow A$
3. $A \rightarrow aCc$
4. $C \rightarrow bbC$
5. $C \rightarrow b$
6. $B \rightarrow cd$

---

## 2. Conjuntos FIRST y FOLLOW

### Conjuntos FIRST
* **$\text{First}(C)$** = $\{ b \}$
* **$\text{First}(B)$** = $\{ c \}$
* **$\text{First}(A)$** = $\{ a \}$
* **$\text{First}(S)$** = $\{ a \}$

### Conjuntos FOLLOW
* **$\text{Follow}(S)$** = $\{ \$ \}$ *(Símbolo inicial)*
* **$\text{Follow}(A)$** = $\{ c, \$ \}$ *(Le sigue B, cuyo First es 'c', y al estar al final de S->A, hereda Follow(S))*
* **$\text{Follow}(B)$** = $\{ \$ \}$ *(Está al final de S->AB, hereda Follow(S))*
* **$\text{Follow}(C)$** = $\{ c \}$ *(En A->aCc, a C le sigue explícitamente el terminal 'c')*

---

## 3. Autómata SLR (Colección de Ítems)
El autómata consta de 12 estados ($I_0$ a $I_{11}$) detallados a continuación:

* **$I_0$ (Estado Inicial):**
  $S' \rightarrow \cdot S$
  $S \rightarrow \cdot AB$
  $S \rightarrow \cdot A$
  $A \rightarrow \cdot aCc$

**Transiciones Principales:**
* Desde **$I_0$** $\xrightarrow{S}$ **$I_1$**: $S' \rightarrow S \cdot$
* Desde **$I_0$** $\xrightarrow{A}$ **$I_2$**: $S \rightarrow A \cdot B$ | $S \rightarrow A \cdot$ | $B \rightarrow \cdot cd$
* Desde **$I_0$** $\xrightarrow{a}$ **$I_3$**: $A \rightarrow a \cdot Cc$ | $C \rightarrow \cdot bbC$ | $C \rightarrow \cdot b$

**Rama B (desde $I_2$):**
* Desde **$I_2$** $\xrightarrow{B}$ **$I_4$**: $S \rightarrow AB \cdot$
* Desde **$I_2$** $\xrightarrow{c}$ **$I_5$**: $B \rightarrow c \cdot d$
* Desde **$I_5$** $\xrightarrow{d}$ **$I_8$**: $B \rightarrow cd \cdot$

**Rama A y C (desde $I_3$):**
* Desde **$I_3$** $\xrightarrow{C}$ **$I_6$**: $A \rightarrow aC \cdot c$
* Desde **$I_3$** $\xrightarrow{b}$ **$I_7$**: $C \rightarrow b \cdot bC$ | $C \rightarrow b \cdot$
* Desde **$I_6$** $\xrightarrow{c}$ **$I_9$**: $A \rightarrow aCc \cdot$
* Desde **$I_7$** $\xrightarrow{b}$ **$I_{10}$**: $C \rightarrow bb \cdot C$ | $C \rightarrow \cdot bbC$ | $C \rightarrow \cdot b$
* Desde **$I_{10}$** $\xrightarrow{C}$ **$I_{11}$**: $C \rightarrow bbC \cdot$

---

## 4. Tabla de Análisis SLR

| Estado | a | b | c | d | $ | S | A | B | C |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **0** | s3 | | | | | 1 | 2 | | |
| **1** | | | | | acc | | | | |
| **2** | | | s5 | | r2 | | | 4 | |
| **3** | | s7 | | | | | | | 6 |
| **4** | | | | | r1 | | | | |
| **5** | | | | s8 | | | | | |
| **6** | | | s9 | | | | | | |
| **7** | | s10 | r5 | | | | | | |
| **8** | | | | | r6 | | | | |
| **9** | | | r3 | | r3 | | | | |
| **10** | | s7 | | | | | | | 11 |
| **11** | | | r4 | | | | | | |

> **Conclusión sobre la gramática:**
> Sí es una gramática SLR válida porque no hay más de una acción por celda (no muestra conflictos de tipo *Shift/Reduce* ni *Reduce/Reduce*).
