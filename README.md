# thePower-DataAnalytics-FinalProject

¿Qué hace que un corredor sea más rápido?

Análisis del rendimiento en running a partir de datos reales de entrenamiento, usando el dataset abierto de GoldenCheetah OpenData.

Proyecto final del Bootcamp de Data & Analytics.

Repositorio de GitHub: https://github.com/alexgarrido-dev/thePower-DataAnalytics-FinalProject 

Dashboard en vivo (Google Sheets): https://docs.google.com/spreadsheets/d/1rueumvMPts6Jim_5PtP_EDuwaGF1qMu6/edit?usp=drive_link

Objetivo del proyecto

Pregunta de investigación:

¿Qué factores relacionados con las características y el entrenamiento de un corredor están asociados con un mejor rendimiento en running?

Variable objetivo: pace (ritmo, en min/km). A menor valor, mejor rendimiento.

El proyecto cubre el ciclo completo de un análisis de datos: unión de dos fuentes distintas, limpieza y transformación profunda, análisis exploratorio y estadístico, visualización, dashboard operativo e informe de conclusiones.

📂 Estructura del repositorio
proyecto-final/
├── data/
│   ├── raw/
│   │   ├── athletes.xlsx              ← Fuente 1: perfil de atletas (Excel)
│   │   └── activities_running.csv     ← Fuente 2: actividades de running (CSV)
│   └── processed/
│       └── final_dataset.csv          ← Dataset final tras merge + limpieza
├── notebooks/
│   ├── FinalProject_Import&Merge.ipynb     ← Carga, merge, limpieza y transformación
│   └── FinalProject_Analysis&Visual.ipynb  ← EDA, estadística, visualizaciones y regresión
├── dashboard/
│   └── [dashboard_running_OK.xlsx ](https://docs.google.com/spreadsheets/d/1rueumvMPts6Jim_5PtP_EDuwaGF1qMu6/edit?usp=drive_link)          ← Dashboard exportado desde Google Sheets - por tamaño no me deja subirlo al repositorio.
└── README.md

Algunos archivos de datos se han comprimido (.zip) en el repositorio por límites de tamaño de GitHub — descomprímelos antes de ejecutar los notebooks.

Fuentes de datos

Nombre:	athletes.xlsx	y activities_running.csv
Origen:	GoldenCheetah OpenData (Kaggle)	Mismo dataset, tabla de actividades
Formato:	Excel	y CSV
Contenido:	1 fila por atleta: edad, género, peso, nº de actividades por deporte & 1 fila por actividad de running: fecha, distancia, tiempo, HR, desnivel...
Clave de unión:	id	id

Nota sobre el origen de los datos:

El archivo activities.csv original de Kaggle contiene más de 2 millones de actividades de ~6.000 atletas, de múltiples deportes (ciclismo, running, natación...). Por límites de tamaño de archivo, se aplicó un filtro sport == 'Run' antes de subir el archivo al repositorio, reduciendo el CSV a las 238.558 actividades de running. Este es un pre-filtro documentado, no un paso de limpieza — la limpieza real (valores imposibles, duplicados, nulos) se aplica después, sobre este subconjunto, y queda recogida íntegramente en FinalProject_Import&Merge.ipynb.

Dataset completo original: GoldenCheetah OpenData en Kaggle

Metodología: transformación y limpieza

1. Selección de columnas relevantes

De athletes.xlsx (36 columnas originales, la mayoría métricas de potencia en ciclismo) se seleccionaron solo las relevantes para running: id, age, gender, activities, run, weightkg, weightstd.

De activities.csv (34 columnas) se seleccionaron mediante usecols en la lectura por chunks: id, file, date, age, gender, sport, data, workout_time, total_distance, elevation_gain, average_speed, average_hr.

2. Filtrado a running y merge
   
Filtrado de activities.csv a sport == 'Run' → 238.558 actividades.
merge con athletes por id (inner join) → 238.558 filas × 18 columnas.
Resueltas las columnas duplicadas age/gender (idénticas en ambas fuentes, 0 discrepancias) → se mantiene una sola versión de cada una.

4. Limpieza de fechas
   
Eliminadas actividades con fecha anterior a 2007 (2 registros con años imposibles como 1990 y 1999 — errores de reloj del dispositivo). El resto de fechas 2007-2018 se mantienen: son historial real importado por los usuarios al unirse a la plataforma.

5. Duplicados
   
17 filas exactamente duplicadas, eliminadas.

6. Cálculo de pace propio

average_speed del dataset original contenía errores de sensor severos (máximos de +500.000 km/h). Se descartó esa columna y se calculó una métrica propia, más fiable:

pace_min_km = (workout_time / 60) / total_distance

6. Filtrado de valores físicamente imposibles

Se eliminaron registros con:

total_distance ≤ 0 o > 100 km
workout_time ≤ 0 o > 6 horas
pace_min_km fuera del rango plausible 3–10 min/km

7. Corrección de age (columna con errores de captura)

Se detectaron tres patrones de error, cada uno corregido de forma distinta en vez de eliminarse directamente:

Valores negativos "normales" (ej. -25) → se corrigió el signo.
Valores tipo -9XX (ej. -965) → prefijo -9 colado por error; se recuperaron los 2 últimos dígitos como edad real.
Valores entre 1900-2020 → año de nacimiento introducido por error en el campo edad; se convirtió a edad real restando del año de la actividad (year - año_nacimiento).
Lo que seguía siendo imposible tras corregir (age < 10 o > 100), se eliminó.

8. Tratamiento de nulos
   
Columna	% nulos	Decisión
elevation_gain	~15%	Se mantiene NaN — refleja actividades sin sensor barométrico, no un error
average_hr	~19%	Se mantiene NaN — refleja actividades sin pulsómetro
workout_time, total_distance	<3%	Filas eliminadas (sin ellos no se puede calcular pace)

No se imputaron valores en elevation_gain ni average_hr para no fabricar datos inexistentes; los análisis que usan estas variables trabajan sobre el subconjunto de actividades donde sí están disponibles, indicándolo explícitamente.

9. Columnas calculadas

A nivel de actividad:

year, month, day_of_week — extraídas de date
pace_kmh — velocidad derivada del pace
distance_bucket — categoría de distancia (corta / media / larga / muy larga)

A nivel de atleta (agregado y unido de vuelta a la tabla principal):

avg_pace, best_pace — pace medio y mejor marca del atleta
weekly_km — km totales ÷ semanas activas
training_frequency — nº actividades ÷ semanas activas
experience_level — bins según nº de actividades de running (principiante / intermedio / avanzado)
Dataset final
	
Filas	216.549
Columnas	26
Atletas únicos	2.379
Rango temporal	2007–2020

Cumple ampliamente los requisitos mínimos del proyecto (>50.000 filas, ≥20 columnas).

Informe de análisis

Contexto y sesgos de la muestra

Antes de interpretar cualquier resultado, es importante señalar dos sesgos estructurales de esta muestra:

Desequilibrio de género: 96,9% hombres / 3,1% mujeres (74 atletas). GoldenCheetah es una app usada mayoritariamente por ciclistas con potenciómetro, un colectivo muy masculinizado. Ninguna conclusión sobre género es generalizable fuera de esta muestra.
Actividad vs. atleta: el 24,9% de atletas "avanzados" concentra el 82,6% de las actividades, mientras que el 48,9% "principiantes" apenas representa el 3,3%. Los análisis a nivel actividad reflejan sobre todo el comportamiento de corredores muy activos; los análisis a nivel atleta (1 fila = 1 atleta) dan una foto más equilibrada.
Distribución del rendimiento
Pace medio por atleta: 6:01 min/km (mediana).
El 10% más rápido de los atletas corre a 5:07 min/km o menos; el 10% más lento, a 7:43 min/km o más — una diferencia de 2:36 min/km entre extremos.
Entrenamiento vs. rendimiento
Variable	Correlación con avg_pace
weekly_km	-0,26
training_frequency	-0,16

El volumen de entrenamiento (km/semana) se asocia más con el rendimiento que la frecuencia (nº de sesiones). Ambas relaciones son de intensidad moderada-débil por separado.

Por nivel de experiencia, los atletas avanzados no solo corren más rápido de media (5:52 min/km) que los principiantes (6:28 min/km), sino que son mucho más consistentes (desviación estándar de 0,71 vs. 1,22).

Perfil del corredor

Edad (r=0,11) y peso (r=0,17): asociaciones positivas pero débiles con el pace — no son factores determinantes por sí solos en esta muestra.
Género: pace medio de 6:40 min/km (mujeres) vs. 6:14 min/km (hombres) — diferencia a interpretar con cautela dado el fuerte desequilibrio muestral.
Fisiología y condiciones de la actividad
Frecuencia cardíaca media (r=-0,35): la variable con mayor asociación individual con el pace de toda la muestra. Más esfuerzo cardiovascular dentro de una actividad concreta se asocia con mayor velocidad — es una relación de intensidad dentro de la sesión, no de "forma física general".
Desnivel acumulado (r=0,004): sin relación relevante en términos absolutos; se descartó como variable explicativa.
Distancia de la actividad: las tiradas largas (10-21km, >21km) muestran mejor pace medio (~5:33) que las cortas (<5km, ~6:10) — probablemente porque quien afronta distancias largas suele ser un corredor más entrenado, no por un efecto directo de la distancia.

Evolución temporal

El pace medio empeora progresivamente entre 2007 (5:34 min/km) y 2020 (5:59 min/km). Este patrón coincide con el fuerte crecimiento de usuarios de la plataforma en el mismo periodo (de <100 a 57.000+ actividades/año), por lo que es más plausible atribuirlo a un cambio en la composición de la muestra que a una tendencia real de rendimiento. La estacionalidad mensual es prácticamente inexistente (variación de apenas ~5 segundos entre el mejor y peor mes).

Conclusiones principales

El volumen semanal de kilómetros es el factor de entrenamiento más asociado al rendimiento, por encima de la frecuencia de sesiones.
La experiencia (medida como nº de actividades registradas) se asocia tanto con mejor pace como con mayor consistencia.
La frecuencia cardíaca dentro de una actividad es la variable con mayor asociación individual al ritmo — refleja intensidad de esfuerzo, no forma física.
Edad, peso y desnivel tienen un peso menor de lo esperado en esta muestra.

Limitaciones del estudio

Muestra con fuerte sesgo de género (96,9% hombres).
Muestra sesgada hacia usuarios de una app orientada a ciclismo con potenciómetro — puede no representar a corredores en general.
Las relaciones encontradas son asociaciones, no relaciones causales.
El pace y la distancia son variables recalculadas por el equipo a partir de datos crudos, no directamente del sensor.

Dashboard

Acceso directo al dashboard en Google Sheets: https://docs.google.com/spreadsheets/d/1rueumvMPts6Jim_5PtP_EDuwaGF1qMu6/edit?usp=drive_link

Incluye:

KPIs generales (nº actividades, nº atletas, pace medio)
Pace por nivel de experiencia y km/semana vs. pace (dispersión)
Frecuencia cardíaca vs. pace y pace por tipo de distancia
Pace por género y por rango de edad
Evolución temporal del pace y volumen de actividades
Notas metodológicas y disclaimers de sesgo integrados en el propio dashboard

Herramientas utilizadas

Python (pandas, numpy, matplotlib, seaborn, scipy, scikit-learn) — VS Code
Excel / Google Sheets — dashboard operativo
Jupyter Notebook — desarrollo del EDA y análisis

Cómo reproducir el análisis

bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn openpyxl
Ejecutar notebooks/FinalProject_Import&Merge.ipynb — carga las fuentes en bruto, hace el merge, limpia y genera data/processed/final_dataset.csv.
Ejecutar notebooks/FinalProject_Analysis&Visual.ipynb — carga el dataset final y reproduce el EDA, la estadística y la regresión.
Abrir el dashboard en Google Sheets directamente.

Autor

Alex Garrido
