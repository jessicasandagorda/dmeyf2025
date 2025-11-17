# Workflow general

1. **Preparación de datos**  
   Se unifican todos los meses disponibles, se limpian variables y se generan _features_ básicas (lags, ratios, etc.).

2. **Target y particionado temporal**  
   Se define `clase_ternaria` (`CONTINUA`, `BAJA+1`, `BAJA+2`).  
   Para entrenar se crea `clase01` (`BAJA+1` y `BAJA+2` = 1, `CONTINUA` = 0).  
   Se fijan meses de *training* y se reserva un mes tipo “future” para **backtesting** (202106) y otro para **scoring final** (202108).

3. **Feature engineering, canaritos y calidad de datos**  

   - **Control de data drifting**: transformación de ranking con cero fijo para estabilizar la distribución de las variables numéricas entre meses.  
   - **Limpieza de variables inútiles**: todas las variables con 100% de ceros se reemplazan por `NA` (o se eliminan del set de entrenamiento) para evitar ruido en el modelo.  
   - Se agregan variables `canarito_*` con ruido para poder detectar overfitting.  
   - Se aplica **undersampling** sobre la clase mayoritaria para balancear el entrenamiento.

4. **Entrenamiento de “modelitos” (ensemble)**  
   Se entrenan múltiples modelos LightGBM con distintas semillas y meta_modelos;  
   cada modelo se guarda en disco como `./modelitos/mod_*.txt`.

5. **Backtesting en 202106**  
   Se aplica el ensemble (promedio por seeds y luego por meta_modelos) sobre 202106.  
   Se ordenan los clientes por _score_ y, para varios cortes (N envíos), se calcula la ganancia usando la función de negocio:  
   –20.000 para no `BAJA+2`, +780.000 para `BAJA+2`.  
   Se elige `corte_mejor` como el N que maximiza la ganancia en 202106.

6. **Scoring final en 202108 y submit**  
   Se aplica el mismo ensemble sobre 202108, se ordena por _score_ y se marcan como `Predicted = 1` los primeros `corte_mejor` clientes.  
   Se genera el CSV final con `numero_de_cliente` y `Predicted` para enviar a Dos Esfinges (Zulip).

7. **Limitación importante**  
   **LAMENTABLEMENTE y EN CONTRA DE MI VOLUNTAD no pude optimizar los hiperparámetros ni la configuración de los “modelitos” de la mejor forma posible** luego de incorporar los canaritos y de fijar el valor final del undersampling, por lo que el modelo y el ensemble son funcionales pero no necesariamente óptimos.

Link a la carpeta del experimento con OB y modelitos: 
https://drive.google.com/drive/folders/16Tkm8OKcm4fzrExMENi1Asl7Mp4XFSKQ?usp=drive_link
   
