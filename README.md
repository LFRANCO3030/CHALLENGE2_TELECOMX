# Acerca del Proyecto
## Objetivo
Aplicar recopilar, procesar y analizar los datos, utilizando Python y sus principales bibliotecas para extraer información valiosa, y desarrollar estrategias para reducir la evasión.

# Descripcion del Caso
Empresa del sector de telecomunicaciones que brinda servicios al mercado, en la actualizada presenta una preocupante tasa de evasion de clientes y desea identificar el motivo y estrategias de solucion 

# Ambiente de Desarrollo
Github : Control de versiones
Google Collab : Editor de Python
Matplotlib.pyplot, plotly.express para generacion de Graficos
Numpy
Pandas

# Extraccion
Clonamos repositorio de Datos de Github desde https://github.com/ingridcristh/challenge2-data-science-LATAM/blob/main/TelecomX_Data.json

# Transformacion(TR)
## TR1. Normalizacion de Datos

## Columnas JSON diccionario de diccionario
columnas_anidadas = ['customer', 'phone','internet','account']

## Crea un DataFrame vacío para guardar los resultados
df_normalizado = pd.DataFrame()

for columna in columnas_anidadas:
## Aplanar los datos JSON de la columna actual
    df_temp = pd.json_normalize(datos[columna])

## Concatenar los nuevos DataFrame a la normalizado
    df_normalizado = pd.concat([df_normalizado, df_temp], axis=1)

## Concatenar el DataFrame normalizado con el original (sin las columnas JSON)
df_json_normalizado = pd.concat([datos.drop(columnas_anidadas, axis=1), df_normalizado], axis=1)

## TR2. Depuracion de Data

### Valores unicos
valores_unicos = {}
for column in df_json_normalizado.columns:
    valores_unicos[column] = df_json_normalizado[column].unique()

for column, unique_values in valores_unicos.items():
    print(f"Valores Unicos por columna '{column}':")
    print(unique_values)
    print("-" * 30)

### Valores duplicados en columnas con datos unicos
valores_unicos = {}
for column in df_json_normalizado.columns:
    valores_unicos[column] = df_json_normalizado[column].unique()

for column, unique_values in valores_unicos.items():
    print(f"Valores Unicos por columna '{column}':")
    print(unique_values)
    print("-" * 30)

## TR3. Tratamiento de Incoherencias
### TR3.1 Clientes sin contrato y con estado de activo
registros_con_valor = df_json_normalizado.loc[(df_json_normalizado['tenure'] == 0)  & (df_json_normalizado['Churn']== 'No')]
print(registros_con_valor)

condicion_incoherente=df_json_normalizado.loc[(df_json_normalizado['tenure'] == 0) & (df_json_normalizado['Churn'] == 'No')]

Eliminar los registros incoherentes
df_json_normalizado = df_json_normalizado[(df_json_normalizado['tenure'] != 0) | (df_json_normalizado['Churn'] != 'No')]

### TR3.2 Validacion de caluclo correcto de meses de contrato con relacion a Cargos Totales y Cargos Mensuales
condicion_caso = (
    (df_json_normalizado['Charges.Total'].isna() | df_json_normalizado['Charges.Monthly'].isna()) &
    (df_json_normalizado['tenure'] != 0)
)

cantidad_casos = condicion_caso.sum()
print("Número de registros con montos nulos y tenure distinto de 0:", cantidad_casos)

### TR3.3 Posibles casos de valores nulos en columnas de Cargos Totales y Cargos Mensuales
df_json_normalizado['Charges.Total'] = df_json_normalizado['Charges.Total'].fillna(0)
df_json_normalizado['Charges.Monthly'] = df_json_normalizado['Charges.Monthly'].fillna(0)
tolerancia = 1
df_json_normalizado.loc[valor_incoherente, 'tenure'] = df_json_normalizado.loc[valor_incoherente, 'valor_correcto']

df_json_normalizado.drop(columns=['valor_temp', 'valor_correcto'], inplace=True)

df_json_normalizado.drop(columns=['resultado_calculado'], inplace=True)

### TR3.4 Creacion de columna de cuentas diarias
df_json_normalizado = df_json_normalizado.copy()
df_json_normalizado['Cuentas_Diarias'] = (df_json_normalizado['Charges.Monthly'] / 30).round(2)
df_json_normalizado.head()

### TR3.5 Transformacion de vlores de texo SI NO en binarios
def convertir_yes_no(df, columnas):
    for col in columnas:
        df[col] = (
            df[col]
            .str.strip()
            .str.lower()
            .map({'yes':1, 'no':0})
        )
    return df
    columnas_binarias = ['Churn', 'Partner', 'Dependents', 'PhoneService','MultipleLines','DeviceProtection','TechSupport','StreamingTV','StreamingMovies','PaperlessBilling']

df_json_normalizado = convertir_yes_no(df_json_normalizado, columnas_binarias)
convertir a enteros
columnas_binarias = ['Churn', 'Partner', 'Dependents', 'PhoneService','MultipleLines','DeviceProtection','TechSupport','StreamingTV','StreamingMovies','PaperlessBilling']
posibles casos de NaN
df_json_normalizado[columnas_binarias] = df_json_normalizado[columnas_binarias].fillna(0)
Convertir a entero
df_json_normalizado[columnas_binarias] = df_json_normalizado[columnas_binarias].astype(int)

# Carga y Analisis (CA)
### CA1 Descripcion de Dataframe 
df_json_normalizado.describe()

### CA2 Grafico de clientes Activos vs Evadidos
Grafico de Evasion de Clientes
import plotly.express as px

Contamos cuántos hay por estado
conteo_churn = df_json_normalizado['Churn'].value_counts().reset_index()
conteo_churn.columns = ['Churn', 'Cantidad']

Convertimos 0 y 1 a etiquetas amigables
conteo_churn['Estado'] = conteo_churn['Churn'].map({0: 'Activos', 1: 'Evadidos'})

Gráfico
fig = px.bar(conteo_churn, x='Estado', y='Cantidad',
             title='Clientes Activos vs Evadidos',
             text='Cantidad')
fig.show()
### CA3 Grafico de clientes Evadidos por Categorias gender,Contract,pyamentmethod,internetservice

import plotly.express as px
columnas_categoricas = ['gender', 'Contract', 'PaymentMethod', 'InternetService']

for col in columnas_categoricas:
    Agrupamos por la columna y Churn
    tabla = df_json_normalizado.groupby([col, 'Churn']).size().reset_index(name='Cantidad')

    Convertimos la columna Churn a nombre legible
    tabla['Estado'] = tabla['Churn'].map({0:'Activo', 1:'Evadido'})

    fig = px.bar(
        tabla,
        x=col, y='Cantidad',
        color='Estado',
        barmode='group',                       # también puedes probar 'stack'
        title=f'Distribución de Evaciones por {col}'
    )
    fig.show()

### CA4 Conteo y Valor de Chargos Totales de clientes evadidos
import plotly.express as px

fig = px.histogram(
    df_json_normalizado,
    x='Charges.Total',
    color='Churn',              # 0 o 1
    nbins=50,
    barmode='overlay',
    opacity=0.7,
    title='Distribución de Gasto Total por cantidad de clientes Evasivos'
)
fig.show()

### CA5 Boxplot de Chargos Totales

fig = px.box(
    df_json_normalizado,
    x='Churn',
    y='Charges.Total',
    points="all",  # opcional: muestra outliers
    title='Distribución de Gasto Total por cantidad de Evasion'D
)
fig.show()


### CA6 Boxplot de Chargos Totales y Cargos Mensuales

variables_numericas = ['Charges.Total', 'Charges.Monthly', 'tenure']

for var in variables_numericas:
    fig = px.box(
        df_json_normalizado,
        x='Churn',

        
        y=var,
        points="outliers",
        title=f'Distribución de {var} por Evasion'
    )

# Conclusiones
A continuacion se detalle las principales productos,contratos y servicios afectados que son motivo de evasion de clientes.

La cantidad de clientes que deciden dejar de usar nuestro servicio reprreesenta un 25% del total de clientes
Servicio de pago: Los clientes que se alejaron escogieron el servicio de **Pago de Cheque Electronico**
Tipo de Contrato: Los clientes que que renunciaron a seguir con suentro servicio utilizaron un **tipo de contrato Mensual**
Tipo de Servicio contratado: Los clientes que renuncian fueron quienes utilizaron el **servicio de Fibra Optica** cabe indicar que la demanda de este servicio es alto, lo cual representa un producto estrella para el mercado 



    fig.show()

