# 📡 Análisis de Clientes ConnectaTel

> Proyecto de análisis de datos para una empresa de telecomunicaciones en Latinoamérica, con el objetivo de comprender el comportamiento de sus clientes, detectar anomalías y construir segmentos accionables para el negocio.

---

## 🎯 Objetivo del Proyecto

Evaluar el comportamiento de los clientes de **ConnectaTel** a partir de datos registrados hasta el año 2024. El análisis busca:

- Construir un **perfil estadístico** de los clientes.
- Detectar **comportamientos atípicos** (outliers / heavy users).
- Crear **segmentos de clientes** por nivel de uso y grupo de edad.
- Identificar patrones de consumo que permitan **diseñar estrategias de retención** y **mejorar los planes** ofrecidos.

---

## 📂 Datasets Utilizados

El proyecto trabaja con tres archivos CSV ubicados en la ruta `/datasets/`:

| Archivo | Descripción |
|---|---|
| `plans.csv` | Información de los planes actuales: precio, minutos incluidos, GB incluidos y costo por excedente. |
| `users_latam.csv` | Información de los clientes: edad, ciudad, fecha de registro, plan contratado y estado de churn. |
| `usage.csv` | Detalle del uso real de los servicios por cliente: llamadas y mensajes (tipo, duración y fecha). |

---

## 🔬 Etapas del Análisis

### Paso 1 — Carga y exploración inicial
- Importación de librerías (`pandas`, `numpy`, `matplotlib`, `seaborn`).
- Carga de los tres datasets y visualización de primeras filas con `.head()`.
- Revisión de dimensiones con `.shape` e inspección de tipos de datos con `.info()`.

### Paso 2 — Identificación de problemas de calidad de datos
- **Valores nulos:** detección de columnas con nulos y cálculo de su proporción.
  - `churn_date`: ~88.3% nulos (esperado, clientes activos).
  - `duration` / `length`: ~55% y ~45% nulos (nulos estructurales por tipo de evento).
  - `city`: ~11.7% nulos.
  - `date`: ~0.12% nulos.
- **Valores inválidos (sentinels):** detección de `age = -999` y `city = "?"`.
- **Revisión de fechas:** identificación de 40 registros con año de registro en 2026 (fechas futuras imposibles).

### Paso 3 — Limpieza de datos
- Reemplazo del sentinel `-999` en `age` por la **mediana** (calculada excluyendo el valor inválido).
- Reemplazo del sentinel `"?"` en `city` por `pd.NA`.
- Marcado de fechas futuras (año > 2024) en `reg_date` como `pd.NaT`.
- Confirmación de nulos **MAR** (*Missing At Random*) en `duration` y `length`: 100% de ausencias condicionadas por el tipo de evento (`call` vs `text`), conservados para rellenarse con `0` en la etapa de agrupación.

### Paso 4 — Resumen estadístico de uso por usuario
- Creación de columnas auxiliares `is_text` e `is_call`.
- Agrupación de `usage` por `user_id` para calcular:
  - `cant_mensajes` — total de mensajes enviados.
  - `cant_llamadas` — total de llamadas realizadas.
  - `cant_minutos_llamada` — total de minutos de llamada.
- Merge con `users` para construir la tabla maestra `user_profile`.
- Resumen estadístico con `.describe()` y distribución porcentual del plan.

### Paso 5 — Visualización de distribuciones y detección de outliers
- **Histogramas** (con `hue='plan'`) para `age`, `cant_mensajes`, `cant_llamadas` y `cant_minutos_llamada`.
  - Distribuciones con claro **sesgo positivo (right-skewed)** en variables de uso.
  - Distribución de edades amplia y uniforme sin pico generacional dominante.
- **Boxplots** por plan para identificar outliers visualmente.
- Cálculo de **límites IQR** para `cant_mensajes`, `cant_llamadas` y `cant_minutos_llamada`.
- Decisión: **mantener todos los outliers** — representan *heavy users* reales, válidos para análisis de negocio.

### Paso 6 — Segmentación de clientes
- **Segmentación por uso** (`grupo_uso`):
  - `Bajo uso`: llamadas < 5 y mensajes < 5.
  - `Uso medio`: llamadas < 10 y mensajes < 10.
  - `Alto uso`: resto de casos.
- **Segmentación por edad** (`grupo_edad`):
  - `Joven`: age < 30.
  - `Adulto`: age < 60.
  - `Adulto Mayor`: age ≥ 60.
- Visualización de ambas segmentaciones con `countplot` diferenciado por plan.

### Paso 7 — Insight ejecutivo para stakeholders
Conclusiones clave traducidas al lenguaje de negocio:
- ConnectaTel tiene una base de clientes **diversa en edad**, sin dependencia generacional.
- El consumo sigue una **distribución asimétrica**: la mayoría usa poco, pero los *heavy users* son críticos para el ARPU.
- Los outliers **no son errores**; representan un segmento premium sin plan adecuado.
- **Recomendaciones accionables:**
  - Campaña de upsell hacia clientes de "Alto uso" en plan Básico.
  - Diseño de un nuevo plan **VIP/Ilimitado** para absorber la sobredemanda.
  - Reporte al área de Arquitectura de Datos para corregir la captura de edades `-999` y fechas futuras `2026`.

---

## ⚙️ Requisitos

```bash
pandas
numpy
matplotlib
seaborn
```

Instala las dependencias con:

```bash
pip install pandas numpy matplotlib seaborn
```

---

## 🚀 Cómo Ejecutar el Notebook

### Opción A — Desde GitHub (recomendada)

1. Clona este repositorio:
   ```bash
   git clone https://github.com/TU_USUARIO/TU_REPOSITORIO.git
   cd TU_REPOSITORIO
   ```
2. Abre el notebook en Jupyter:
   ```bash
   jupyter notebook S7_Version-Estudiante-Project-ConnectaTel.ipynb
   ```

### Opción B — Ejecutar en Jupyter desde el repositorio

1. Descarga el `.ipynb` directamente desde GitHub (botón **Download raw file**).
2. Abre Jupyter Notebook o JupyterLab en tu máquina local.
3. Navega hasta la carpeta del archivo descargado y ábrelo.

---

## 🗂️ Guía de Reproducción

1. **Clona o descarga** el repositorio.
2. **Coloca los datasets** en una carpeta llamada `datasets/` en la raíz del proyecto:
   ```
   /datasets/plans.csv
   /datasets/users_latam.csv
   /datasets/usage.csv
   ```
3. **Ejecuta las celdas en orden** desde el Paso 1 hasta el Paso 7. Cada paso depende del anterior.
4. Los **comentarios e insights** dentro del notebook explican las decisiones tomadas en cada etapa.
5. La tabla maestra final se llama `user_profile` e incluye el perfil completo de cada cliente con sus métricas de uso y segmentos asignados.

---

## 📁 Estructura del Repositorio

```
📦 TU_REPOSITORIO
 ┣ 📂 datasets/
 ┃ ┣ plans.csv
 ┃ ┣ users_latam.csv
 ┃ ┗ usage.csv
 ┣ 📂 datasets_clean/
 ┃ ┣ plans_clean.csv
 ┃ ┣ users_latam_clean.csv
 ┃ ┗ usage_clean.csv
 ┣ 📓 S7-Project-ConnectaTel.ipynb
 ┗ 📄 README.md

