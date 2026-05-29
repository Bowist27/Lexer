# Apuntes de Compiladores: Autómata LR y Tabla de Análisis

## 1. Gramática y Producciones
Para construir el autómata LR, primero se enumera la gramática (incluyendo la producción aumentada $S \rightarrow E$ para el símbolo inicial).

**Gramática Aumentada:**
$S \rightarrow E$
1. $E \rightarrow E + T$
2. $E \rightarrow T$
3. $T \rightarrow T * F$
4. $T \rightarrow F$
5. $F \rightarrow \text{id}$
6. $F \rightarrow (E)$

---

## 2. Conjuntos FIRST y FOLLOW
*(Nota: En los apuntes escritos a mano, el símbolo que parece una 'C' es en realidad el paréntesis de apertura '(').*

* **$\text{First}(E)$** = $\{ \text{id}, ( \}$
* **$\text{First}(T)$** = $\{ \text{id}, ( \}$
* **$\text{First}(F)$** = $\{ \text{id}, ( \}$

* **$\text{Follow}(E)$** = $\{ \$, +, ) \}$
* **$\text{Follow}(T)$** = $\{ *, +, \$, ) \}$
* **$\text{Follow}(F)$** = $\{ *, +, \$, ) \}$

---

## 3. Autómata LR(0) (Colección de Ítems)
El diagrama muestra la transición de estados (ítems LR) al leer distintos símbolos (terminales y no terminales).

* **$I_0$ (Estado Inicial):**
  $S \rightarrow \cdot E$
  $E \rightarrow \cdot E + T$
  $E \rightarrow \cdot T$
  $T \rightarrow \cdot T * F$
  $T \rightarrow \cdot F$
  $F \rightarrow \cdot \text{id}$
  $F \rightarrow \cdot (E)$

**Transiciones desde $I_0$:**
* $\xrightarrow{E}$ **$I_1$**: $S \rightarrow E \cdot$ | $E \rightarrow E \cdot + T$
* $\xrightarrow{T}$ **$I_2$**: $E \rightarrow T \cdot$ | $T \rightarrow T \cdot * F$
* $\xrightarrow{F}$ **$I_3$**: $T \rightarrow F \cdot$
* $\xrightarrow{\text{id}}$ **$I_4$**: $F \rightarrow \text{id} \cdot$
* $\xrightarrow{(}$ **$I_5$**: $F \rightarrow ( \cdot E)$ *(más la cerradura de E)*

**Otras Transiciones Clave (Mostradas en el diagrama):**
* **Desde $I_1$** $\xrightarrow{+}$ **$I_6$**: $E \rightarrow E + \cdot T$ *(y cerradura de T)*
* **Desde $I_6$** $\xrightarrow{T}$ **$I_9$**: $E \rightarrow E + T \cdot$ | $T \rightarrow T \cdot * F$
* **Desde $I_2$ y $I_9$** $\xrightarrow{*}$ **$I_7$**: $T \rightarrow T * \cdot F$ *(y cerradura de F)*
* **Desde $I_5$** $\xrightarrow{E}$ **$I_8$**: $F \rightarrow (E \cdot)$ | $E \rightarrow E \cdot + T$
* **Desde $I_8$** $\xrightarrow{+}$ **$I_6$**
* **Desde $I_8$** $\xrightarrow{)}$ **$I_{11}$**: $F \rightarrow (E) \cdot$

---

## 4. Tabla de Análisis LR (Shift / Reduce)
*(Nota: En la imagen tiene el título "Tabla de analisis predictivo", pero por su estructura con desplazamientos **'s' (shift)** y reducciones **'r' (reduce)**, se trata de una tabla de análisis ascendente SLR/LR).*

| Estado | id | ( | ) | + | * | $ | E | T | F |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **0** | s4 | s5 | | | | | 1 | 2 | 3 |
| **1** | | | | s6 | | | | | |
| **2** | | | r2 | r2 | | r2 | | | |
| **3** | | | r4 | r4 | r4 | r4 | | | |
| **4** | | | r5 | r5 | r5 | r5 | | | |
| **5** | s4 | s5 | | | | | 8 | 2 | 3 |
| **6** | s4 | s5 | | | | | | 9 | 3 |
| **7** | s4 | s5 | | | | | | | 10 |
| **8** | | | s11 | s6 | | | | | |
| **9** | | | r1 | r1 | s7 | r1 | | | |
| **10** | | | r3 | r3 | r3 | r3 | | | |
| **11** | | | r6 | r6 | r6 | r6 | | | |

**Simbología de la Tabla:**
* **sX:** Shift (Desplazar) al estado X.
* **rX:** Reduce (Reducir) usando la producción número X de la gramática.
* **Números solos (en columnas E, T, F):** Representan la función *Goto* (Ir al estado X) después de reducir un No Terminal.
