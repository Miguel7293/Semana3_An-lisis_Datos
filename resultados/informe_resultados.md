# Semana 3 · Monitoreo de infraestructura con series temporales
**Ingeniería de Sistemas · IX ciclo · Semestre 2026-II**

Desarrollo en el mismo orden y con la misma numeración de la guía. Las respuestas numéricas corresponden al CSV incluido y se verifican con las celdas ejecutadas desde la sección 6. Los bloques titulados **Código recomendado en el PDF** conservan sus instrucciones; los cálculos adicionales se identifican como complementarios.

## 1. Identificación de la práctica
**Caso aplicado:** describir el uso de CPU de una infraestructura AWS y revisar la estructura de sus datos antes de realizar análisis temporal o construir modelos predictivos.

El archivo seleccionado es `cpu_utilization_asg_misconfiguration.csv`, asociado a la sección `realKnownCause` de NAB. El README de NAB describe la métrica como el uso promedio de CPU de un grupo de máquinas. Se utiliza el [enlace de Kaggle indicado por la guía](https://www.kaggle.com/code/joshuaswords/time-series-anomaly-detection).

**Verificación de acceso:** el CSV está junto al notebook y contiene `timestamp` y `value`. El enlace de Kaggle no expuso sus celdas durante la consulta, de modo que no se afirma haber comprobado la ruta interna de ese notebook. La ruta local y las columnas sí se verifican en la sección 6.

## 2. Preparación del entorno y descarga
El dataset ya estaba descargado en `Semana3`. Se conserva intacto y se utiliza una copia junto a este notebook. No se sustituyó el archivo.

**Versión y fecha de descarga:** no documentadas en los archivos entregados. La consulta de referencias se realizó el 13/10/2026; esa fecha no se considera fecha de descarga. Se registra la huella SHA-256 en la ficha técnica para identificar la copia utilizada.

**Entorno:** Python, pandas y matplotlib; los controles complementarios utilizan numpy. Abrir Jupyter en la carpeta del notebook y ejecutar las celdas en orden. Los archivos de resultados se guardan en `resultados/`. El código de análisis comienza en la sección 6, como en el PDF.

## 3. Contexto y preguntas de análisis
**¿Qué representa una fila: un servidor distinto, una métrica en un instante o un evento?**

Representa una medición de la métrica de CPU en un instante. En esta serie es el uso promedio del grupo de máquinas; cada fila no identifica un servidor distinto ni un evento operativo.

**¿Cuál es la unidad de análisis y cuál es la unidad temporal?**

La unidad de análisis es una medición del uso promedio de CPU del grupo en un instante. La unidad temporal observada es un intervalo de **cinco minutos**, verificado en la sección 9.

**¿Las marcas temporales son únicas, ordenadas y regularmente espaciadas?**

Sí. Las 18 050 marcas son únicas y ya están ordenadas en el archivo original. Las 18 049 diferencias consecutivas son de cinco minutos. Las secciones 8 y 9 verifican duplicados, orden y regularidad.

**¿Qué proporción de registros está disponible por intervalo?**

La cobertura dentro de la ventana observada es **100 %**: 18 050 puntos observados de 18 050 esperados. Las horas completas tienen 12 registros y los días completos, 288. La primera hora tiene 10 de 12 y la última 4 de 12; el primer día tiene 274 de 288 (**95,14 %**) y el último 208 de 288 (**72,22 %**). Estos extremos son intervalos calendario parciales por los límites del archivo, no huecos internos. La sección 11 muestra ambos tipos de cobertura.

**¿Cómo cambia el uso de CPU a lo largo del período?**

Se observa una base frecuente alrededor de 30-35, con picos recurrentes que llegan a 100 y variabilidad cambiante. Las medias diarias suben a 57,98 y 56,75 el 12 y 13 de julio; el 14 registran 52,35. El último día observado baja a 13,78, aunque es parcial. No hay una tendencia lineal única que describa todo el período. La sección 10 presenta la gráfica y la sección 11 los resúmenes.

**¿Qué conclusiones no pueden obtenerse con una sola serie?**

No permite comparar servidores individuales, generalizar a toda AWS o a todos los servidores de una empresa, ni atribuir causalidad a cada pico. Tampoco permite confirmar por sí sola fallas, acciones de autoescalado o demanda futura. El promedio del grupo puede ocultar diferencias entre sus máquinas.

## 4. Unidad de observación y estructura
La unidad de observación es una medición de CPU del grupo en un instante. El archivo es una **serie temporal univariada**: una variable numérica observada secuencialmente, acompañada de su marca temporal. Se mantienen los nombres originales porque coinciden con la guía.

| Nombre | Significado | Tipo conceptual | Tipo al cargar / después de convertir | Unidad | Valores posibles |
|---|---|---|---|---|---|
| timestamp | Instante de medición | Temporal | Cadena / datetime con UTC | Fecha y hora; UTC asumida | Fechas válidas; intervalo observado de cinco minutos |
| value | Utilización promedio de CPU del grupo AWS | Cuantitativa continua | float64 / float64 | Porcentaje interpretado según la guía | Esperado: 0 a 100; observado: 11,529 a 100 |

El CSV no incorpora metadatos de unidad ni zona horaria. Por eso la interpretación porcentual y UTC se documentan como supuestos, no como atributos confirmados por el archivo.

## 5. Población, muestra y alcance
- **Población objetivo posible:** todas las mediciones de CPU del grupo definido durante el período operativo de interés.
- **Datos observados:** las 18 050 mediciones disponibles en el CSV.
- **Cobertura:** una serie del 14/05/2014 a las 01:14 al 15/07/2014 a las 17:19, con UTC asumida para procesar.
- **Unidad de análisis:** medición de CPU en un instante.

**Discusión crítica: ¿Es correcto afirmar que la muestra representa todos los servidores de una organización?**

No. El archivo contiene una serie agregada de un grupo concreto. No se conoce un mecanismo de selección que garantice representatividad de todos los servidores, no hay identificadores individuales y el período es limitado. Una cobertura interna completa no convierte al archivo en una muestra representativa de la organización; tampoco acredita cobertura fuera de su ventana.

## 6. Carga e inspección inicial
### Código recomendado en el PDF

### Respuesta a la tarea de la sección 6
Se registran **18 050 filas y 2 columnas**, llamadas `timestamp` y `value`. `timestamp` se carga como texto y `value` como `float64`; la salida de `info()` muestra el nombre exacto del tipo de texto de la versión instalada de pandas. El rango numérico observado es **11,529 a 100**, con media **38,283** y mediana **32,001**.

La estructura coincide con la descripción del dataset: una marca temporal y una métrica numérica por fila. La ausencia de una columna de servidor confirma que este archivo no permite comparar máquinas individualmente. El `None` mostrado después de `info()` es el valor de retorno de esa función al envolverla en `print`, tal como hace el PDF; no es un dato faltante.

## 7. Conversión temporal y ordenamiento
### Código recomendado en el PDF

### Interpretación y decisión sobre zona horaria
La conversión produce **0 fechas inválidas y 0 valores no numéricos**. `timestamp` queda como fecha y hora con UTC y `value` permanece numérica. El ordenamiento dispone las observaciones cronológicamente.

Se utiliza `utc=True` para seguir el procedimiento del PDF. Como las cadenas originales no especifican zona, esta decisión **asigna UTC como supuesto de procesamiento**; no demuestra que las mediciones originales se capturaran en UTC. No se interpretan como hora local peruana ni se aplica conversión local.

## 8. Calidad estructural del dato
### Código recomendado en el PDF

### Interpretación y regla de tratamiento
Hay **0 filas duplicadas, 0 marcas duplicadas, 0 faltantes por columna y 0 valores fuera de [0, 100]**. El rango temporal es del **14/05/2014 01:14 al 15/07/2014 17:19** con UTC asumida. El mínimo es **11,529** y el máximo **100**.

Se conservan todas las filas; no se imputan faltantes ni se eliminan duplicados automáticamente. Los picos que llegan a 100 son plausibles bajo la interpretación porcentual y se mantienen. Si aparecieran incidencias en otro archivo, se contrastarían con la fuente antes de corregirlas o excluirlas. La tabla con conteos y porcentajes se incluye en la actividad 13.3.

## 9. Regularidad de la serie
### Código recomendado en el PDF

### Interpretación
La moda, mediana, mínimo y máximo de los intervalos son **cinco minutos**. Las **18 049 transiciones** tienen esa duración. Para contrastar la frecuencia con toda la ventana se construye una malla, incluidos ambos extremos, y se compara con los instantes existentes. Además, se comprueba el orden del archivo antes de la conversión y ordenamiento.

### Código complementario: comprobación de huecos y orden original

**Resultado:** 18 050 puntos esperados y observados, cero huecos y cero instantes fuera de la malla; cobertura interna de 100 %. El original ya estaba ordenado. Esto acredita regularidad dentro del archivo, pero no demuestra que el monitoreo estuviera completo antes del primer registro o después del último.

## 10. Visualización de la serie
### Código recomendado en el PDF

### Descripción solicitada: tendencia, variabilidad, picos, cambios bruscos y estabilidad
- **Tendencia:** no hay una tendencia lineal única. Se repiten fases de crecimiento de los picos y descensos de nivel.
- **Variabilidad:** la base frecuente está cerca de 30-35, pero la dispersión varía a lo largo del período. La desviación global es 15,639 bajo la interpretación porcentual.
- **Picos:** el máximo es 100, alcanzado en 425 mediciones (2,35 % del total). Se conservan porque un valor alto no implica por sí mismo un error.
- **Cambios bruscos:** el nivel aumenta alrededor del 12 de julio y cae hacia el final del 14 de julio. La media del 12 de julio es 57,98 frente a 40,19 el día anterior. El 15 de julio, parcial, tiene media 13,78.
- **Períodos de estabilidad relativa:** antes del 12 de julio aparece una base recurrente cercana a 30 entre los picos. Tras la caída final, las mediciones se concentran alrededor de un nivel más bajo, aunque siguen existiendo oscilaciones. La desviación del último día es 3,19.

La gráfica describe cambios, pero no demuestra qué los causó. Para asociarlos a autoescalado o incidentes harían falta registros operativos. El eje temporal mantiene UTC asumida en la sección 7.

## 11. Resumen por intervalo temporal
### Código recomendado en el PDF

### Respuestas a las preguntas de la sección 11
**¿Qué información se pierde al agregar de la frecuencia original a una frecuencia horaria?**

Se pierde el orden exacto de las mediciones de cinco minutos dentro de la hora, la posición y duración de los picos y las oscilaciones breves. El máximo conserva la amplitud del pico, pero no su momento exacto ni su duración. La media horaria puede suavizar episodios relevantes; por eso debe conservarse la serie original.

**¿Cómo distinguir una media baja con picos breves de una carga estable?**

Se comparan media, mediana, máximo, desviación y gráfica. Una media moderada con máximo elevado y dispersión apreciable puede corresponder a picos breves; una carga estable tendrá poca dispersión y un máximo cercano al nivel habitual. Como ejemplo, el 16 de mayo tiene media **35,66**, máximo **100** y desviación **9,51**, por lo que la media sola oculta el pico.

**¿Qué significa count en cada intervalo y cómo ayuda a detectar cobertura incompleta?**

`count` es el número de valores no nulos usados en el resumen. Con frecuencia de cinco minutos se esperan 12 por hora completa y 288 por día completo. Un conteo inferior exige revisar faltantes o límites parciales del archivo. En este caso, los intervalos internos están completos; los extremos tienen menos observaciones porque la ventana comienza y termina dentro de una hora y un día.

### Código complementario: cobertura de cada intervalo
Se conservan los resúmenes originales del PDF. Se crean tablas adicionales con cobertura interna y cobertura de la hora o día completos. Los intervalos son cerrados por la izquierda y etiquetados por su inicio. `std` es la desviación estándar muestral, con `ddof=1`.

**Resultado:** 1 505 intervalos horarios y 63 diarios. La primera hora tiene 10 puntos y la última 4 (83,33 % y 33,33 % de una hora completa). El primer día tiene 274 puntos y el último 208 (95,14 % y 72,22 % del día completo). Respecto a los puntos esperados dentro de la ventana, todos tienen 100 % de cobertura.

## 12. De serie temporal a datos de sección transversal
**Serie temporal:** se observa una métrica del mismo grupo en muchos instantes. Es la estructura del archivo utilizado.

**Sección transversal:** se observarían distintos servidores o grupos en el mismo instante o período comparable. Se necesitarían identificador de servidor, métrica y unidad homogéneas, período de agregación equivalente y cobertura conocida.

Agrupar esta serie por día no la convierte en sección transversal: los días siguen siendo momentos de observación del mismo recurso. No se puede comparar el rendimiento entre servidores individuales con este CSV.

## 13. Actividades del laboratorio
### 13.1. Ficha técnica
| Elemento | Registro |
|---|---|
| Fuente | Kaggle indicado por la guía; corpus NAB, realKnownCause |
| Archivo | cpu_utilization_asg_misconfiguration.csv |
| Versión / fecha de descarga | No documentadas |
| Fecha de consulta de referencias | 13/10/2026 |
| Período | 14/05/2014 01:14 a 15/07/2014 17:19; UTC asumida |
| Dimensiones | 18 050 filas y 2 columnas |
| Variables | timestamp y value |
| Frecuencia observada | Cinco minutos |

La huella del archivo y versiones de software se exportan en la sección 14.

### 13.2. Modelo de datos
Las unidades de observación y análisis son una medición de CPU del grupo en un instante. La población objetivo son las mediciones de ese grupo durante el período operativo de interés y los datos observados son los 18 050 registros disponibles. Las secciones 4 y 5 desarrollan la estructura, el diccionario y las limitaciones de representatividad.

### 13.3. Auditoría de calidad
Código complementario para presentar los controles con conteos, denominadores y porcentajes. Las incidencias pueden solaparse y no se suman como registros distintos. Duplicados cuenta repeticiones posteriores a la primera aparición.

### 13.4. Análisis temporal
La sección 10 contiene la gráfica recomendada y su interpretación. La sección 11 contiene los resúmenes horarios y diarios y las respuestas sobre agregación y cobertura. Se exportan las tablas completas en la sección 14.

### 13.5. Decisiones reproducibles
| Acción | Decisión y justificación |
|---|---|
| Conservar | Las 18 050 filas, incluidos los extremos plausibles y valores iguales a 100 |
| Corregir | Solo convertir tipos y ordenar con el código del PDF; no se altera el valor medido |
| Excluir | Ninguna fila: no se detectaron incidencias que justifiquen su exclusión |
| Imputar | No se imputa: no hay huecos ni nulos |
| Marcar | Se identifican horas y días parciales por los límites del archivo; las marcas de CPU ≥90 e igual a 100 son exploratorias, no etiquetas oficiales de anomalía |

Si aparecieran duplicados, fechas inválidas o huecos en una nueva versión, se investigaría su causa antes de continuar. No se aplicaría una corrección silenciosa.

### 13.6. Informe técnico
**Hallazgos:** la serie tiene 18 050 mediciones cada cinco minutos, sin incidencias en los controles estructurales y con cobertura interna completa. La media es 38,283, la mediana 32,001 y la desviación estándar 15,639. Los valores altos elevan la media respecto a la mediana. Hay 545 registros ≥90 (3,02 %) y 425 iguales a 100 (2,35 %). La gráfica muestra cambios de nivel en julio y una caída final.

**Limitaciones:** se dispone de un promedio de grupo, sin identificadores de servidor ni eventos operativos. No se conocen la versión exacta y fecha de descarga; unidad y zona horaria no están declaradas en el CSV. La cobertura del 100 % solo corresponde a la ventana observada. Un extremo plausible no prueba un error y un cambio de nivel no identifica su causa.

**Recomendaciones de instrumentación:** registrar grupo y servidor, unidad, zona horaria original, estadístico de CloudWatch, período de agregación, número de instancias y eventos de autoescalado. Incorporar tráfico, latencia y despliegues para contrastar hipótesis; monitorizar cobertura y retraso de ingestión. Diseñar alertas considerando persistencia y contexto, no solo picos aislados.

**Conclusión:** los datos son aptos para descripción temporal dentro de su ventana y se conservan íntegros. Las conclusiones deben limitarse al recurso y período observados.

## 14. Entregables
1. Este notebook ejecutado, organizado según las 16 secciones del PDF.
2. Diccionario de datos: sección 4 y archivo CSV.
3. Auditoría con conteos y porcentajes: sección 13.3 y archivo CSV.
4. Gráfico de la sección 10 y tablas horarias y diarias de la sección 11, también exportados.

### Código complementario: exportación y comprobaciones
Este bloque guarda los entregables sin modificar el código recomendado de las secciones 6-11 ni sobrescribir el CSV de entrada.

## 15. Criterios de evaluación
| Criterio del PDF | Puntaje máximo | Evidencia en este desarrollo |
|---|---:|---|
| Identificación de unidad de observación, variables y alcance | 4 | Secciones 3, 4, 5 y 12 |
| Auditoría de calidad y justificación de reglas | 5 | Secciones 7, 8, 9, 13.3 y 13.5 |
| Código reproducible y correcto | 4 | Código recomendado ejecutado en 6-11; controles en 14 |
| Visualización y resúmenes temporales | 3 | Secciones 10 y 11 |
| Interpretación crítica, limitaciones y comunicación | 4 | Respuestas en 3, 5, 10 y 11; informe en 13.6 |
| **Total de la rúbrica** | **20** | No se asigna una calificación propia |

## 16. Referencias y trazabilidad
- **Guía local:** *Semana 3_Monitoreo_Infraestructura_Kaggle.pdf*, secciones 1-16. Sus bloques de código se utilizan en las secciones 6-11 de este notebook.
- **Numenta, NAB:** [repositorio](https://github.com/numenta/NAB) y [README del corpus](https://github.com/numenta/NAB/blob/master/data/README.md), sección `realKnownCause`, consultados el 13/10/2026. La descripción respalda que la métrica corresponde al promedio de CPU de un grupo, pero no confirma la versión de la copia local.
- **Josh, Kaggle:** [Time Series Anomaly Detection](https://www.kaggle.com/code/joshuaswords/time-series-anomaly-detection), enlace proporcionado por la guía. Sus celdas no pudieron verificarse en la consulta disponible.
- **pandas:** [documentación](https://pandas.pydata.org/docs/), referencia incluida en el PDF.

La versión concreta y la fecha de descarga permanecen como **no documentadas**. No se inventan ni se deducen de la fecha de modificación del archivo. La ficha técnica guarda SHA-256 y versiones de software; las cifras de las respuestas corresponden a esa copia del dataset. Si se sustituye el CSV, deben revisarse también las respuestas escritas después de ejecutar nuevamente los cálculos.