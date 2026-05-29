# Apuntes de Compiladores: Tabla de Análisis Predictivo y Evaluación de Cadenas

## 1. Problema y Gramática a Evaluar
Para la siguiente gramática, se pide generar la tabla de análisis predictivo (Algoritmo 4.31) y verificar si se reconocen las cadenas **bda** y **dba** (Algoritmo 4.34).

**Gramática:**
1.  $S \rightarrow A\ a$
2.  $A \rightarrow B\ D$
3.  $B \rightarrow b \mid \epsilon$
4.  $D \rightarrow d \mid \epsilon$

---

## 2. Obtención de Conjuntos FIRST y FOLLOW

### Conjuntos FIRST
El conjunto FIRST de un no terminal es el conjunto de terminales con los que pueden empezar las cadenas derivadas de él.
* **$\text{First}(B)$** = $\{b, \epsilon\}$
* **$\text{First}(D)$** = $\{d, \epsilon\}$
* **$\text{First}(A)$** = $\text{First}(B) \cup \text{First}(D)$ *(debido a que $B$ puede ser $\epsilon$, pasamos a evaluar $D$. Si ambos pueden ser $\epsilon$, incluimos $\epsilon$)* = $\{b, d, \epsilon\}$
* **$\text{First}(S)$** = $\text{First}(A)$ *(como $A$ puede ser $\epsilon$, tomamos también el siguiente terminal que es 'a')* = $\{b, d, a\}$

### Conjuntos FOLLOW
El conjunto FOLLOW de un no terminal es el conjunto de terminales que pueden aparecer inmediatamente a su derecha en alguna forma sentencial.
* **$\text{Follow}(S)$** = $\{\$\}$ *(Símbolo inicial)*
* **$\text{Follow}(A)$** = $\{a\}$ *(En $S \rightarrow A\ a$, lo que le sigue a $A$ es el terminal 'a')*
* **$\text{Follow}(B)$** = $\text{First}(D) \setminus \{\epsilon\} \cup \text{Follow}(A)$ *(Lo que sigue de $B$ es $D$. $\text{First}(D)$ es 'd'. Como $D$ puede ser $\epsilon$, también hereda $\text{Follow}(A)$)* = $\{d, a\}$
* **$\text{Follow}(D)$** = $\text{Follow}(A)$ *(Está al final de la producción $A \rightarrow B\ D$, hereda lo que le sigue a $A$)* = $\{a\}$

---

## 3. Construcción de la Tabla de Análisis Predictivo (Algoritmo 4.31)

**Reglas de llenado:**
1. Por cada terminal $t$ en $\text{First}(\alpha)$, agregar $X \rightarrow \alpha$ en $M[X, t]$.
2. Si $\epsilon$ está en $\text{First}(\alpha)$, por cada terminal $t$ en $\text{Follow}(X)$, agregar $X \rightarrow \alpha$ en $M[X, t]$.
3. Si $\epsilon \in \text{First}(\alpha)$ y $\$ \in \text{Follow}(X)$, agregar en $M[X, \$]$.

**Tabla Resultante:**

| No Terminal | a | b | d | $ |
| :--- | :--- | :--- | :--- | :--- |
| **S** | $S \rightarrow A\ a$ | $S \rightarrow A\ a$ | $S \rightarrow A\ a$ | |
| **A** | $A \rightarrow B\ D$ | $A \rightarrow B\ D$ | $A \rightarrow B\ D$ | |
| **B** | $B \rightarrow \epsilon$ | $B \rightarrow b$ | $B \rightarrow \epsilon$ | |
| **D** | $D \rightarrow \epsilon$ | | $D \rightarrow d$ | |

---

## 4. Evaluación de Cadenas con Pila (Algoritmo 4.34)

### Evaluación de la cadena: "bda" (Exitosa)
| Pila | Entrada | Acción |
| :--- | :--- | :--- |
| $\$ S$ | bda $\$$ | $M[S, b] = S \rightarrow A\ a$ |
| $\$ a A$ | bda $\$$ | $M[A, b] = A \rightarrow B\ D$ |
| $\$ a D B$ | bda $\$$ | $M[B, b] = B \rightarrow b$ |
| $\$ a D b$ | bda $\$$ | $\text{Match(b)}$ *(Tope = entrada = b)* |
| $\$ a D$ | da $\$$ | $M[D, d] = D \rightarrow d$ |
| $\$ a d$ | da $\$$ | $\text{Match(d)}$ *(Tope = entrada = d)* |
| $\$ a$ | a $\$$ | $\text{Match(a)}$ *(Tope = entrada = a)* |
| $\$$ | $\$$ | **Finalizado / Aceptado ✓** |

### Evaluación de la cadena: "dba" (Error)
| Pila | Entrada | Acción |
| :--- | :--- | :--- |
| $\$ S$ | dba $\$$ | $M[S, d] = S \rightarrow A\ a$ |
| $\$ a A$ | dba $\$$ | $M[A, d] = A \rightarrow B\ D$ |
| $\$ a D B$ | dba $\$$ | $M[B, d] = B \rightarrow \epsilon$ *(B desaparece, no se mete nada)* |
| $\$ a D$ | dba $\$$ | $M[D, d] = D \rightarrow d$ |
| $\$ a d$ | dba $\$$ | $\text{Match(d)}$ *(Tope = entrada = d)* |
| $\$ a$ | ba $\$$ | **ERROR ❌** |

> **Análisis del Error:** El tope de la pila es `a`, pero la entrada esperando es `b`. Como $a \neq b$, se rechaza la cadena y se lanza un error sintáctico.
