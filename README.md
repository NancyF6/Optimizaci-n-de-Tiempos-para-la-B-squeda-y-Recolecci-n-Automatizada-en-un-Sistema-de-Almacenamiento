# Optimizaci-n-de-Tiempos-para-la-B-squeda-y-Recolecci-n-Automatizada-en-un-Sistema-de-Almacenamiento
Prototipo mecatrónico que identifica, localiza y transporta cajas de un anaquel de nueve posiciones. Una cámara con YOLO26 reconoce la figura de cada caja y un ESP32 guía un brazo robótico de 4 GDL que la deposita en una banda transportadora con sensores infrarrojos.
# Recolector Automático de Componentes con Visión Artificial (YOLO26)

Prototipo mecatrónico que identifica, localiza y transporta cajas con componentes eléctricos almacenadas en un anaquel de nueve posiciones. Una cámara fija y una red neuronal YOLO26 reconocen la figura geométrica de cada caja; un ESP32 mueve un brazo robótico de 4 grados de libertad que toma la caja y la deposita en una banda transportadora controlada con sensores infrarrojos.

---

## 1. Problema
En el almacenamiento y manejo de componentes eléctricos, buscar y recolectar piezas de forma manual consume tiempo y puede producir errores de selección. Este proyecto resuelve el problema a pequeña escala: el usuario elige un componente en una interfaz y el sistema lo localiza y lo recolecta de forma autónoma, aunque las cajas cambien de lugar entre solicitudes.

---

## 2. Resultados Principales
Se solicitó la recolección de la caja de cada una de las 9 posiciones, repitiendo cada una 3 veces (27 intentos en total).

| Meta | Objetivo | Resultado | ¿Cumple? |
| :--- | :---: | :---: | :---: |
| **Tiempo máximo de recolección** | $\le 30\text{ s}$ | $25.3\text{ s}$ (máx. por posición) | **Sí** |
| **Precisión de detección** | $\ge 90\,\%$ | $92.6\,\% \ (25/27)$ | **Sí** |
| **Agarres exitosos** | $\ge 90\,\%$ | $85.2\,\% \ (23/27)$ | **No** |

* **Tiempo promedio general:** $22.9\text{ s}$ desde que el usuario selecciona el componente hasta que la caja queda sobre la banda.

### Desglose por Posición

| Posición | Figura | Intentos | Detecciones | Agarres | Tiempo Prom. (s) |
| :---: | :--- | :---: | :---: | :---: | :---: |
| **F1-C1** | Círculo | 3 | 3 | 3 | 22.4 |
| **F1-C2** | Estrella | 3 | 3 | 2 | 24.1 |
| **F1-C3** | Estrella de 8 picos | 3 | 2 | 2 | 25.3 |
| **F2-C1** | Trapecio | 3 | 3 | 3 | 21.8 |
| **F2-C2** | Triángulo | 3 | 3 | 3 | 22.9 |
| **F2-C3** | Pentágono | 3 | 3 | 2 | 24.6 |
| **F3-C1** | Óvalo | 3 | 3 | 3 | 20.7 |
| **F3-C2** | Octágono | 3 | 2 | 2 | 23.5 |
| **F3-C3** | Cuadrado | 3 | 3 | 3 | 21.2 |
| **Total** | — | **27** | **25** | **23** | **22.9 (Promedio)** |

* **Análisis:** Las dos fallas de detección se concentraron en la estrella de ocho picos y el octágono, formas visualmente parecidas a otras clases (hipótesis pendiente de verificación formal). Los agarres fallidos con detección correcta apuntan a la holgura mecánica de los servomotores SG90 al alinear la pinza con el asa.

---

## 3. ¿Cómo Funciona?
El sistema cierra un ciclo **percepción $\rightarrow$ decisión $\rightarrow$ acción** mediante dos subsistemas sensoriales:

1. **Percepción visual:** Una cámara web USB fija observa la cara trasera de las nueve cajas a través de aberturas en forma de rombo. Cada cuadro se procesa con **YOLO26**, devolviendo clase, recuadro delimitador (*bounding box*) y confianza.
2. **Decisión:** Se filtran las detecciones con confianza $< 0.5$ y cada objeto se asigna a una celda ($3 \times 3$) del anaquel. Esto genera un mapa dinámico `{clase $\rightarrow$ posición}`. Al seleccionar un componente, se consulta una tabla de ángulos calibrada para el brazo.
3. **Acción (Brazo):** Los ángulos se envían por puerto serial ($115200\text{ bauds}$) al ESP32 para generar las señales PWM de los 5 servos SG90. El brazo se aproxima, sujeta el asa, retira la caja y la deposita en la banda.
4. **Banda transportadora:** Su lógica corre de forma local en el ESP32 mediante dos sensores infrarrojos HW-201 (el sensor de recepción enciende el motor y el sensor del extremo final lo detiene).

> *Nota:* Si la figura solicitada no aparece en la escena, el sistema emite el aviso *"componente no encontrado"* y omite el movimiento del brazo.
