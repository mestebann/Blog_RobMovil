# Blog_RobMovil

## Introducción

El objetivo principal de esta práctica es dotar a un robot móvil de la capacidad de localizarse en un entorno conocido utilizando visión artificial, concretamente mediante la detección de balizas AprilTag, y combinar esta información con odometría para mantener una estimación continua de su pose. 

Se implementa también un comportamiento básico de navegación reactiva, permitiendo al robot desplazarse por el entorno evitando obstáculos.

La técnica empleada para la localización visual se basa en:

1. La detección de AprilTags en la imagen de la cámara.
2. La estimación de la pose relativa entre la cámara y la baliza mediante Perspective-n-Point.
3. El uso de transformaciones homogéneas para expresar finalmente la pose del robot en el sistema de referencia global (mapa).

Cuando no se detectan balizas, la localización se mantiene mediante odometría incremental, integrando los desplazamientos medidos por el robot. De esta forma, se obtiene un sistema de localización híbrido (visual + odométrico) robusto frente a pérdidas temporales de información visual.


## Estructura del código

### Inicialización y configuración

Se define la geometría de las balizas, especificando el tamaño real del AprilTag y los puntos 3D de sus esquinas en el sistema de referencia del tag. Estos puntos son fundamentales para el cálculo de la pose con PnP.

Finalmente, se carga el archivo YAML que contiene las poses conocidas de las balizas en el mundo, lo que permite relacionar la información visual con el mapa global.

### Transformaciones y utilidades matemáticas

Se implementa una función `rpy_a_rotacion` que genera una matriz de rotación a partir de ángulos roll, pitch y yaw. Esta función se utiliza tanto para definir la orientación de las balizas en el mundo como para describir la orientación de la cámara respecto al robot.

Posteriormente, se define la transformación entre la cámara y el robot, utilizando los valores obtenidos del modelo del robot. Esta transformación tiene en cuenta tanto la traslación como el pitch de la cámara.

Además, se introduce una corrección del sistema de referencia de OpenCV al sistema del robot, ya que OpenCV utiliza un eje Z hacia delante y un sistema de coordenadas diferente al habitual en robótica móvil.

### Localización visual mediante AprilTags

En cada iteración del bucle principal:

1. Se captura una imagen de la cámara y se convierte a escala de grises
2. Se detectan las balizas visibles en la imagen
3. Para cada baliza válida:
   - Se calculan las correspondencias entre puntos 3D del tag y puntos 2D de la imagen.
   - Se estima la pose relativa cámara-baliza con `cv2.solvePnP`
   - Se construye la transformación homogénea correspondiente
   - Se encadenan las transformaciones:
       - Mundo -> Tag
       - Tag -> Cámara
       - Cámara -> Robot
         obteniendo finalmente la pose del robot en el mundo

La orientación del robot se extrae a partir de la matriz de rotación final, y se aplica una corrección de 90 grados (π/2 radianes) para alinear correctamente el eje del robot con el sistema de referencia del mundo.

### Odometría cuando no se ven balizas

Cuando no se detectan AprilTags y se dispone de una última pose visual válida, el sistema pasa a integrar la odometría incremental:

- Se calculan los incrementos de posición y orientación
- Estos incrementos se transforman al sistema global utilizando el yaw estimado del robot.
- La pose se actualiza y se sigue mostrando en la interfaz.

De esta forma, el robot mantiene una estimación coherente de su localización incluso en ausencia de información visual.

### Navegación reactiva

Se implementa una máquina de estados simple para la navegación:

- AVANZAR: el robot se mueve hacia delante mientras no haya obstáculos.
- GIRAR: si se detecta un obstáculo frontal, el robot gira sobre sí mismo.
- AJUSTE: cuando no se detectan balizas, el robot realiza pequeños giros con tiempo aleatorio para no realizar siempre el mismo giro.

El uso del láser permite detectar obstáculos frontales y cambiar de estado en función de la distancia medida.


## Problemas y soluciones

### Desalineación de la orientación del robot

La estimación de la orientación del robot aparecía girada aproximadamente 90 grados respecto a la real.

El problema se debía a una diferencia entre el eje X del robot y el eje utilizado al extraer el yaw de la matriz de transformación. Se corrigió añadiendo un desplazamiento fijo de π/2 radianes al yaw estimado, alineando correctamente los sistemas de referencia.

### Incompatibilidad entre sistemas de coordenadas

La pose calculada a partir de `solvePnP` no coincidía con la esperada en el sistema del robot.

Se introdujo una transformación de corrección entre el sistema de referencia de OpenCV y el sistema del robot, asegurando que los ejes X, Y y Z coincidieran correctamente antes de encadenar transformaciones.

### Pérdida de localización al no ver balizas

Cuando el robot dejaba de ver balizas, la estimación de la pose se perdía o se volvía errática.

Se integró la odometría únicamente cuando no hay detecciones visuales, usando como referencia la última pose visual válida. Esto permitió mantener una estimación continua y estable.

### Dificultad para volver a detectar balizas

En ocasiones el robot avanzaba sin volver a encontrar balizas durante un tiempo prolongado.

Se añadió el estado AJUSTE, en el que el robot realiza pequeños giros aleatorios para modificar su campo de visión y aumentar la probabilidad de detectar nuevamente una baliza.


## Vídeo

[Vídeo del funcionamiento](https://drive.google.com/file/d/1EYHkRAx2g4_EQMCxcjYMXLXKKmcDOHbs/view?usp=sharing)
