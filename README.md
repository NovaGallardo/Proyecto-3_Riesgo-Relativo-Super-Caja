# Proyecto 3 - Riesgo Relativo: Análisis de Crédito en Super Caja

## Enlaces y Recursos

- [Bitácora](https://www.notion.so/Proyecto-3-Riesgo-Relativo-3d3f6110a66d4f1dbe5653251ed14a5b?pvs=4)
- [Dashboard Looker Studio](https://lookerstudio.google.com/u/0/reporting/ed2a3bf6-df72-411f-8e4c-a914a24da3f7/page/p_4g97w6orkd/edit)
- [Presentación en Google Slides](https://docs.google.com/presentation/d/139iLcsgYCTyhAVKJ1rT15xF5bn_5RNTVsSqnRtAGSrE/edit?usp=sharing)


Este proyecto tiene como objetivo automatizar el proceso de análisis de crédito en el banco 'Super Caja', mejorando la eficiencia, precisión y rapidez en la evaluación de solicitudes de crédito. Con la implementación de técnicas avanzadas de análisis de datos, se busca clasificar a los solicitantes en diferentes categorías de riesgo basadas en su probabilidad de incumplimiento.

## Contexto

Debido a la disminución de las tasas de interés en el mercado, el banco ha experimentado un aumento en las solicitudes de crédito. Sin embargo, el análisis manual de cada solicitud ha sobrecargado al equipo de análisis de crédito, resultando en un proceso ineficiente y demorado. Además, la tasa de incumplimiento ha crecido, aumentando el riesgo asociado con los préstamos otorgados.

El objetivo de este proyecto es automatizar el análisis de crédito utilizando un sistema basado en un score crediticio, el cual permitirá clasificar a los clientes en diferentes categorías de riesgo. Esto ayudará al banco a tomar decisiones informadas y mitigar riesgos futuros.

## Objetivo

- **Automatización**: Reducir el tiempo y esfuerzo manual en el análisis de solicitudes de crédito.
- **Riesgo Relativo**: Evaluar el riesgo relativo de cada cliente para clasificarlo en categorías de riesgo.
- **Score crediticio**: Calcular un score basado en la probabilidad de incumplimiento de los solicitantes.
- **Integración de métricas**: Incluir una métrica de pagos atrasados ya existente para fortalecer el modelo predictivo.

## Herramientas y Tecnologías Utilizadas

- **BigQuery**: Manipulación y consulta de grandes volúmenes de datos.
- **SQL**: Consultas avanzadas para la preparación de datos.
- **LookerStudio**: Visualización de los resultados a través de dashboards interactivos.
- **Loom y Google Slides**: Presentación de resultados y reportes.

## Hipótesis

- Los clientes más jóvenes tienen un mayor riesgo de impago.
- Aquellos con más préstamos activos son más propensos a retrasarse en pagos.
- Las personas que han retrasado sus pagos por más de 90 días tienen mayor riesgo de ser malos pagadores.

## Composición del Dataset

El dataset está compuesto por variables como edad, género, nivel salarial, historial de préstamos y otras características clave que se analizaron para la evaluación de riesgo. Se utilizaron técnicas estadísticas avanzadas para la correlación entre estas variables y el riesgo de incumplimiento.

## Composición del Dataset

### Default Dataset

| Nombre de la Columna  | Descripción                                        |
|-----------------------|----------------------------------------------------|
| user_id               | Identificador único para cada usuario              |
| default_flag          | Indicador de si el usuario ha incumplido (1) o no (0) |

### Loans Detail Dataset

| Nombre de la Columna                              | Descripción                                                          |
|---------------------------------------------------|----------------------------------------------------------------------|
| user_id                                           | Identificador único para cada usuario                                |
| more_90_days_overdue                              | Número de veces que el usuario ha tenido más de 90 días de retraso en un pago |
| using_lines_not_secured_personal_assets           | Indicador de uso de activos personales no asegurados para préstamos  |
| number_times_delayed_payment_loan_30_59_days      | Número de veces que el usuario ha retrasado pagos entre 30-59 días   |
| debt_ratio                                        | Relación de deuda del usuario                                        |
| number_times_delayed_payment_loan_60_89_days      | Número de veces que el usuario ha retrasado pagos entre 60-89 días   |

### Loans Outstanding Dataset

| Nombre de la Columna  | Descripción                                   |
|-----------------------|-----------------------------------------------|
| loan_id               | Identificador único para cada préstamo        |
| user_id               | Identificador único para cada usuario         |
| loan_type             | Tipo de préstamo (por ejemplo, asegurado, no asegurado) |

### User Info Dataset

| Nombre de la Columna  | Descripción                                        |
|-----------------------|----------------------------------------------------|
| user_id               | Identificador único para cada usuario              |
| age                   | Edad del usuario                                   |
| sex                   | Género del usuario                                 |
| last_month_salary     | Salario del usuario en el último mes               |
| number_dependents     | Número de dependientes que tiene el usuario        |


## Proceso

### 1. Procesamiento y Preparación de Datos

- [Identificación y manejo de valores nulos](./Procesamiento%20y%20Preparaci%C3%B3n%20de%20Datos/valores_nulos.md)
- [Identificación y manejo de valores duplicados](./Procesamiento%20y%20Preparaci%C3%B3n%20de%20Datos/identificar%20y%20manejar%20valores%20duplicados.md)
- [Agrupación de datos según variables categóricas](./Procesamiento%20y%20Preparaci%C3%B3n%20de%20Datos/Identificar%20y%20manejar%20datos%20discrepantes%20en%20variables%20categ%C3%B3ricas.md)
- [Aplicación de medidas de tendencia central (moda, media, mediana) y dispersión (desviación estándar)](./Procesamiento%20y%20Preparaci%C3%B3n%20de%20Datos/Identificar%20y%20manejar%20datos%20discrepantes%20en%20variables%20num%C3%A9ricas.md)
- [Visualización de la distribución de los datos y creación de nuevas variables para enriquecer el análisis](./Procesamiento%20y%20Preparaci%C3%B3n%20de%20Datos/Crear%20nuevas%20variables.md)

### 2. Análisis Exploratorio

- [Construcción de tablas auxiliares para facilitar el análisis](./Procesamiento%20y%20Preparaci%C3%B3n%20de%20Datos/Construir%20tablas%20auxiliares.md)
- Cálculo de cuartiles, deciles y percentiles
- Estudio de la correlación entre variables numéricas relevantes, utilizando la correlación de Pearson para identificar relaciones significativas

## 3. Aplicación del Modelo de Riesgo Relativo

- [Implementación de un análisis de riesgo relativo para calcular la probabilidad de incumplimiento](./Riesgo%20Relativo/Calcular%20Riesgo%20Relativo.md)


## Resultados

El sistema automatizado permite al banco clasificar a los solicitantes de crédito de manera eficiente, reduciendo significativamente el tiempo de procesamiento de solicitudes y ayudando a mitigar el riesgo de incumplimiento. La integración de las métricas existentes de pagos atrasados ha fortalecido la capacidad predictiva del modelo.

