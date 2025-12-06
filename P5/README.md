# Blog_RobMovil

El objetivo de esta quinta práctica es implementar un sistema capaz de: 

1. Controlar la navegación reactiva del robot en un entorno utilizando datos del sensor láser.

2. Generar un mapa de ocupación 2D en tiempo real, donde cada celda del mundo mantiene una probabilidad de estar ocupada o libre.

3. Aplicar una aproximación basada en la regla de Bayes utilizando ratios para actualizar las celdas.

El robot, equipado con un sensor LIDAR de 360°, avanza por el escenario evitando obstáculos a la vez que “pinta” el mapa mediante la información de distancias detectadas.

## Estructura del código

### Navegación reactiva

La navegación está basada únicamente en el láser frontal:

- Si detecta obstáculos cerca en 0° o 45° gira a la izquierda.
- Si detecta obstáculo en 315° gira a la derecha.
- Si no detecta nada cerca avanza recto.

```python
if datos_laser[0] < 0.7 or datos_laser[45] < 0.7:
  # Gira a la izquierda
  HAL.setV(0)
  HAL.setW(-0.5)
```

### Mapa de ocupación

Se establecen los ratios Bayesianos:

- `ratio_obstaculo` incrementa la probabilidad de la ocupación
- `ratio_libre` incrementa la probabilidad de celda libre

El mapa no se actualiza en cada iteración, si no que se calcula la distancia que ha recorrido el robot desde la última vez que se actualizó el mapa y se actualiza si el robot se ha movido al menos 1.5 metros. Esto reduce el cómputo y evita marcar continuamente los mismo rayos.

```python
if distancia_recorrida >= 1.5:
```

Para construir el mapa se procesan los 360º del laser. Para cada rayo del laser:

1. Se convierte el ángulo del laser de grados a radianes
```python
ang_rad = math.radians(ang_grados)
```

2. Se determina si el rayo ha detectado un obstáculo. Si no, se extiende artificialmente el rayo hasta un rango máximo.

3. Se calcula el punto de impacto en coordenadas globales. Primero se calcula el punto relativo en el sistema local del robot. Después, se transforma a coordenadas globales usando la orientación del robot.
```python
mundo_x = pos_x + (math.cos(orientacion)*dx - math.sin(orientacion)*dy)
mundo_y = pos_y + (math.sin(orientacion)*dx + math.cos(orientacion)*dy)
```

4. Se transforman las coordenadas del mundo real al mapa de ocupación.
```python
celda_x = int((origen_x + mundo_x) * ESCALA_CELDA)
celda_y = int((origen_y - mundo_y) * ESCALA_CELDA)
```

5. Se dibuja el rayo marcando las celdas entre la posición del robot y el punto final del rayo.

6. Si el flag `hay_obstáculo` es True, se incrementa la probabilidad de ocupación y se actualiza la celda final del rayo.
```python
if hay_obstaculo:
  mapa_ocupacion[celda_y, celda_x] = min(1, mapa_ocupacion[celda_y, celda_x] * ratio_obstaculo)
```

## Problema

Hay que matizar un problema que no he conseguido solucionar. El problema se trata de que en la visualización del mapa, me aparece el mapa rotado 90 grados. He intentado solucionarlo modificando los ejes e intercambiando filas por columnas en varias partes del script, pero no he conseguido arreglarlo. Se puede observar este problema en el vídeo de muestra.

## Vídeo

[Vídeo del funcionamiento](https://drive.google.com/file/d/1Yl9-i8aPvBN6NKl1hL-U8tDwOE6A1smr/view?usp=sharing)











