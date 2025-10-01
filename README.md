# Blog_RobMovil

Para realizar esta primera práctica mi código consiste en un bucle infinito en el que dependiendo de las necesidades, el robot entrará en un estado u otro.
Los tres estados que se han establecido son: ir hacia delante, ir hacia detrás y girar.

# Delante
En el estado de ir hacia delante, el robot realiza una espiral en la que en cada iteración va aumentando la velocidad en 0.001 para que vaya haciendo cada vez un radio mayor. Si se mantuviese la velocidad en cada iteración, la espiral no iría aumentando y se limpiaría únicamente en círculos. Esta espiral se hace para que el robot haga un barrido de una superficie mayor en lugar de ir en línea recta. Además, es difícil ajustar las velocidades para que el robot no vaya ni demasiado rápido ni demasiado lento.

# Detrás
En este estado, el robot toma una velocidad negativa y la velocidad angular se mantiene en 0 para que sólo vaya hacia detrás el robot. Aquí se toma el tiempo en el que entra en el estado para que este estado se ejecute durante un determinado tiempo. Si no se hace esto, el robot continúa yendo marcha atrás y no cambia de estado.

# Giro
Este estado se ejecuta después del estado de ir hacia detrás y, al igual que el anterior, se ejecuta durante un tiempo determinado al inicio del código ya que si no seguiría girando. Además, en este estado se toma un valor al azar entre -1 y 1. Este valor al azar será la velocidad angular que el robot va a tomar. En este estado de giro, a diferencia que en el anterior, la velocidad lineal será 0 y lo único que hay es velocidad angular.

# Problemas y soluciones
Durante la programación del robot me he encontrado diversos problemas:

- Uno de los problemas ha sido el establecer un tiempo para los estados de ir hacia detrás y de girar. He establecido un tiempo de 1 segundo para cada estado y se verifica en estos estados si se ha cumplido el tiempo para cambiar al siguiente estado o no.

- Otro problema que he encontrado ha sido el que al no reiniciar la velocidad, el robot cada vez que entraba en el estado de ir hacia delante me iba demasiado rápido. Para solucionarlo, se reinicia la velocidad a la velocidad inicial en el estado de giro para que así al volver a entrar en el estado de ir hacia delante empiece con la velocidad inicial.

- También he encontrado dificultad, aunque en menor medida, a la hora de hacer la espiral. Lo que me ocurría es que me limpiaba en círculos, para ello, me di cuenta que había que ir aumentando en cada iteración la velocidad para que así el robot vaya barriendo en un rango mayor y aumentando así la espiral.

# Vídeo
https://github.com/user-attachments/assets/3a7e8d3a-c95c-4436-a92c-f33215912ab5
