# Apuntes de Compiladores: Análisis Sintáctico y Gramáticas

## 1. Formato Backus-Naur (BNF)
- Se utiliza para **documentar gramáticas** de manera que evolucionen con el tiempo.
- La ventaja principal es que si se necesita corregir algo en la estructura de una gramática, simplemente se actualiza esta documentación.

## 2. Componentes de una Gramática
*Ejemplo clásico de una Gramática LR:*
$$E \rightarrow E + T \mid T$$
$$T \rightarrow T * F \mid F$$
$$F \rightarrow (E) \mid \mathbf{id}$$

Una gramática se compone de los siguientes elementos:
* **Terminales:** Son los símbolos básicos (hojas) de los que están formados los strings o cadenas. (Ej. `id`, `+`, `*`, `(`, `)`).
* **No terminales:** Son variables que hacen referencia a un conjunto de cadenas (agrupaciones abstractas que reconoce el analizador).
* **Símbolo Inicial:** El no terminal por el cual arranca la evaluación (generalmente la primera producción).
* **Producciones:** Son las reglas de reemplazo y se componen de:
    * **Cabeza:** El *No terminal* ubicado en el lado izquierdo.
    * **Símbolo de derivación:** Representado con una flecha (`->` o $\rightarrow$).
    * **Cuerpo:** El lado derecho, que consiste en una mezcla de más *terminales* y *no terminales*.

## 3. Rol del Analizador Sintáctico (Parser)
- **Regla fundamental:** Si la gramática tiene ambigüedad, no funciona ninguno de manera determinista.
- **Existen 3 tipos principales de analizadores sintácticos:**
    1.  **Universal:**
        * No importa qué tipo de gramática se le introduzca, son las herramientas más poderosas.
        * *Desventaja:* Tienden a tener un costo computacional muy alto (complejidad cuadrática o cúbica).
        * *Ejemplos:* Algoritmo de *Cocke-Younger-Kasami (CYK)*, Algoritmo de *Earley*.
    2.  **Top-Down (Descendente):** Construyen el árbol sintáctico desde la raíz bajando hacia las hojas.
    3.  **Bottom-Up (Ascendente):**
        * Leen de Izquierda a Derecha (*Left to Right*).
        * Son mucho más robustos y permiten manejar gramáticas más complejas.
        * *Desventaja:* Entre más compleja sea la gramática, más difíciles se vuelven de manejar e implementar.

## 4. Derivaciones
Es el proceso de reemplazar un "no terminal" por el cuerpo de su producción.

### Derivación por la Izquierda (Leftmost - $lm$)
Siempre se busca y se reemplaza el no terminal que esté más a la izquierda en la expresión.
*Ejemplo:*
$$E \xrightarrow{lm} -E \xrightarrow{lm} -(E) \xrightarrow{lm} -(E+E) \xrightarrow{lm} -(id+E) \xrightarrow{lm} -(id+id)$$

### Derivación por la Derecha (Rightmost - $rm$)
Siempre se busca y se reemplaza el no terminal que esté más a la derecha en la expresión.
*Ejemplo:*
$$E \xrightarrow{rm} E + E \xrightarrow{rm} E + (E) \xrightarrow{rm} E + (E + E) \xrightarrow{rm} E + (E + id) \dots$$

### Ejemplo Completo: Gramática $S \rightarrow SS+ \mid SS* \mid a$
Generando la cadena `aa+a*`:
* **Derivación Izquierda:** $S \xrightarrow{lm} SS* \xrightarrow{lm} SS+S* \xrightarrow{lm} aS+S* \xrightarrow{lm} aa+S* \xrightarrow{lm} aa+a*$
* **Derivación Derecha:** $S \xrightarrow{rm} SS* \xrightarrow{rm} Sa* \xrightarrow{rm} SS+a* \xrightarrow{rm} Sa+a* \xrightarrow{rm} aa+a*$

> **Nota sobre ambigüedad:** Esta gramática *no es ambigua* porque los operadores (`+`, `*`) están forzados a aparecer de la forma `SS+` o `SS*`. Al no haber dos posibles caminos matemáticos para estructurarlo, solo existe una agrupación matemática válida.

## 5. Árboles Sintácticos y Precedencia
La estructura gráfica del árbol es literalmente un **mapa de prioridad matemática** dictado por la gramática. Una gramática no ambigua evita que existan 2 caminos distintos para formar el mismo árbol.

**Analogía del edificio (Cimientos vs. Techo):**
* **Mayor Precedencia (Nivel más profundo / Cimientos):**
    * Son las hojas o nodos externos (hasta abajo del árbol).
    * Aquí viven las multiplicaciones (`*`), números negativos (`-`) y los paréntesis `()`.
    * Al ser los cimientos, la computadora *obligatoriamente* los tiene que calcular **primero**.
* **Menor Precedencia (Nivel superficial / Techo):**
    * Es la raíz o los nodos internos superiores (hasta arriba del árbol).
    * Aquí suelen vivir las sumas (`+`) o restas (`-`) principales.
    * Al ser el techo, tienen que esperar pacientemente a que todo lo de abajo esté resuelto, por lo que se evalúan al **final**.

**Ejemplo de árbol evaluando `3 + (4 * 5 + 2)`:**
1.  **Nivel más profundo (Mayor precedencia):** La rama dentro de los paréntesis efectúa $4 * 5 = 20$. El compilador está obligado a resolver esto al fondo.
2.  **Nivel medio (Precedencia intermedia):** La suma toma el $20$ y le suma el $2$. ($20 + 2 = 22$). El nodo del paréntesis actuó como un "escudo", obligando a que esta operación se hundiera en el árbol para resolverse antes.
3.  **La Raíz (Menor precedencia):** La rama derecha devolvió el $22$. El número $3$ (rama izquierda) usó las reglas de transición simple como un "elevador" para subir hasta la raíz. Finalmente, la raíz ejecuta la suma principal: $3 + 22 = 25$.

## 6. Recursividad en Gramáticas
* **¿Qué significa que sea recursiva?** Que el "No Terminal" de la cabeza aparece dentro de su propia producción.
* Si como *primer* elemento del cuerpo de la producción tienes a la misma cabeza, se indica que es una gramática **recursiva por la izquierda**.

## 7. Manejo de Errores Sintácticos
Un error sintáctico ocurre cuando se rompe la estructura esperada (por ejemplo, tokens invertidos o faltantes).

**Tipos de errores en programación:**
1. Léxicos (símbolos mal escritos).
2. Sintácticos (estructura mal formada).
3. Semánticos (tipos incompatibles, variables no declaradas).
4. Lógicos (el código corre pero hace mal el cálculo).

**Metas del Analizador ante un Error Sintáctico:**
1. Reportar el error lo más cercano posible al lugar donde ocurrió.
2. Recuperarse del error rápidamente para seguir analizando el resto del archivo.
3. Agregar la mínima sobrecarga de procesamiento a los programas que sí están correctos.

**Estrategias de Recuperación:**
* **Modo Pánico:** Al encontrar el error, ignora (salta) tokens hasta llegar a un "token de sincronización" seguro (ej. en `C`, desechar tokens hasta toparse con una llave de cierre `}` o un punto y coma `;`).
* **Nivel de Frase:** Intenta reemplazar un prefijo de la entrada restante por alguna cadena lógica que permita continuar. (Ej. Si se lee `acb` pero la gramática exige `abc`, se busca una permutación o conmutación para arreglar la lectura localmente).
* **Producciones Erróneas:** Se agregan producciones artificiales a la gramática que contemplen los errores más comunes de los programadores, permitiendo que el analizador atrape el error de manera controlada y dé un mensaje de ayuda exacto.
