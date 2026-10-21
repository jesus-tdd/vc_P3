# Práctica 3. Detección y reconocimiento de formas

## Tarea 1: Detección de monedas

Se detectan las monedas en cámara y se cuenta la cantidad de dinero.

El código para esta tarea está separado en dos bloques: uno con funciones y otro con el bucle principal; de modo que el código fuera más limpio.

Se debe seleccionar la moneda de referencia con un click.
El valor de esta se puede editar en la variable `REF` del código, en el bloque de funciones.


### Funciones
A continuación se explican las funciones del bloque de funciones.


#### `on_mouse()`
Registra los clicks del ratón.


#### `classify(d_px)`
Una vez calculada la equivalencia entre píxeles y milímetros (en el bucle principal),
se encarga de traducir esto al valor de una moneda a partir de su diámetro.

La relación entre los diámetros de las monedas y su valor se almacena en el diccionario `CENT_DIAM_MM`.

Esta función tiene una cierta tolerancia,
se puede modificar en la variable `TOL_MM`.

El método retorna una tupla con el valor de la moneda y su diámetro calculado en milímetros.


#### `segment(frame)`
Se encarga de realizar las operaciones necesarias en el fotograma de la cámara antes de proceder a la detección de las monedas.

Primero se pasa la imagen a escala de grises y se le aplica un desenfoque gaussiano para reducir el ruido.

Después, se umbraliza la imagen.
Para reducir el efecto de elementos del ambiente como la iluminación, se ha usado una función de umbralizado adaptativa.
De este modo, se ha logrado que el umbralizado se realizara teniendo encuenta los píxeles en la vecindad
en lugar de los píxeles de toda la imagen.
Al aplicar esto se ha observado una mejora sustancial en los resultados frente al umbralizado fijo.

Por último, se hacen operaciones para eliminar el ruido causado por el umbralizado.
Esto último ha sido un consejo de la Inteligencia Artificial Claude.ai. 

El método retorna el fotograma tras las operaciones. 


#### `detect(frame)`
Es la función que,
tras el umbralizado,
realiza la detección de contornos.

A los contornos detectados se les aplica un filtro de
área, perímetro cerrado y circularidad para reducir el riesgo de errores.

El área mínima se puede editar en la variable `MIN_AREA`.

La circularidad mínima se puede editar en la variable `MIN_CIRC`.

La función retorna una tupla con el contorno, su centro y su diámetro.


### Bucle principal
Emplea los métodos detallados anteriormente para detectar monedas a partir de los fotogramas de la cámara y calcular su valor total.

Además, una vez seleccionada la moneda de referencia,
se muestra el valor de cada moneda detectada por separado.


## Tarea 2: Clasificador

El código para esta tarea está separado en 4 celdas.


### Celda 1. Segmentación

Para la detección de partículas se tiene en cuenta el color del fondo, 
estimado mediante el cálculo de la mediana de los píxeles de la imagen.

Una característica a tener en cuenta es el color de las partículas,
tanto para no perder partículas claras
como para distinguir de forma más precisa entre los tipos de las mismas.
Para facilitar la detección del color (principalmente de la intensidad del mismo), la imagen se trabaja en formato LAB. 

Para el umbralizado de la imagen,
se hace uso de *Otsu* para calcular el factor de umbralizado.

Salvando los detalles mencionados,
el proceso de segmentación es análogo al de la **tarea 1**. 

El resultado de la celda son las imágenes de entrenamiento y test 
con el contorno de las partículas encontradas trazado en verde.
Esto se ha usado para calibrar el área máxima y mínima, así como el umbralizado mínimo.

Los factores previamente mencionados son configurables en las variables `MAX_AREA`, `MIN_AREA` y `MIN_T` respectivamente.


### Celda 2. Detección de características

En esta celda se calculan las características que se utilizarán para clasificar las partículas:
- Circularidad
- Solidez (rugosidad)
- Relación de ejes
- Saturación
- Brillo respecto al fondo

Los pellets poseen circularidad alta y colores apagados.
Los fragmentos tienen formas irregulares y colores vivos.
El alquitrán es oscuro y rugoso.


### Celda 3. Entrenamiento del clasificador

Como las imágenes de entrenamiento únicamente poseen un tipo de partícula,
el clasificador aprende qué características representan a cada tipo a partir de ellas.

En el diagrama de cajas mostrado al ejecutar la celda,
se aprecia la diferencia entre la distribución de las características
para cada tipo de partícula.


### Celda 4. Resultado

En esta celda se pone el clasificador a prueba,
evaluando los resultados obtenidos frente a los esperados
mediante una matriz de confusión.

En la matriz, se observa que el clasificador distingue casi a la perfección los fragmentos de las otras partículas.
El alquitrán, por su parte, parece causarle más problemas, aunque mantiene una proporción alta de aciertos.
Por último, se observa que tiene tendencia a confundir los pellets con fragmentos.

El clasificador ha pasado por alto 86 partículas de la imagen test.
Teniendo en cuenta las partículas detectadas, ha obtenido una precisión del 79,4%. 
