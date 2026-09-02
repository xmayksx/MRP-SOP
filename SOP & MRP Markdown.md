# MANUAL OPERATIVO Y TÉCNICO: SIMULADOR S&OP Y PLANIFICACIÓN MRP

## 1. DESCRIPCIÓN GENERAL
La aplicación es una plataforma integral desarrollada en Streamlit para el control de inventarios, pronóstico predictivo de demanda con Machine Learning y algoritmos de series temporales, auditoría logística de tránsitos (OTIF) y simulación dinámica MRP bajo un modelo de revisión continua (Modelo Q).

---

## 2. REQUISITOS Y DEPENDENCIAS DEL ENTORNO
Para ejecutar la aplicación se requiere un entorno de Python 3.9 o superior con las siguientes librerías instaladas:

- streamlit
- pandas
- numpy
- statsmodels
- scikit-learn
- xgboost
- scipy
- plotly
- openpyxl (para lectura/escritura de archivos Excel)

Comando de instalación:
pip install streamlit pandas numpy statsmodels scikit-learn xgboost scipy plotly openpyxl

---

## 3. ESTRUCTURA DE ARCHIVOS Y RUTAS LOCALES
El sistema opera sobre rutas locales configuradas en el diccionario `RUTAS`. Debe asegurarse de que las carpetas existan y los archivos mantengan los formatos requeridos:

1. CARPETA VENTAS (`RUTAS["VENTAS"]`):
   - Contiene archivos Excel con el histórico de ventas.
   - Pestaña requerida: "BD"
   - Columnas requeridas: 'Item', 'Fecha', 'Valor', opcionalmente 'Categoria' y 'Descripción'.

2. CARPETA COMERCIAL (`RUTAS["COMERCIAL"]`):
   - Archivos Excel con pronósticos comerciales colaborativos.
   - Pestaña requerida: "BD"
   - Columnas: 'Item', 'Fecha', 'Valor'.

3. CARPETA/ARCHIVO TRÁNSITO (`RUTAS["TRANSITO"]` / `RUTAS["TRANSITO_DIRECTA"]`):
   - Seguimiento de embarques y contenedores en curso.
   - Columnas detectadas dinámicamente: Código/Material, Fecha Arribo/ETA, Cantidad/Peso, Estatus, Unidad/Medida.

4. MATRIZ DE PROVEEDORES (`RUTAS["PROVEEDORES"]`):
   - Cruce dimensional de compras.
   - Columnas detectadas: 'CÓDIGO SAP', 'PROVEEDOR', 'COD_PROV' (PN o PE), 'LEAD TIME'.

5. AUDITORÍA OTIF (`RUTAS["REGISTRO_OTIF"]`):
   - Archivo generado y actualizado por la app para persistir confirmaciones manuales y recepciones de planta.

---

## 4. REGLAS DE NEGOCIO Y MODELADO MATEMÁTICO

### A. Unidades y Factores de Conversión:
- SKUs '05*': Unidad base KG (premezclas con pesos fijos por bolsa según tabla interna).
- SKUs '06*': Unidad base Bolsas (detección regex de capacidad en KG desde la descripción).
- SKUs '18*': Unidades físicas independientes.
- Conversión a Quintales (QQ): KG / 45.3592.

### B. Consumo Simulado S&OP:
Para amortiguar quiebres sin sobredimensionar stock, la demanda base se calcula considerando las últimas 6 semanas:
Consumo Simulado = (Max_1 + Max_2 + Promedio_4W) / 3
Donde Max_1 y Max_2 representan los dos picos de consumo más altos de la ventana.

### C. Limpieza de Outliers y Modelos Predictivos:
- Detección de atípicos con Rango Intercuartílico Dinámico (IQR móvil de 12 semanas).
- Modelos entrenados por SKU:
  * Suavizamiento Exponencial Simple (SES)
  * Holt-Winters Aditivo con estacionalidad
  * Promedio Móvil 4S Optimizado (vía SLSQP en scipy.optimize)
  * Random Forest Regressor (Lags 1-6, 52, estacionales y tendencia)
  * XGBoost Regressor (Gradiente de árboles con hiperparámetros ajustados)

### D. Parámetros MRP (Modelo Q):
- Stock de Seguridad (SS):
  SS = 1.645 * Desviación_6W * sqrt((Lead_Time + 15) / 7)
- Punto de Reorden (ROP):
  ROP = (Demanda_Diaria * Lead_Time_Total) + SS
- Lote Sugerido de Compra:
  * Proveedores PE (Extranjeros / Importación): 13 semanas de consumo.
  * Proveedores PN (Nacionales): 2 semanas de consumo.

---

## 5. GUÍA DE USO DE LOS MÓDULOS (PESTAÑAS)

### Sidebar (Configuración Inicial):
1. Seleccione las categorías o SKUs a procesar.
2. Defina la fecha de corte de entrenamiento.
3. Cargue el archivo Excel del inventario actual (columnas requeridas: CODIGO, CANTIDAD, COSTO/MONTO).
4. Presione "Ejecutar Diagnóstico".

### Pestaña 1: Dashboard Ejecutivo Global
- Vista de KPIs principales: Inversión en C$, inventario físico (KG/QQ/Bolsas), días de cobertura y próximos ingresos.
- Gráficas de Pareto de valor inmovilizado y distribución por categoría.
- Tabla analítica con alertas automáticas de caída/aumento de demanda (+/- 15%) y materiales con riesgo por consumo pico menor a 20 días.

### Pestaña 2: OTIF - Seguimiento de Tránsitos
- Sub-pestaña "Dashboard de Tránsitos": Métricas de cumplimiento logístico, volumen mensual y arribos pendientes por proveedor.
- Sub-pestaña "Confirmación y Corrección Manual": Interfaz editable para dar ingreso real a bodega a pedidos de tránsito o revertir ingresos erróneos. Guarda automáticamente en el archivo Excel de auditoría.

### Pestaña 3: Simulador S&OP y Contenedores (What-If)
- Permite programar de 1 a 5 contenedores marítimos/terrestres con fechas ETA específicas.
- Cálculo de cubicación en tiempo real comparado con el estándar de 20,000 KG por contenedor.
- Simulación de la curva de stock resultante con las compras inyectadas.

### Pestaña 4: Detalle MRP por SKU
- Desglose semanal de balance de masas: Inventario Inicial + Tránsito Real + Simulación Sugerida - Consumo = Inventario Final.
- Gráfico interactivo con semáforo de inventario sano, consumo de SS o quiebre proyectado.
- Alerta visual roja cuando se detecta un quiebre en menos de 15 días.
- Capacidad de sobreescribir proyecciones manuales mes a mes y aplicar división de pedidos (Split de entregas).

### Pestaña 5: Parámetros de Reorden (ROP/Q)
- Visualización consolidada de Lead Times, gracia logística (15 días), SS, ROP y Lote sugerido para todos los SKUs procesados.

### Pestaña 6: Auditoría de Modelos
- Backtesting exhaustivo por SKU comparando MAPE (%), RMSE y porcentaje de Exactitud sobre la data real no vista.