# MINERÍA DE DATOS: MANUAL DE REFERENCIA TEÓRICO-PRÁCTICO (BÁSICO A AVANZADO)
## Algoritmo Apriori y Reglas de Asociación

El algoritmo **Apriori** es un método clásico de aprendizaje no supervisado diseñado para descubrir patrones de coaparición frecuente en grandes bases de datos transaccionales. Su objetivo es transformar registros históricos en reglas accionables ($A \rightarrow B$) aplicables a cualquier industria.

---

## 1. Fundamentos: De la Canasta de Supermercado a Cualquier Dominio

* **Diferencia con Clustering (K-Means / DBSCAN):** El clustering agrupa observaciones o individuos similares (*filas*) midiendo distancias matemáticas. Apriori descubre elementos o atributos que aparecen juntos (*columnas*) dentro de un mismo evento o contenedor.

### Anatomía de los Datos (Transacción, Ítem e Itemset)
* **Transacción (Contenedor):** Es el evento agrupador único (ej. un ticket de compra `order_id`, una sesión web, un paciente o el perfil de un cliente).
* **Ítem (Elemento):** Una variable binaria individual (ej. `Leche`, `Teclado_Mecánico`, `Fiebre`).
* **Itemset (Conjunto):** Un grupo de ítems sin orden ni repetición: $\{A, B\}$ es idéntico a $\{B, A\}$. Un conjunto con $k$ elementos se denomina **$k$-itemset**.
* **Matriz Booleana de Presencia ($0$ y $1$):** A Apriori clásico solo le importa si el ítem estuvo presente ($1$) o ausente ($0$) en la transacción, sin importar cuántas unidades se llevaron. Si la cantidad o el monto son relevantes para el negocio, se deben discretizar creando ítems categóricos (ej. `Compra_Mayorista`, `Ticket_Alto`).

### Reproducibilidad: ¿Dónde aplicar Apriori en el mundo real?

| Dominio / Industria | ¿Qué es una Transacción? | ¿Qué son los ítems? | Ejemplo de Regla Descubierta |
| :--- | :--- | :--- | :--- |
| **Retail / E-commerce** | Un ticket o carrito (`order_id`) | Productos comprados | $\{\text{Notebook}\} \rightarrow \{\text{Mouse, Funda}\}$ |
| **Streaming (Spotify/Netflix)** | Historial de un usuario | Películas o artistas vistos | $\{\text{Batman, Inception}\} \rightarrow \{\text{Interstellar}\}$ |
| **Salud / Diagnóstico** | Consulta médica de paciente | Síntomas y antecedentes | $\{\text{Fiebre, Dolor\_Muscular}\} \rightarrow \{\text{Gripe\_A}\}$ |
| **Banca / Clientes** | Ficha de un cliente (`user_id`) | Atributos discretizados | $\{\text{Edad\_25\_35, Zona\_Sur}\} \rightarrow \{\text{Mora\_Baja}\}$ |

---

## 2. Las 3 Métricas Fundamentales (Cálculo e Interpretación Universal)

Para evaluar qué combinaciones sirven y cuáles son puro ruido azaroso, el análisis utiliza tres medidas en cascada. Imaginemos tanto el caso de clase ($10$ ventas de supermercado) como una tienda de tecnología con $1.000$ facturas:

### A. Soporte (Popularidad o Relevancia Global)
Mide en qué proporción del total absoluto de transacciones ($N$) aparece un ítem o conjunto de ítems. Sirve para garantizar que el patrón tenga volumen comercial suficiente.

$$\text{Soporte}(X) = P(X) = \frac{\text{Transacciones que contienen } X}{\text{Total de transacciones } (N)}$$

* **Regla de cálculo para conjuntos:** Nunca se suman los soportes individuales; se cuentan las filas donde todos los elementos del conjunto aparecen simultáneamente.
* **Ejemplo Supermercado ($N=10$):** `Pan` está en $8$ ventas ($\text{Soporte} = 8/10 = 80\%$). El par $\{\text{Leche, Pan}\}$ coincide en $5$ ventas ($\text{Soporte} = 5/10 = 50\%$).
* **Ejemplo Tech ($N=1.000$):** Si $120$ clientes compraron $\{\text{Auriculares, Micrófono}\}$ en la misma factura, el soporte del par es $120/1.000 = 0,12$ ($12\%$).

### B. Confianza (Frecuencia Condicional o Fidelidad de la Regla)
Una vez hallados los conjuntos frecuentes, se evalúan reglas direccionales ($\text{Antecedente } A \rightarrow \text{Consecuente } B$). Responde a la pregunta: *"Dado que un cliente ya tiene $A$ en su canasta, ¿qué probabilidad hay de que también lleve $B$?"*

$$\text{Confianza}(A \rightarrow B) = P(B|A) = \frac{\text{Soporte}(A \cup B)}{\text{Soporte}(A)} = \frac{\text{Transacciones con } A \text{ y } B \text{ juntos}}{\text{Transacciones con } A}$$

* **La Confianza NO es simétrica:** Invertir la flecha mantiene el numerador igual, pero cambia el denominador.
  * *En el supermercado:* $\text{Leche} \rightarrow \text{Pan} = 5/6 = 83,3\%$ (de $6$ compras con leche, $5$ llevan pan). En cambio, $\text{Pan} \rightarrow \text{Leche} = 5/8 = 62,5\%$ (de $8$ compras con pan, $5$ llevan leche).
  * *En E-commerce:* Si $100$ personas compran una `Consola` y $80$ de ellas llevan un `Joystick_Extra`, la confianza de $\text{Consola} \rightarrow \text{Joystick\_Extra}$ es $80/100 = 80\%$.

> ⚠️ **El peligro de mirar solo la Confianza (La trampa del producto estrella):** Si en un supermercado el $90\%$ de la gente compra `Bolsa_Plástica` por defecto, cualquier regla como $\text{Atún} \rightarrow \text{Bolsa\_Plástica}$ tendrá $\sim 90\%$ de confianza, aunque el atún y la bolsa no tengan ninguna relación comercial real. Para detectar esta ilusión estadística existe el **Lift**.

### C. Lift (Fuerza Real de Asociación vs. Azar)
Compara la confianza de la regla contra la popularidad natural del consecuente $B$ por sí solo. Mide cuántas veces más probable es llevar $B$ cuando se compra $A$, respecto de comprarlo al azar.

$$\text{Lift}(A \rightarrow B) = \frac{\text{Confianza}(A \rightarrow B)}{\text{Soporte}(B)} = \frac{\text{Soporte}(A \cup B)}{\text{Soporte}(A) \times \text{Soporte}(B)}$$

* **$\text{Lift} = 1$ (Independencia):** $A$ y $B$ ocurren juntos por simple casualidad estadística. No hay vínculo real.
* **$\text{Lift} > 1$ (Asociación Positiva / Complementarios):** $A$ impulsa la presencia de $B$. Ej: $\text{Pañales} \rightarrow \text{Toallitas}$ tiene $\text{Confianza} = 70\%$ y $\text{Soporte}(\text{Toallitas}) = 20\% \implies \text{Lift} = 0,70 / 0,20 = 3,5$ (aparecen juntos $3,5$ veces más de lo normal).
* **$\text{Lift} < 1$ (Asociación Negativa / Sustitutos):** Quien lleva $A$ tiende a **NO** llevar $B$ (ej. $\text{Coca\_Cola} \rightarrow \text{Pepsi}$ con $\text{Lift} = 0,30$; o en clase $\text{Leche} \rightarrow \text{Queso}$ con $\text{Lift} = 0,833$).
* **Propiedad de Simetría:** Como el denominador es un producto, $\text{Lift}(A \rightarrow B) = \text{Lift}(B \rightarrow A)$ siempre.

---

## 3. El Motor Matemático: Propiedad Apriori y Poda (Anti-monotonía)

En un catálogo pequeño de solo $20$ productos existen $2^{20} - 1 = 1.048.575$ combinaciones posibles (en un supermercado con $1.000$ ítems, el número supera los átomos del universo). Para evitar el colapso computacional, Apriori aplica su ley de oro:

| Caso de Análisis | Nivel 1 / Condición Inicial | Nivel 2 / Pares | Resultado de Poda (Anti-monotonía) |
| :--- | :--- | :--- | :--- |
| **Caso A: Poda en Nivel 2** | $\{\text{Pan}\}, \{\text{Leche}\}$ <br> *(Superan `min_support`)* | **Supuesto:** $\{\text{Leche, Pan}\}$ falla <br> *(No alcanza el soporte mínimo ❌)* | **¿Qué pasa con el trío $\{\text{Leche, Pan, Queso}\}$?** <br> ¡Imposible que sea frecuente! Se poda sin contar en la base. |
| **Caso B: Poda en Nivel 1** | $\{\text{Huevos}\} \ (20\%)$ <br> *(Falla `min_support` $30\%$ ❌)* | **PODA AUTOMÁTICA** | Se descartan directamente $\{\text{Huevos, Pan}\}$, $\{\text{Huevos, Leche}\}$... |

* **Enunciado de la Propiedad Apriori:** *"Todos los subconjuntos no vacíos de un itemset frecuente también deben ser frecuentes"*. Dicho al revés (para podar): si un conjunto **NO** alcanza el soporte mínimo, ningún superconjunto más grande que lo contenga podrá alcanzarlo jamás.
* **Demostración intuitiva (Anti-monotonía):** Agregar condiciones a una búsqueda solo puede mantener o achicar los resultados: 
  $$\text{Soporte}(\{A, B, C\}) \le \text{Soporte}(\{A, B\}) \le \text{Soporte}(\{A\})$$
  Por ende, si $\{\text{Leche, Pan}\}$ no alcanza el soporte mínimo, $\{\text{Leche, Pan, Queso}\}$ jamás podrá ser frecuente con ese mismo soporte mínimo, sin importar qué tan alto sea el soporte de `Queso` por separado.

---

## 4. Nivel Avanzado: Métricas Adicionales, Escalabilidad y Umbrales

Cuando trabajes con bases de datos reales en producción, notarás que Apriori tiene extensiones y alternativas técnicas clave que distinguen a un analista avanzado:

| Categoría | Concepto / Algoritmo | Descripción Técnica |
| :--- | :--- | :--- |
| **Métricas extra en `mlxtend`** | **Leverage (Apalancamiento)** | Mide la diferencia absoluta: $\text{Soporte}(A \cup B) - [\text{Soporte}(A) \times \text{Soporte}(B)]$. A diferencia del Lift (que puede dar altísimo con solo $2$ ventas raras), el Leverage prioriza pares con alto volumen total. |
| **Métricas extra en `mlxtend`** | **Conviction (Convicción)** | Mide qué tanto depende $B$ de $A$ evaluando la frecuencia de error (cuando aparece $A$ pero **NO** $B$). Es sensible a la dirección de la flecha. |
| **Apriori vs. FP-Growth** | **Apriori** | Genera candidatos por niveles ($L_1, L_2, L_3\dots$) y escanea la base de datos repetidas veces. Excelente para explicar el proceso, pero lento con millones de filas. |
| **Apriori vs. FP-Growth** | **FP-Growth** | Comprime la base en un árbol en memoria (*FP-Tree*) y extrae conjuntos frecuentes sin generar candidatos masivos. Escala mucho mejor en Big Data. |

* **Criterio profesional para elegir umbrales:** No existe una fórmula fija; depende de la dispersión del negocio. En un ejemplo de aula ($10$ tickets) se usa `min_support = 0.30` ($30\%$). En un supermercado o e-commerce real con miles de artículos, casi ningún par aparece en el $30\%$ de los carritos; allí se configura entre `0.01` ($1\%$) y `0.05` ($5\%$). Si el soporte es muy alto, perdés nichos rentables; si es muy bajo, explota la memoria con reglas espurias.

---

## 5. Plantilla Reproducible en Python (Para cualquier Dataset)

En tus proyectos futuros te vas a encontrar con dos tipos de tablas crudas: **Formato Largo** (una fila por producto con un ID de orden repetido) o **Formato Horizontal** (una fila por compra con columnas `Item 1`, `Item 2`...). Este bloque resuelve ambos casos y ejecuta el flujo completo:

```python
# Importar librerías necesarias
import pandas as pd
from mlxtend.frequent_patterns import apriori, association_rules

# CASO A: Si tu dataset viene en Formato Horizontal (ej. Groceries: Item 1, Item 2...)
# Lo apilamos a formato largo para estandarizarlo en dos columnas: order_id e item_name
cols_items = [c for c in df_raw.columns if c.startswith('Item ')]
df_largo = df_raw[cols_items].stack().reset_index(level=1, drop=True).reset_index()
df_largo.columns = ['order_id', 'item_name']

# CASO B: Si tu dataset YA viene en Formato Largo (con columna de orden y de producto)
# Simplemente creamos la matriz binaria de presencia (0 y 1) usando crosstab y .gt(0)
df_binario = pd.crosstab(df_largo['order_id'], df_largo['item_name']).gt(0)

# Paso 1 de Apriori: Encontrar itemsets frecuentes según el soporte mínimo elegido
frecuentes = apriori(df_binario, min_support=0.05, use_colnames=True)

# Paso 2 de Apriori: Generar reglas de asociación filtrando por confianza o lift
reglas = association_rules(frecuentes, metric='confidence', min_threshold=0.50)

print(reglas[['antecedents', 'consequents', 'support', 'confidence', 'lift']].sort_values(by='lift', ascending=False))
```

---

## 6. Cuestionario Maestro: Trampas de Examen y Preguntas Clave

Repaso definitivo de las afirmaciones capciosas que suelen evaluarse en parciales y cuestionarios teóricos:

| Afirmación o Situación de Examen | Veredicto | Justificación Técnica para Responder |
| :--- | :---: | :--- |
| *"Si $\{\text{Leche, Pan}\}$ no alcanza el soporte mínimo, ¿qué pasa con $\{\text{Leche, Pan, Queso}\}$?"* | **Se poda** | No puede ser frecuente con ese mismo soporte mínimo. Por la Propiedad Apriori, al agregar ítems el soporte solo puede mantenerse o bajar, nunca subir. |
| *"La regla $A \rightarrow B$ indica que el cliente puso en el carrito primero $A$ y luego $B$"* | **Falso** | Un itemset es un conjunto sin orden temporal ni secuencia de escaneo en caja; la flecha solo indica probabilidad condicional. |
| *"Una regla con alto Soporte, Confianza y Lift demuestra que comprar $A$ causa la compra de $B$"* | **Falso** | Coaparición no es causalidad. Ambos productos pueden estar impulsados por un tercer factor externo (ej. clima o día festivo). |
| *"Una confianza del $90\%$ siempre garantiza una asociación fuerte y útil"* | **Falso** | Si el producto $B$ aparece en el $95\%$ de todas las ventas, una confianza del $90\%$ da un $\text{Lift} < 1 \ (0,94)$, indicando que se llevan juntos menos de lo esperado. |
| *"Alcanza con ordenar las reglas por Lift y elegir la más alta"* | **Falso** | Dos productos raros que casualmente se compraron juntos en un único ticket tendrán un Lift gigantesco pero soporte ínfimo (regla inestable). |
| *"Si invierto la regla de $A \rightarrow B$ a $B \rightarrow A$, cambian tanto la Confianza como el Lift"* | **Falso** | La Confianza sí cambia (cambia su denominador $\text{Soporte}(B)$), pero el Soporte y el Lift permanecen exactamente iguales (son simétricos). |
| *"Si un cliente lleva $4$ botellas de leche en el mismo ticket, el soporte de `Leche` sube más"* | **Falso** | En Apriori binario cada transacción pesa una sola vez ($1 =$ presente, $0 =$ ausente), sin importar la cantidad de unidades. |

---

## 7. Checklist para Resolver Ejercicios a Mano en 5 Pasos

1. **Contar $N$:** Identificar el número total de transacciones únicas (dividirás todos los soportes por este número $N$).
2. **Calcular Nivel 1 ($L_1$) y Podar:** Contar en cuántos tickets aparece cada ítem suelto, dividir por $N$ y tachar los que sean menores a `min_support`.
3. **Armar Pares ($L_2$) solo con sobrevivientes:** Nunca armes parejas usando ítems tachados en el paso anterior. Contar apariciones simultáneas, dividir por $N$ y volver a filtrar por `min_support`.
4. **Calcular Confianza ($A \rightarrow B$):** Dividir el soporte del par $\{A, B\}$ por el soporte del antecedente $A$ (el que está a la izquierda de la flecha).
5. **Calcular Lift ($A \rightarrow B$):** Dividir la Confianza recién obtenida por el soporte individual del consecuente $B$ (el que está a la derecha de la flecha). Si da $> 1$, la regla es válida.