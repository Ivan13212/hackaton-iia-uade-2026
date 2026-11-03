# <M05> · <¿Este cliente se va el próximo trimestre?>

> **Hackatón IIA · UADE · 3.4.257.** Este README es la plantilla del grupo. En su fork, reemplacen todo lo que está entre `< >` y borren esta nota. Las bases están en `BASES.md` y su ficha en `problematicas/`.

**Integrantes:** <Ivan Fuente> · <mi companero no pudo asistir>
**IA declarada:** <claude>
**Pista:** <Reglas>
**Publicación en LinkedIn:** <>

---

## 1. Formulación y PEAS

**La pregunta.** <Para cada caso, ¿…? SÍ significa: …>

| PEAS | Su sistema |
|---|---|
| Medida de rendimiento | <Costo total de los errores: bonificaciones regaladas (FP) más facturación perdida (FN). Subordinada a esto: sensibilidad alta> |
| Entorno | <Cartera de comercios que tercerizan sus envíos; el área comercial decide si ofrece bonificación> |
| Actuadores | <Recomienda SÍ/NO; si es SÍ, el área comercial ofrece la bonificación del 10 %> |
| Sensores | <Variables de cada cliente que aparezcan en la ficha (volumen de envíos, reclamos, etc.)> |

**Objetivo y métrica.** <Lo que importa es minimizar el costo, no acertar la mayor cantidad de casos. Por eso la exactitud no alcanza como métrica: un sistema que siempre dice NO puede tener buena exactitud si pocos clientes se van, pero pierde toda la facturación de los que sí se van. La métrica central tendría que ser la sensibilidad (de los que se van, cuántos detecto), acompañada del costo total.>

## 2. El sistema

**Cómo decide.** <Regla (decide SÍ si se cumplen las dos condiciones):

SI entregas_tardias >= 5 %  Y  reclamos_abiertos >= 3  →  SÍ
EN OTRO CASO                                          →  NO>

**Variables que usa y por qué.** <Se descarto la antigüedad de los clientes, ¿Por qué la descarté?

En el entrenamiento, la antigüedad no separa a los que se van de los que se quedan:

	Antigüedad (meses)	Promedio
Se fueron (9 casos)	1, 14, 22, 24, 29, 35, 45, 46, 50	≈ 30
Se quedaron (5 casos)	9, 21, 41, 45, 48	≈ 33

Los rangos se pisan casi por completo y los promedios son prácticamente iguales. Hay clientes de 45 meses que se fueron (casos 8 y 12) y clientes de 45 y 48 meses que se quedaron (casos 4 y 5). Como no hay ningún umbral que ayude a decidir, agregarla a la regla solo sumaría ruido.>

**Cómo se corre en vivo.** <se miran las entregas tardías y los reclamos del cliente y se aplican las dos condiciones. No hace falta Colab, alcanza con la cuenta a mano.>

**Dos casos de ejemplo.**

| Caso inventado | Valores (tardías, reclamos, antigüedad) | Salida del sistema |
|---|---|---|
| A | <30 %, 7, 12 meses> | <SÍ> |
| B | <2 %, 1, 30 meses> | <NO> |

## 3. Medición

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <9> | FN = <0> |
| **Real NO** | FP = <0> | VN = <5> |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <2 (casos 15, 19)> | FN = <0> |
| **Real NO** | FP = <3 (casos 16, 18, 20)> | VN = <1 (caso 17)> |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | <14/14 = 100 %> | <3/6 = 50 %> |
| Precisión | <9/9 = 100 %> | <2/5 = 40 %> |
| Sensibilidad | <9/9 = 100 %> | <2/2 = 100 %> |
| Especificidad | <5/5 = 100 %> | <1/4 = 25 %> |

**Entrenamiento contra prueba.** <Cae todo salvo la sensibilidad. Es un caso claro de sobreajuste: las reglas se ajustaron a 14 casos, y en 6 casos nuevos aparecen 3 falsos positivos. Hay que ser prudentes con la lectura, porque con solo 6 casos de prueba un caso mueve las métricas casi 17 puntos. Lo que sí se sostuvo fue lo importante: no se escapó ningún cliente que se iba.>

## 4. El error más caro

<El falso negativo es mucho peor: se pierde toda la facturación anual del cliente, contra el 10 % de una factura del falso positivo. Si la factura es mensual, el FN cuesta unas 240 veces más. Para cometer menos FN hay que sesgar el sistema hacia el SÍ, es decir, bajar el umbral de decisión (en Neurona, mover el sesgo; en Reglas, hacer las condiciones más permisivas). Es esperable que suban los FP, y eso es aceptable porque cada uno cuesta poco.>

## 5. ¿Comparado contra qué?

**Línea base trivial.** <
Sistema	Entrenamiento (exactitud)	Prueba (exactitud)	Sensibilidad en prueba	Costo en prueba*
Siempre NO	36 %	67 %	0 %	2 facturación anual
Siempre SÍ	64 %	33 %	100 %	≈ 0,017
Nuestro sistema	100 %	50 %	100 %	≈ 0,0125

*Costo medido en "facturaciones anuales de un cliente". Se supone factura mensual: un FN cuesta 1 y un FP cuesta 0,05/12 ≈ 0,0042. Si fuera una factura anual, el FP costaría 0,05 y el orden entre los sistemas no cambia.*

**Lectura de la comparación**
- **La exactitud engaña.** "Siempre NO" saca 67 % en prueba, más que nuestro sistema, pero no detecta a ningún cliente que se va y pierde toda su facturación.
- **Contra "siempre SÍ" la ventaja es chica.** Nuestro sistema es más barato, pero solo evita 1 bonificación en 6 casos de prueba. Su valor real aparece en una cartera grande, donde evitar bonificaciones innecesarias se acumula.
- **Con 6 casos de prueba** una sola predicción cambia las métricas casi 17 puntos, así que estos resultados son indicativos y no concluyentes.>

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | <20 casos de la ficha: 14 de entrenamiento (casos 1–14) y 6 de prueba (casos 15–20). Variables: entregas tardías (%), reclamos abiertos y antigüedad (esta última descartada). |
| ¿Qué métrica? | Sensibilidad y costo esperado como métricas principales, porque el FN es mucho más caro que el FP. La exactitud se reporta, pero es secundaria. |
| ¿Comparado contra qué? | Contra las líneas base "siempre NO" y "siempre SÍ". |
| ¿Quién lo midió? | Nosotros mismos, con los datos de la ficha. La prueba no fue ciega: los umbrales se eligieron razonando con el entrenamiento y el costo asimétrico, pero ya conocíamos los casos de prueba. No hubo evaluación independiente. |

## 6. Riesgo: el camino completo

<Datos: con solo 20 casos el patrón puede ser casual y no generalizar.
Modelo: si el sistema se ajusta a los casos 1–14, puede fallar en los nuevos.
Decisión: un sistema demasiado cauteloso dice SÍ a casi todos.
Consecuencia: el área comercial regala bonificaciones sin criterio y se erosiona el margen. Además, el cliente que recibe la oferta podría pedirla siempre.>

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | < el siguiente contexto: las problematicas del M05, El README.md, y tambien le habia pasado el link del github (https://github.com/dharioprodigio/hackaton-iia-uade-2026.git). lo que le pedi especificamente fue:
ya hice el fork, Te paso el contexto del trabajo:
Dominio: Logística: Fuga de clientes · IA declarada: Claude
Situación. Una empresa de logística trabaja con comercios que le tercerizan sus envíos. Quiere detectar a tiempo qué clientes están por irse a la competencia. Cada caso es un cliente.
Quién decide. El área comercial, que puede ofrecer una bonificación de retención.
La pregunta. Para cada caso, ¿SÍ o NO? SÍ = el cliente se va en el próximo trimestre.
Si el sistema se equivoca

Falso positivo (dice SÍ y era NO): Bonificar a un cliente que no se iba: se regala un 5 % de una factura.
Falso negativo (dice NO y era SÍ): No detectar a un cliente que se va: se pierde toda su facturación anual.

te copio y pego el README, ya que no es tan largo
# <Código> · <Título de su problemática>

**Hackatón IIA · UADE · 3.4.257.** Este README es la plantilla del grupo. En su fork, reemplacen todo lo que está entre < > y borren esta nota. Las bases están en BASES.md y su ficha en problematicas/.

**Integrantes:** <nombre y apellido> · <nombre y apellido>
**IA declarada:** <la del grupo desde la Clase 1>
**Pista:** <Neurona / Reglas / Agente>
**Publicación en LinkedIn:** <link>

---

## 1. Formulación y PEAS

**La pregunta.** <Para cada caso, ¿…? SÍ significa: …>

| PEAS | Su sistema |
|---|---|
| Medida de rendimiento | <cómo se sabe si decidió bien> |
| Entorno | <dónde y con qué decide> |
| Actuadores | <qué hace con la decisión> |
| Sensores | <qué datos recibe> |

**Objetivo y métrica.** <Qué es lo que de verdad importa en su problema y con qué métrica lo van a medir. ¿Son lo mismo?>

## 2. El sistema

**Cómo decide.** <Neurona: pesos y sesgo. Reglas: la lista de reglas si‑entonces. Agente: la regla con la que elige la acción.>

**Variables que usa y por qué.** <Cuáles usa, cuáles descarta y por qué.>

**Cómo se corre en vivo.** <Pasos concretos para resolver un caso nuevo: link al Colab, a la planilla o la cuenta a mano.>

**Dos casos de ejemplo.**

| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| A | <…> | <SÍ/NO> |
| B | <…> | <SÍ/NO> |

## 3. Medición

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <> | FN = <> |
| **Real NO** | FP = <> | VN = <> |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <> | FN = <> |
| **Real NO** | FP = <> | VN = <> |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | <> | <> |
| Precisión | <> | <> |
| Sensibilidad | <> | <> |
| Especificidad | <> | <> |

**Entrenamiento contra prueba.** <¿Hay diferencia? ¿Por qué? ¿Sobreajuste o generalización?>

## 4. El error más caro

<¿Es peor el falso positivo o el falso negativo en su problema? ¿Por qué, según la ficha? ¿Qué cambiaron en el sistema para cometer menos ese error?>

## 5. ¿Comparado contra qué?

**Línea base trivial.** <Qué resultado saca un sistema que responde siempre lo mismo, y cómo se compara el suyo.>

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | <> |
| ¿Qué métrica? | <> |
| ¿Comparado contra qué? | <> |
| ¿Quién lo midió? | <> |

## 6. Riesgo: el camino completo

<Datos → modelo → decisión → consecuencia aplicado a su caso. Al menos un riesgo concreto.>

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | <> | <> | <> |>

la pista elegida son Reglas y aca te paso los 20 casos de la ficha
Caso    Conjunto    Entregas tardías (% del último trimestre)    Reclamos abiertos (cantidad)    Antigüedad como cliente (meses)    Resultado real (1 = SÍ)
1    Entrenamiento    39    6    1    1
2    Entrenamiento    31    6    35    1
3    Entrenamiento    4    5    21    0
4    Entrenamiento    1    9    45    0
5    Entrenamiento    18    2    48    0
6    Entrenamiento    0    1    9    0
7    Entrenamiento    21    6    29    1
8    Entrenamiento    39    8    46    1
9    Entrenamiento    34    10    22    1
10    Entrenamiento    15    6    50    1
11    Entrenamiento    28    8    24    1
12    Entrenamiento    40    8    45    1
13    Entrenamiento    1    8    41    0
14    Entrenamiento    33    6    14    1
15    Prueba    5    6    17    1
16    Prueba    13    6    50    0
17    Prueba    2    5    20    0
18    Prueba    6    9    49    0
19    Prueba    20    4    6    1
20    Prueba    24    3    25    0

de la seccion 2 explicame porque no usaste las variables Antigüedad como cliente 

haz de vuelta la sección 5 de forma mas organizada

| <> | <Rechaze la exesiva explicacion de porque el rechazo de la variable antiguedad (lo resumi acortando la respuesta) y tambien le exigi que cambie la forma en la que resolvio la seccion 5 ¿comparado contra que? (aunque esto quizas no cuenta del todo como un rechazo, si no mas bien un cambio, para que la respuesta de la IA sea mas comoda de analizar, ya que se me habia hecho confusa)> |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | <> | <> | <> |
