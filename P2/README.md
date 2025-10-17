# Blog_RobMovil

En esta segunda práctica, se propone hacer funcionar un coche de carreras que realiza una vuelta a un circuito en el menor tiempo posible siguiendo una línea. Para ello, se captura la imagen de la línea y se implementan dos controladores PID, uno para el giro y otro para la velocidad lineal.

## Capturar imagen

- ```python
  image = HAL.getImage()
  ```
  Se obtiene la imagen proporcionada por el coche.

- ```python
  hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
  ```
  Convertimos la imagen desde el formato habitual BGR al espacio de color HSV. Usar HSV en lugar de BGR hace     que la detección de color sea más robusta frente a cambios de luz y sombras.

- ```python
  r1_inf = np.array([0, 100, 100])
  ```
  Hacemos la selección de rangos del color rojo.

- ```python
  mask_r1 = cv2.inRange(hsv, r1_inf, r1_sup)
  mask = cv2.bitwise_or(mask_r1, mask_r2)
  ```
  Se generan primero dos máscaras binarias de blanco y negro que marcan las zonas del primer y segundo rango     de rojo y después se combinan ambas máscaras.

- ```python
  M = cv2.moments(mask)
  ```
  Una vez que se tiene la máscara binaria, se calculan los momentos geométricos de la imagen.

- ```python
  if M["m00"] > 0:
    cx = int(M["m10"] / M["m00"])
    cy = int(M["m01"] / M["m00"])

    err_w = (anchura / 2) - cx
    ```
  La condición de `M["m00"] > 0` quiere decir que ha detectado la línea, por lo que seguidamente se calcula la   posición horizontal dentro de la línea (cx) y se calcula también el error para realizar el giro más            adelante.

## Controladores PID

Ahora, vamos a analizar ambos controladores, los cuales son diferentes, uno es un PID para el giro y otro es un PD para la velocidad lineal.

Para cada controlador se definen sus constantes con unos valores determinados, los cuales se mostrarán a continuación.

Se definen también unas variables que son los errores previos (necesarios para el término derivativo) y la suma de errores (usados por el término integral

- PID (W):

En cuanto al controlador PID del giro, se utilizan los siguientes valores para las constantes: 
`Kp_w = 0.006
Ki_w = 0.000007 
Kd_w = 0.0122`  
Cada una de estas constantes tiene su función para que el coche siga la línea, Kp_w controla la reacción inmediata al error, Ki_w corrige errores acumulados en el tiempo y Kd_w suaviza el giro para evitar oscilaciones.

`w = (Kp_w * err_w) + (Ki_w * sum_err_w) + (Kd_w * diff_w)` para calcular la w se utiliza la fórmula, en la que se multiplica la constante proporcional por el error calculado de la línea, se multiplica la constante integral por el error acumulado y la constante derivativa por la diferencia del error actual con el anterior, y todas ellas sumadas.


- PD (v):

La idea de este controlador es simple, si el coche está bien centrado acelera y si no, frena. Los valores de las constantes de este controlador son:
`Kp_v = 0.016
Kd_v = 0.008`

`ajuste_v = (Kp_v * err_v) + (Kd_v * diff_v)` se calcula un ajuste de la velocidad para más adelante calcularla, este ajuste se calcula con la fórmula en la que la constante proporcional se multiplica por el error de v que es tomado por el valor absoluto del error de w, y se suma a la constante derivativa multiplicada por la diferencia del error actual y el anterior.

Ahora se calcula la v final en la que si el error es pequeño, se toma la velocidad máxima establecida, y si no, se le resta el ajuste calculado antes y se toma la velocidad dentro de un rango protegiendo así al coche de oscilaciones.
```python
if err_v < 4:
    v = Vmax
else:
    v = Vmax - ajuste_v

v = max(Vmin, min(Vmax, v))
```

## Problemas y soluciones

- VALORES:
  El principal problema que he tenido ha sido el encontrar los valores adecuados de las constantes de los PID, ya que, por ejemplo, si ponía una Kp_w demasiado alta, el coche giraba demasiado en un sentido perdiendo así la línea por el otro lado, generando oscilaciones. Además, todas estas constantes debían estar en concordancia con los valores de la velocidad (Vmax), ya que si ponía un valor alto, el coche podía perder la curva al no darle tiempo a reducir velocidad y chocarse en el exterior.

- IMAGEN:
  Otro problema que me ha surgido fue al inicio, a la hora de saber cómo procesar la imagen obtenida, ya que no estaba muy familiarizado con el procesamiento de imágenes mediante HSV.

- SI NO VE LA LÍNEA:
  Otro problema ha sido el programar al coche si no ve la línea, lo que he hecho ha sido ir guardando en cada iteración el signo del último giro del coche y luego dentro del else, que gire hacia ese lado y si no hay un giro previo, hacia la derecha por defecto. Por lo tanto, es como si el coche tuviese "memoria" de hacia donde está la línea por el último giro realizado.
  ```python
  ultimo_giro = np.sign(w) if abs(w) > 0.001 else ultimo_giro

  else:
      HAL.setV(2.0)  #avance lento
      HAL.setW(0.4 * ultimo_giro if ultimo_giro != 0 else 0.4) 

## Vídeo
El video se puede encontrar en el siguiente enlace: [video del funcionamiento]https://drive.google.com/file/d/1SJken-aVUpuuLflx7KAfTMk7PYbxxlQP/view?usp=sharing

Hay que matizar que dependiendo de cuánto de sobrecargada esté la CPU irá mejor o peor el coche, en cas de que esté sobrecargada, la frecuencia con la que se ejecuta el código es menor y por tanto el coche no responderá de igual manera. 
