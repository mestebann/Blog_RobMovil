# Blog_RobMovil

En esta práctica, se pide un sistema que consiste en un coche que esquive
obstáculos a medida que avanza mediante técnicas de navegación local.

La técnica de navegación local que he empleado ha sido la VFF, que consiste en
generar tres fuerzas virtuales: vector repulsivo, vector atractivo y vector
resultante.

Una característica importante de mi sistema, es que únicamente se calcula la 
velocidad angular del coche, manteniéndose así la velocidad lineal constante.

## Vector repulsivo

Este vector consiste en calcular la fuerza que generan los diferentes obstáculos
en nuestro coche, así, cuanto más cerca estén los obstáculos, mayor será la 
magnitud del vector.

```python
for dist, angle in laser_polar:
  if 0.01 < dist < influencia:
    mag = k * ((1.0/dist) - (1.0/influencia))**n
    Fx_total += -mag * math.cos(angle)
    Fy_total += -mag * math.sin(angle)
```

Para calcular este vector, se va iterando sobre los datos del láser y se calculan,
primero la magnitud y luego las dos componentes x e y del vector. 

Después, se calcula finalmente el vector con las dos componentes multiplicadas por
la ponderación que depende de la distancia mínima del láser.

```python
vector_repulsion = beta * (np.array([Fx_total, Fy_total]))
```

## Vector atractivo

Para calcular este vector, se obtiene primero la orientación del coche y para obtener
las componentes del vector, se pasan las coordenadas al marco del robot. 
Finalmente, se multiplica la ponderación por ambas componentes, esta ponderación
cuando no haya obstáculos cerca.

```python
robot_theta = HAL.getPose3d().yaw
atractivo_x, atractivo_y = absolute2relative(obj_x, obj_y, coord_x, coord_y, robot_theta)
vector_atractivo = alpha * (np.array([atractivo_x, atractivo_y]))
```

## Vector resultante

Para el cálculo de este vector, se tienen en cuenta ambas componentes de los vectores
anteriores y se suman, generándose así el vector resultante.

```python
resultante_x = vector_repulsion[0] + vector_atractivo[0]
resultante_y = vector_repulsion[1] + vector_atractivo[1]
vector_resultante = np.array([resultante_x, resultante_y])
```

## Problemas y soluciones

- VECTOR REPULSIVO: este ha sido el principal problema que me ha surgido, pues
  la magnitud de este vector me salía demasiado grande, hasta que conseguí
  disminuirla o agrandarla exponencialmente según la influencia de los obstáculos.
  Esto se puede ver en el cálculo del vector repulsivo anteriormente.

- PONDERACIONES: otro problema ha sido el ajustar las ponderaciones de los vectores
  repulsivo y atractivo, pues cuanto más bajo sea el valor mínimo percibido por el
  láser, mayor será beta (ponderación vector repulsivo) y viceversa. 

  ```python
  if dist_min < 0.5:  # Obstáculo muy cercano
        alpha = 0.3
        beta = 1.2
    elif dist_min < 1.0:
        alpha = 0.6
        beta = 0.8
    else:
        alpha = 1.0
        beta = 0.3
  ```

- GIRO: este problema ha surgido tras varias pruebas que el coche me
  oscilaba demasiado tras alcanzar un objetivo. La solución a esto ha sido
  calcular el error angular del vector resultante y después limitar el giro para
  que sea lo más suave posible.

  ```python
  error_angular = math.atan2(resultante_y, resultante_x)
  error_angular = math.atan2(math.sin(error_angular), math.cos(error_angular))
 
  k_ang = 2.0
  vel_ang = k_ang * error_angular
  vel_ang = max(-2, min(1.5, vel_ang))
  ```
  
## Vídeo

A continuación se muestra un vídeo del funcionamiento del sistema. He decidido
darle más importancia a la seguridad antes que a la velocidad. Es por esto que
a velocidad normal el coche tarda bastante en realizar la vuelta, por ello se
adjuntan dos vídeos, uno a velocidad normal y otro en x2.

[Vídeo a velocidad normal](https://drive.google.com/file/d/1lr8MIqNvYdt5kKYLx6r90WG7W-ZuGHPN/view?usp=sharing)
[Vídeo en velocidad aumentada](https://drive.google.com/file/d/1cKvBo7g64gEa7oU3NuuJ1zBAeHETAa2g/view?usp=sharing)


