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
| Actuadores | <Recomienda SÍ/NO; si es SÍ, el área comercial ofrece la bonificación del 5 %> |
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

<El falso negativo es mucho peor: se pierde toda la facturación anual del cliente, contra el 5 % de una factura del falso positivo. Si la factura es mensual, el FN cuesta unas 240 veces más. Para cometer menos FN hay que sesgar el sistema hacia el SÍ, es decir, bajar el umbral de decisión (en Neurona, mover el sesgo; en Reglas, hacer las condiciones más permisivas). Es esperable que suban los FP, y eso es aceptable porque cada uno cuesta poco.>

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

haz de vuelta la sección 5 de forma mas organizada> |

<Aclaraciones sobre "lo que aceptamos": las respuestas que claude concluyo fueron modificadas ligeramente para que el trabajo sea mas presentables, pero realmente las respuestas que dio terminan siendo iguales (conceptualmente) a las respuestas que coloque en el trabajo

Voy a mirar qué trae el repositorio para ayudarte con base en eso.

No pude abrir el repositorio (GitHub bloquea el acceso automático), así que no sé qué archivos trae. Si me pegás el README, la consigna o la estructura de carpetas, trabajo con eso. Mientras tanto, te adelanto lo que sale del enunciado.

Qué dice el costo de los errores

Los dos errores no pesan lo mismo, así que no conviene decidir SÍ/NO con el corte clásico de 50 %. Conviene decir SÍ cuando la probabilidad de fuga supera:

umbral = costo_FP / (costo_FP + costo_FN)
Si “una factura” es una factura mensual, el FP cuesta 5 % × (facturación anual / 12) ≈ 0,4 % de la facturación anual y el FN cuesta el 100 %. El umbral queda en torno a 0,4 %.
Si fuera una factura anual, el umbral sería ≈ 4,8 %.

En cualquiera de los dos casos el umbral es muy bajo: ante la menor sospecha de fuga conviene decir SÍ. Es un supuesto que conviene dejar explícito en tu entrega.

Cómo plantearía el trabajo

Datos y variables: frecuencia y volumen de envíos (y su tendencia), reclamos, demoras o incidencias, antigüedad, facturación, cambios recientes de comportamiento.
Modelo: algo simple y explicable, como regresión logística o un árbol, que devuelva una probabilidad y no solo SÍ/NO.
Decisión: aplicar el umbral basado en costos en lugar del 50 %.
Evaluación: medir el costo total esperado y no solo la exactitud. Acá importa mucho el recall de los SÍ, porque un modelo con buena exactitud puede ser muy caro si deja escapar clientes.
Rol de Claude (IA declarada): dejar documentado cómo se usó (generar variables, revisar el razonamiento, redactar la justificación) y qué decidiste vos.

¿Qué necesitás concretamente: entender la consigna, armar el código en Python, o redactar la justificación de la decisión? Si me pasás qué hay en el repo (o los datos), seguimos desde ahí.

Gracias, con esto se ve la estructura. Las secciones 2, 3 y 5 dependen de datos que no tengo: los 20 casos (están en la ficha, dentro de problematicas/) y la pista que eligieron (Neurona, Reglas o Agente). Mientras tanto te dejo borradores de las partes que salen del enunciado, para que los ajusten con sus palabras.

1. Formulación y PEAS (borrador)

La pregunta. Para cada cliente, ¿se va a la competencia en el próximo trimestre? SÍ significa que el cliente se va; NO, que se queda.

PEAS	Su sistema
Medida de rendimiento	Costo total de los errores: bonificaciones regaladas (FP) más facturación perdida (FN). Subordinada a esto: sensibilidad alta
Entorno	Cartera de comercios que tercerizan sus envíos; el área comercial decide si ofrece bonificación
Actuadores	Recomienda SÍ/NO; si es SÍ, el área comercial ofrece la bonificación del 5 %
Sensores	Variables de cada cliente que aparezcan en la ficha (volumen de envíos, reclamos, etc.)

Objetivo y métrica. Lo que importa es minimizar el costo, no acertar la mayor cantidad de casos. Por eso la exactitud no alcanza como métrica: un sistema que siempre dice NO puede tener buena exactitud si pocos clientes se van, pero pierde toda la facturación de los que sí se van. La métrica central tendría que ser la sensibilidad (de los que se van, cuántos detecto), acompañada del costo total.

4. El error más caro (borrador)

El falso negativo es mucho peor: se pierde toda la facturación anual del cliente, contra el 5 % de una factura del falso positivo. Si la factura es mensual, el FN cuesta unas 240 veces más. Para cometer menos FN hay que sesgar el sistema hacia el SÍ, es decir, bajar el umbral de decisión (en Neurona, mover el sesgo; en Reglas, hacer las condiciones más permisivas). Es esperable que suban los FP, y eso es aceptable porque cada uno cuesta poco.

6. Riesgo (ideas para desarrollar)
Datos: con solo 20 casos el patrón puede ser casual y no generalizar.
Modelo: si el sistema se ajusta a los casos 1–14, puede fallar en los nuevos.
Decisión: un sistema demasiado cauteloso dice SÍ a casi todos.
Consecuencia: el área comercial regala bonificaciones sin criterio y se erosiona el margen. Además, el cliente que recibe la oferta podría pedirla siempre.
Para seguir necesito
Los 20 casos de la ficha (pegados como texto o tabla).
La pista elegida.

Con eso armamos el sistema, las matrices de confusión, las métricas y la línea base (responder siempre SÍ o siempre NO). En la bitácora (sección 7) tienen que registrar lo que realmente me pidan, qué aceptan y qué rechazan, así que la vamos completando a medida que trabajemos.

Con los 20 casos ya se puede armar el sistema de reglas y completar las secciones 2, 3 y 5.

Qué muestran los datos
Entregas tardías es la variable más fuerte. Todos los que se fueron en entrenamiento tienen 15 % o más, y los clientes con 4 % o menos de entregas tardías se quedaron.
Reclamos corrige la excepción del caso 5 (18 % tarde pero solo 2 reclamos, y se quedó). Todos los que se fueron tienen 6 o más reclamos, y el caso 5 tiene 2.
Antigüedad no muestra patrón: se van clientes de 1 y de 50 meses, y también se quedan clientes de 9 y de 48. Conviene descartarla y explicarlo en “Variables que usa y por qué”.
Sección 2: el sistema

Regla (decide SÍ si se cumplen las dos condiciones):

SI entregas_tardias >= 5 %  Y  reclamos_abiertos >= 3  →  SÍ
EN OTRO CASO                                          →  NO

Cómo se eligieron los umbrales (solo con entrenamiento):

Entre los casos con 3 o más reclamos, los que se quedaron tienen como máximo 4 % de entregas tardías, y los que se fueron tienen 15 % o más.
Cualquier umbral entre 5 y 15 separa bien el entrenamiento. Elegí el extremo bajo (5 %) porque el falso negativo es muchísimo más caro, así que conviene errar hacia el SÍ.
Con los reclamos pasa igual: el hueco va de 2 a 6, y elegí 3 por la misma razón.

Cómo se corre en vivo: se miran las entregas tardías y los reclamos del cliente y se aplican las dos condiciones. No hace falta Colab, alcanza con la cuenta a mano.

Caso inventado	Valores (tardías, reclamos, antigüedad)	Salida
A	30 %, 7, 12 meses	SÍ
B	2 %, 1, 30 meses	NO
Sección 3: medición

Entrenamiento (1–14)

	Predice SÍ	Predice NO
Real SÍ	VP = 9	FN = 0
Real NO	FP = 0	VN = 5

Prueba (15–20)

	Predice SÍ	Predice NO
Real SÍ	VP = 2 (casos 15, 19)	FN = 0
Real NO	FP = 3 (casos 16, 18, 20)	VN = 1 (caso 17)
Métrica	Entrenamiento	Prueba
Exactitud	14/14 = 100 %	3/6 = 50 %
Precisión	9/9 = 100 %	2/5 = 40 %
Sensibilidad	9/9 = 100 %	2/2 = 100 %
Especificidad	5/5 = 100 %	1/4 = 25 %

Entrenamiento contra prueba. Cae todo salvo la sensibilidad. Es un caso claro de sobreajuste: las reglas se ajustaron a 14 casos, y en 6 casos nuevos aparecen 3 falsos positivos. Hay que ser prudentes con la lectura, porque con solo 6 casos de prueba un caso mueve las métricas casi 17 puntos. Lo que sí se sostuvo fue lo importante: no se escapó ningún cliente que se iba.

Sección 4: el error más caro

El falso negativo, porque se pierde toda la facturación anual, contra el 5 % de una factura. Si la factura es mensual, un FN cuesta unas 240 veces más que un FP. Para cometer menos FN se bajaron los umbrales hasta el extremo permisivo del hueco. El costo es aceptar FP, que son baratos.

Sección 5: ¿comparado contra qué?
Sistema	Entrenamiento (exactitud)	Prueba (exactitud)	Sensibilidad en prueba	Costo en prueba*
Siempre NO	36 %	67 %	0 %	2 facturación anual
Siempre SÍ	64 %	33 %	100 %	≈ 0,017
Nuestro sistema	100 %	50 %	100 %	≈ 0,0125

*Medido en “facturaciones anuales de un cliente”, suponiendo factura mensual: FN = 1 y FP = 0,05/12 ≈ 0,0042. Si fuera una sola factura anual, el FP vale 0,05 y el orden de los sistemas no cambia.

Conclusiones para escribir:

La exactitud engaña. “Siempre NO” saca 67 % en prueba, más que el sistema, y no detecta a ningún cliente que se va.
El valor del sistema es modesto. Es apenas más barato que bonificar a todos, porque ahorra 1 bonificación en 6 casos. Conviene decirlo con esas palabras. Si les preguntan, esa es la comparación honesta.
Una advertencia sobre la bitácora

Los umbrales los elegí razonando con el entrenamiento, pero ya conocía los casos de prueba cuando armé la regla. Una versión más estricta (tardías ≥ 15 %) deja un FN en prueba (el caso 15), que es justamente el error caro. Declaren en la bitácora que el umbral final se decidió por el costo asimétrico y que la prueba no era “ciega”, porque eso es lo que la sección 5 (“¿Quién lo midió?”) busca.

Sección 6: riesgos concretos
Datos: 20 casos y 3 variables. Faltan señales como caídas de volumen o contacto con la competencia.
Decisión: el área comercial podría bonificar a casi todos y erosionar el margen.
Consecuencia: si los clientes se enteran de que reclamar da bonificación, pueden empezar a reclamar más.

¿Querés que te arme la bitácora con lo que pedimos y lo que aceptamos o rechazamos, o que armemos el texto de la sección 4 y 6 en tus palabras?

Una aclaración primero: “Resultado real” no es una variable que se pueda usar, es la respuesta que el sistema tiene que predecir. Si la usara en la regla estaría haciendo trampa (en datos se llama fuga de información). La pregunta de verdad es por Antigüedad como cliente.

Por qué la descarté

En el entrenamiento, la antigüedad no separa a los que se van de los que se quedan:

	Antigüedad (meses)	Promedio
Se fueron (9 casos)	1, 14, 22, 24, 29, 35, 45, 46, 50	≈ 30
Se quedaron (5 casos)	9, 21, 41, 45, 48	≈ 33

Los rangos se pisan casi por completo y los promedios son prácticamente iguales. Hay clientes de 45 meses que se fueron (casos 8 y 12) y clientes de 45 y 48 meses que se quedaron (casos 4 y 5). Como no hay ningún umbral que ayude a decidir, agregarla a la regla solo sumaría ruido.

Además, sumarla a la regla con un “Y” la hace más estricta, y eso aumenta el riesgo de falsos negativos, que es justo el error caro.

Lo que hay que mencionar con honestidad

En los casos de prueba aparece algo curioso: los dos clientes que se fueron (casos 15 y 19) tienen poca antigüedad (17 y 6 meses), y los que se quedaron tienen 20, 25, 49 y 50. Podría parecer que los clientes nuevos se van más.

No la incorporé por dos razones:

Con solo 2 casos positivos en prueba no se puede concluir nada. Puede ser casualidad.
El entrenamiento no lo respalda, porque ahí hay clientes de 1 mes y de 50 meses que se van. Si la agregara mirando la prueba, estaría ajustando el sistema a los datos con los que después lo mido.

Es una buena hipótesis para escribir en la sección 6 como mejora futura: probarla con más datos.

Texto para pegar en el README (“Variables que usa y por qué”)

Usa: entregas tardías (≥ 5 %) y reclamos abiertos (≥ 3). Descarta: antigüedad como cliente, porque en entrenamiento los clientes que se fueron y los que se quedaron tienen antigüedades casi idénticas (promedio ≈ 30 contra ≈ 33 meses, rangos superpuestos), así que no aporta información para decidir. En los casos de prueba los dos clientes que se fueron tienen poca antigüedad, pero con solo 2 casos no alcanza para sacar una regla y el entrenamiento no lo respalda; queda como hipótesis a validar con más datos.

Acá tenés la sección 5 lista para pegar en el README:

markdown
## 5. ¿Comparado contra qué?

**Línea base trivial.** Se comparó el sistema contra dos respuestas fijas: decir siempre NO (nunca se bonifica) y decir siempre SÍ (se bonifica a todos).

| Sistema | Exactitud (entrenamiento) | Exactitud (prueba) | Sensibilidad (prueba) | Costo (entrenamiento) | Costo (prueba) |
|---|---|---|---|---|---|
| Siempre NO | 5/14 = 36 % | 4/6 = 67 % | 0 % | 9 | 2 |
| Siempre SÍ | 9/14 = 64 % | 2/6 = 33 % | 100 % | ≈ 0,021 | ≈ 0,017 |
| **Nuestro sistema** | 14/14 = 100 % | 3/6 = 50 % | 100 % | 0 | ≈ 0,0125 |

*Costo medido en "facturaciones anuales de un cliente". Se supone factura mensual: un FN cuesta 1 y un FP cuesta 0,05/12 ≈ 0,0042. Si fuera una factura anual, el FP costaría 0,05 y el orden entre los sistemas no cambia.*

**Lectura de la comparación**
- **La exactitud engaña.** "Siempre NO" saca 67 % en prueba, más que nuestro sistema, pero no detecta a ningún cliente que se va y pierde toda su facturación.
- **Contra "siempre SÍ" la ventaja es chica.** Nuestro sistema es más barato, pero solo evita 1 bonificación en 6 casos de prueba. Su valor real aparece en una cartera grande, donde evitar bonificaciones innecesarias se acumula.
- **Con 6 casos de prueba** una sola predicción cambia las métricas casi 17 puntos, así que estos resultados son indicativos y no concluyentes.

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | 20 casos de la ficha: 14 de entrenamiento (casos 1–14) y 6 de prueba (casos 15–20). Variables: entregas tardías (%), reclamos abiertos y antigüedad (esta última descartada). |
| ¿Qué métrica? | Sensibilidad y costo esperado como métricas principales, porque el FN es mucho más caro que el FP. La exactitud se reporta, pero es secundaria. |
| ¿Comparado contra qué? | Contra las líneas base "siempre NO" y "siempre SÍ". |
| ¿Quién lo midió? | Nosotros mismos, con los datos de la ficha. La prueba no fue ciega: los umbrales se eligieron razonando con el entrenamiento y el costo asimétrico, pero ya conocíamos los casos de prueba. No hubo evaluación independiente. |

Un detalle: la fila “¿Quién lo midió?” tiene que reflejar lo que de verdad pasó en su grupo. Si cambian algo del proceso (por ejemplo, si alguien más valida el sistema), ajusten esa respuesta.> |

<Rechaze la exesiva explicacion de porque el rechazo de la variable antiguedad (lo resumi acortando la respuesta) y tambien le exigi que cambie la forma en la que resolvio la seccion 5 ¿comparado contra que? (aunque esto quizas no cuenta del todo como un rechazo, si no mas bien un cambio, para que la respuesta de la IA sea mas comoda de analizar, ya que se me habia hecho confusa)> |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | <8.5> | <¿Cuál es el porcentaje de bonificación que dice la ficha: 5 % o 10 %? Tu README usa los dos.
Otros:
¿Qué parte de la respuesta de la IA aceptaron tal cual, cuál cambiaron y por qué?
En la sección 2 dicen que el caso 8 tiene 45 meses. ¿Cuántos meses tiene realmente?> | <me dedique a cambiar el porcentaje erroneo del 10% que contradecia al que se pide en el trabajo (5%) y, tambien, deje una aclaracion en "lo que aceptamos" para que se entienda mejor la bitacora> |
