# Blog_RobMovil

En esta cuarta práctica se pide programar un coche autónomo desarrolland un sistema de navegación global. Para ello, he usado:

  - Campo de costes desde el objetivo
  - Penalización por proximidad a obstáculos
  - Selección de sub-objetivo

El resultado final es un coche que puede navegar por un mapa, esquivar obstáculos, elegir rutas y llegar al destino seleccionado.

## Estructura del código

### Cargar el mapa
Lo primero que se hace es cargar el mapa y definir las celdas vecinas asignándolas un coste, que será 1 en las adyacentes y raíz de 2 las diagonales.
```python
vecinos = [ (-1,0,1.0),(1,0,1.0),(0,-1,1.0),(0,1,1.0), (-1,-1,sqrt(2)),(-1,1,sqrt(2)),(1,-1,sqrt(2)),(1,1,sqrt(2))]
```

### Generación del campo de costes

El campo de costes se genera desde la celda destino hacia el resto del mapa usando una expansión tipo BFS que acumula coste según el movimiento. Para limitar el coste computacional y evitar expandir todo el mapa, la expansión se corta cuando alcanza la celda del robot y sobrepasa un margen `exp_extra`
```python
campo = campo_ondas(df, dc, robot_fila, robot_col, exp_extra=15)
```

### Penalización por proximidad a obstáculos

Para evitar que el coche pase demasiado cerca de las paredes, se aplica una capa de penalización sobre el mapa. Primero se crea una máscara de obstáculos (celdas con valor 0). A continuación se calcula la distance transform sobre la negación de esa máscara para obtener, para cada celda, la distancia (en celdas) al obstáculo más cercano. Las celdas cuya distancia es menor o igual que un `radio` fijado reciben una penalización fija (`valor_penalizacion`).
```python
def penalizacion_obstaculos(mapa, radio, valor_penalizacion):
    mascara_obst = (mapa == 0)
    dist = distance_transform_edt(~mascara_obst)
    penal = np.zeros_like(mapa, dtype=float)
    penal[dist <= radio] = valor_penalizacion
    return penal
```

Estos dos puntos, es lo que se denomina PATH PLANNING, que consiste en generar el campo con sus costes y cada celda tendrá un valor descendiente desde el coche hasta el destino. Como resultado, el coche seguirá el descenso de gradiente.

Ahora, pasamos a la parte de navegación local, PATH NAVIGATION, en la que el coche debe ser capaz de navegar por el mapa y dirigirse hacia el destino. A continuación se explica cómo se hace.

### Selección del sub-objetivo

El sub-objetivo es la celda hacia la que el robot se dirige localmente. No se usa la celda más cercana, sino una celda segura, alejada del robot y con buen coste, lo que evita giros bruscos y oscilaciones. El algoritmo explora varias ventanas concéntricas alrededor del robot. Dentro de cada ventana se ordenan las celdas por su coste combinado (coste del campo + penalización).

Para cada candidata se aplican filtros (estabilidad):
  - "que no sea un muro"
  - "que pertenezca a la zona segura"
  - "que esté a más de una distancia mínima del robot"

Además, si el sub-objetivo anterior sigue siendo bueno, recibe una pequeña ventaja (histéresis) para no cambiarlo en cada iteración.

Dentro del bucle principal, vamos obteniendo las filas y las columnas del sub-objetivo local más estable:

```python
sub_f, sub_c = subobjetivo_estable(combinado, robot_fila, robot_col, seguro, sub_prev)
```

### Control y envío de velocidades 

Una vez elegido el sub-objetivo (`sub_f`, `sub_c`) se calcula el vector en celdas que apunta a él: 
`dx = sub_f - robot_fila`, `dy = sub_c - robot_col`.

Luego obtenemos el ángulo deseado hacia el sub-objetivo con `atan2(dy, dx)` y se normaliza para obtener un valor entre -π y π.

Se usan ganancias dependientes de la distancia al objetivo final, cerca del objetivo se usan `K_w` y `K_v` más suaves, lejos, ganancias más agresivas.

## Vídeo

[Vídeo del funcionamiento](https://drive.google.com/file/d/1guxLJTNSv8cSHiIZRw0bpvkQjSmWmex-/view?usp=sharing)



