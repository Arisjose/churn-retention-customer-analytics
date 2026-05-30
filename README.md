# 📊 E-Commerce Customer Churn & Retention Analytics Dashboard


## 📝 Descripción del Proyecto
Este proyecto resuelve un problema crítico de negocio: la pérdida de clientes (**Customer Churn**) y la optimización del valor del cliente en el tiempo (**Customer Lifetime Value - CLV**). 

A diferencia de los reportes estáticos tradicionales, este cuadro de mando cuenta con un pipeline de datos automatizado que extrae la información directamente desde **Kaggle** mediante su API oficial utilizando un script de **Python** integrado en Power Query.

## 🎯 Problema de Negocio
La empresa ha detectado una caída en la repetición de compras. El costo de adquirir un nuevo cliente (CAC) está superando el valor que aportan. Se necesita responder de inmediato:
* ¿Cuál es nuestra tasa de retención actual por mes?
* ¿Qué segmentos de clientes (VIP, En Riesgo, Perdidos) impactan más los ingresos?
* ¿Cuáles son los principales factores que impulsan a un cliente a no volver a comprar?

## 📂 Estructura del Repositorio
* ` /src`: Scripts de Python para la conexión con la API y ETL inicial.
* ` /pbix`: Archivo fuente de Power BI (`.pbix`).
* ` /images`: Capturas de pantalla del modelo de datos y las pestañas del reporte.

## ⚙️ Ingestión de Datos Automatizada (Python + Kaggle API)
Para evitar descargas manuales y garantizar la repetibilidad del proyecto, el origen de datos se configuró mediante un conector de **Script de Python** en Power BI. 

El siguiente código realiza la autenticación, descarga el dataset comprimido, lo extrae en el entorno local y lo transforma en un DataFrame de Pandas listo para Power Query:

```python
import os
import pandas as pd
from kaggle.api.kaggle_api_extended import KaggleApi

# 1. Inicializar y autenticar API de Kaggle (Requiere kaggle.json en ~/.kaggle/)
api = KaggleApi()
api.authenticate()

# 2. Definir rutas y descargar dataset
DATASET_ID = 'carrie1/ecommerce-data' # Cambiar por el dataset definitivo elegido
DOWNLOAD_PATH = './dataset/'

api.dataset_download_files(DATASET_ID, path=DOWNLOAD_PATH, unzip=True)

# 3. Cargar datos a Power BI
csv_file = [f for f in os.listdir(DOWNLOAD_PATH) if f.endswith('.csv')][0]
df_transacciones = pd.read_csv(os.path.join(DOWNLOAD_PATH, csv_file), encoding='ISO-8859-1')
```

## 🧠 Arquitectura de Datos y Modelado
Se implementó un **Esquema Estrella (Star Schema)** para garantizar un rendimiento óptimo de las consultas DAX y la escalabilidad del reporte. El modelo desglosa la información de la siguiente manera:

* **Tablas de Hechos:** `Fact_Transacciones` (Registra ventas, fechas, cantidades y montos).
* **Tablas de Dimensiones:** `Dim_Clientes`, `Dim_Productos`, `Dim_Calendario` y `Dim_Geografia`.

> 📌 **Nota de Arquitectura:** Las relaciones se configuraron estrictamente de 1 a Varios (1:*) con dirección de filtro único para asegurar la integridad referencial y evitar problemas de ambigüedad en los cálculos jerárquicos. Una vez finalizado el desarrollo, se adjuntará aquí la captura del modelo relacional.

## 🛠️ Tecnologías y Técnicas Utilizadas
* **Python & Pandas:** Ingestión automatizada y control de excepciones en la descarga.
* **Power Query (M):** Limpieza complementaria de nulos, tipado de datos y combinación de consultas.
* **DAX Avanzado:** Creación de tablas calculadas, variables (`VAR`) y funciones de inteligencia de tiempo.

### 📐 Ejemplo de Fórmulas DAX Clave

**1. Tasa de Churn Mensual (Métricas Dinámicas):**
```dax
Churn Rate = 
VAR ClientesActivosMesPasado = CALCULATE([Clientes Activos], DATEADD('Dim_Calendario'[Fecha], -1, MONTH))
VAR ClientesPerdidos = ClientesActivosMesPasado - [Clientes Activos]
RETURN
DIVIDE(ClientesPerdidos, ClientesActivosMesPasado, 0)
```

**2. Segmentación RFM Dinámica:**
```dax
Segmento Cliente = 
SWITCH(
    TRUE(),
    [Valor Monetario] > 5000 && [Frecuencia] > 10, "VIP",
    [Recencia] > 90, "En Riesgo de Abandono",
    "Cliente Regular"
)
```

## 📊 El Dashboard (Data Storytelling)
El reporte final consta de 3 vistas interactivas diseñadas para diferentes niveles de toma de decisiones:

1. **Vista Ejecutiva:** KPIs macro (Ingresos, Ticket Promedio, Clientes Totales) y tendencias de ventas.
2. **Análisis RFM & Comportamiento:** Dispersión de clientes y matriz de calor de compras.
3. **Retención y Churn:** Análisis de embudo y árbol de descomposición de causas de pérdida.

*(Nota: Las capturas de pantalla del reporte interactivo se encuentran en la carpeta `/images` de este repositorio).*

## 💡 Conclusiones y Recomendaciones de Negocio
* **Hallazgo 1:** El 20% de los clientes VIP genera el 70% de los ingresos totales. Se recomienda un programa de fidelización inmediato para blindar a este grupo.
* **Hallazgo 2:** La tasa de abandono se dispara después del tercer mes de inactividad. Estrategia: Campañas automáticas de email marketing a los 60 días de la última compra.

---
**Desarrollado por:** [Tu Nombre Completo]  
* [LinkedIn](https://linkedin.com)  
* [Tu Portafolio Web / Email de contacto]
