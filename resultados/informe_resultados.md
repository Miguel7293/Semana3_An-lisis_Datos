# Informe técnico · Semana 3

### Resultados principales
- Se analizaron **18,050 filas y 2 columnas**, entre **2014-05-14 01:14:00+00:00** y **2014-07-15 17:19:00+00:00**.
- Todas las **18,049 transiciones** duran **0 days 00:05:00**. Hay **18,050 puntos esperados**, sin huecos internos, duplicados, nulos ni valores fuera de [0, 100]. La cobertura interna es **100 %**.
- Media: **38.283**; mediana: **32.001**; desviación estándar: **15.639** puntos porcentuales bajo el supuesto de unidad. Mínimo: **11.529**; máximo: **100.000**.
- El 50 % central está entre **30.789** y **35.660**. La media mayor que la mediana es compatible con la influencia de picos altos.
- Hay **545 mediciones ≥90 (3.02 %)** y **425 iguales a 100 (2.35 %)**. Son observaciones del promedio del grupo; no miden cuántos servidores se saturaron.
- Se generaron **1,505 intervalos horarios** y **63 diarios**. El primer día tiene 274 de 288 puntos (95.14 %) y el último 208 de 288 (72.22 %). Ambos tienen cobertura interna completa.

### Tendencia, variabilidad y cambios
La serie presenta una base frecuente cercana a 30-35 y picos repetidos, con variabilidad cambiante; no se resume adecuadamente como una tendencia lineal única. Las medias de los días completos anteriores al 12 de julio se mantienen aproximadamente entre 33.04 y 41.52. El 12 y 13 de julio aumentan a **57.98** y **56.75**. El 14 registra **52.35** y el último día observado baja a **13.78**, con desviación **3.19**, aunque es un día parcial.

El mayor aumento entre mediciones consecutivas es **78.833** puntos en 2014-07-14 05:09:00+00:00, y la mayor caída es **-69.045** puntos en 2014-06-21 01:14:00+00:00. Son cambios descriptivos, no pruebas de su causa. El nombre del archivo sugiere un contexto de configuración de autoescalado, pero la serie sola no permite atribuir cada pico a una acción concreta.

### ¿Qué se pierde al agregar por hora?
Se pierde el orden dentro de la hora, la hora exacta de los picos y la duración de episodios breves. Aun conservando máximo y desviación, no se reconstruye la secuencia original. Por eso se conserva la serie de cinco minutos.

### ¿Cómo distinguir media baja con picos de carga estable?
Comparando media, mediana, máximo, desviación y gráfico. Por ejemplo, el 16 de mayo la media es **35.66**, el máximo **100.00** y la desviación **9.51**: una media moderada coexistió con un pico alto. Una carga estable tendría poca dispersión y máximo próximo al nivel habitual.

### ¿Qué aporta count?
Cuenta valores numéricos disponibles en cada intervalo. Aquí se compara con 12 por hora y 288 por día completo, y con los esperados dentro de la ventana. Sin esa comparación, una media podría representar pocos registros. La cobertura del 100 % solo se refiere a la malla entre el primer y último registro, no al período operativo fuera del archivo.

### ¿Qué no puede concluirse?
No se puede comparar servidores, inferir toda la infraestructura AWS, identificar la causa de los cambios, confirmar fallas ni predecir demanda futura con estos controles descriptivos. Tampoco se verifica la zona horaria o unidad solo por observar los valores.

### Recomendaciones de instrumentación
Registrar identificador de grupo y servidor, zona horaria original, unidad, estadístico de CloudWatch y período de agregación; conservar versión y fecha de descarga. Incorporar número de instancias, eventos de autoescalado, despliegues, tráfico y latencia para contrastar hipótesis. Monitorizar cobertura y retraso de ingestión. Si se definen alertas de CPU, considerar persistencia, contexto operativo y validación histórica en lugar de un único pico.

### Conclusión
El archivo es estructuralmente apto para análisis temporal descriptivo dentro de su ventana. Se conservan todas las mediciones. La calidad estructural no elimina las limitaciones de representatividad ni sustituye los metadatos operativos ausentes.
