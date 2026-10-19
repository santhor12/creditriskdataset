# Credit Risk: limpieza y análisis exploratorio

Proyecto del curso **Data Science I: Fundamentos de la Ciencia de Datos** (Coderhouse). Esta es la pre-entrega del Módulo 5, donde limpio un dataset real con Pandas y hago un primer análisis exploratorio.

Autor: Santiago Lliteras

## De dónde vienen los datos

Uso el [Credit Risk Dataset](https://www.kaggle.com/datasets/laotse/credit-risk-dataset) de Kaggle. Cada fila es una solicitud de préstamo personal con datos de la persona (edad, ingreso, tipo de vivienda, antigüedad laboral, historial crediticio) y del préstamo (monto, destino, grado, tasa de interés). La columna `loan_status` indica si el préstamo terminó en default (1) o se pagó con normalidad (0).

El archivo original tiene 32.581 filas y 12 columnas. Está en este repositorio como `credit_risk_dataset.csv`, y el notebook lo lee directo desde GitHub.

La pregunta que guía el proyecto: ¿se puede predecir si un préstamo va a caer en default a partir del perfil de la persona y de las condiciones del préstamo?

## Qué hay en el repositorio

| Archivo | Para qué sirve |
|---|---|
| `credit_risk_dataset.csv` | Dataset original, sin modificar |
| `Pre_entrega_Modulo5_Santiago_Lliteras.ipynb` | Notebook con el diagnóstico, la limpieza y las agregaciones |
| `README.md` | Este archivo |

## Cómo ejecutarlo

Abrir el notebook en Google Colab y elegir *Entorno de ejecución → Ejecutar todas*. No hace falta subir ningún archivo, porque el CSV se descarga solo desde GitHub. Solo usa `pandas`.

## Decisiones de limpieza

| Problema | Qué encontré | Qué hice y por qué |
|---|---|---|
| Valores imposibles | 5 edades de más de 100 años (llegan a 144) y 2 antigüedades laborales de más de 60 (llegan a 123) | Eliminé esas 7 filas con filtros booleanos, porque no pueden ser reales |
| Duplicados | 165 filas duplicadas | `drop_duplicates()` |
| Nulos en `person_emp_length` | 887 (2,7 %) | `fillna` con la **mediana** (4 años). La media (4,78) está estirada por quienes llevan muchos años (sesgo de 1,25) y la moda es 0 |
| Nulos en `loan_int_rate` | 3.094 (9,5 %) | `fillna` con la **media de cada grado** (`groupby` + `transform`). La tasa va de 7,33 % en el grado A a 20,25 % en el G, así que una media general sería poco realista |
| Tipo de dato | `person_emp_length` era `float64` solo por los nulos | `astype(int)`, tras comprobar que todos los valores eran enteros |
| Índice | Quedó con huecos al borrar filas | `reset_index(drop=True)` |

Resultado: 32.409 filas y 12 columnas, sin nulos y sin duplicados.

## Hallazgos principales

Con `groupby` y `pivot_table`, llamando **tasa de default** a la proporción de préstamos con `loan_status = 1`:

- **El grado es lo que más separa el riesgo.** La tasa de default va de 10,0 % en el grado A a 20,8 % en el C, y salta a 59,0 % en el D. El grado G llega a 98,4 %, pero son solo 64 préstamos.
- **El tipo de vivienda también pesa.** Quienes alquilan tienen 31,6 % de default, contra 12,6 % con hipoteca y 7,5 % con vivienda propia. La diferencia se mantiene dentro del mismo grado: en el grado A es 17,5 % contra 4,4 %.
- **El destino del préstamo importa.** Consolidar deudas (28,7 %), gastos médicos (26,8 %) y mejoras del hogar (26,2 %) son los más riesgosos. Emprendimiento (14,9 %) y educación (17,3 %), los menos.
- **Quienes caen en default piden préstamos que pesan más sobre su ingreso.** La mediana de `loan_percent_income` es 0,24 contra 0,13, con una tasa de 13,46 % contra 10,65 % y un ingreso mediano de 41.655 contra 60.000.

## Limitaciones

- Los valores imputados (887 antigüedades y 3.094 tasas) no son observaciones reales. Además, en las filas sin antigüedad laboral el default es 31,7 %, contra 21,6 % en el resto, y esa información se pierde al rellenar con la mediana.
- Algunos grupos son muy chicos (grado G con 64 préstamos, vivienda `OTHER` con 106), por lo que sus porcentajes son poco confiables.
- Al llegar al modelado habrá que revisar la imputación: lo habitual es calcular las medianas y medias solo con los datos de entrenamiento.
