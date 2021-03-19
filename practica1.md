# Práctica 1
Esta práctica consiste en la realización de un control reactivo PID implementado en un coche que corra en un circuito de Fórmula 1 implementando un código de Python.
El circuito contará con una línea roja en el medio de la carretera que habrá que seguir con un controlador de línea.
Dicho control se realizará gracias a la imagen obtenida por una cámara situada en la parte frontal de coche, aun asi esta imagen no estará completamente centrada. 
El objetivo principal es un buen seguimiento de la línea, tanto en rectas como en curvas, y a una velocidad que permita dar una vuelta al circuito en un tiempo inferior a 1 minuto.


## Introducción
La implementación de la práctica ha de realizarse en [Unibotics](https://unibotics.org/) y las instrucciones para su realización se encuentran en [jderobot](http://jderobot.github.io/RoboticsAcademy/exercises/AutonomousCars/follow_line/).
Para su realización es imprescindible la instalación de un Docker y seguir los pasos indicados para hacer el pull del Docker de Robotics Academy y poder comenzar a trabajar en el entorno de **Unibotics**.

Antes de comezar a enfrentarse a este reto, es imprescindible tener claros los conceptos en los que se van a basar nuestro código:
- En las rectas se correrá a la velocidad máxima y no se girará.
- En las curvas habrá que frenar y reducir esa velocidad, puesto que hay que girar y seguir manteniéndose en la línea.

La velocidad variará de 0 a 5 unidades, lo que se traducirá en el programa de 0 a 120 km/h aproximadamente. 

Por otra parte, la velocidad de giro seguirá la regla de la mano derecha, siendo positiva cuando se gire a izquierdas y negativa cuando se gire a derechas.

## Primeros pasos
Lo primero que se ha de hacer es la detección de la línea. Para ello se trabajará en el espacio de trabajo HSV y se realizará una máscara en los rangos en los que se encuentre el rojo. 

Una vez realizada la máscara se ha de tener en cuenta que la imagen puede contener rojos que no pertenezcan a esa línea. 
Por ejemplo, en algunas ocasiones se puede observar la parte delantera de nuestro vehículo, la cual también es roja.
Para evitar dichos errores se realizará una limpieza con operaciones de dilatación y erosión.
El resultado de dichas operaciones se muestra en la siguiente imagen:

![image](https://user-images.githubusercontent.com/72757217/111777967-4818ae80-88b4-11eb-8f39-45ef4b71dac2.png)

Tras detectar la línea hay que plantearse como realizar su seguimiento. En nuestro caso, se ha decidido trabajar con un punto colocado al horizonte de dicha línea para poder anticiparnos ante los cambio de líneas y curvas.
Para ello se han limitado los rangos de la línea a los más lejanos, es decir, los más cercanos al centro de nuestra imagen. Una vez establecidos dichos límites se ha realizado el contorno de la máscara y se ha calculado su centro de masas para mostrar un punto central en la línea.
De esta forma, se obtiene un punto de control situado en el centro de la línea en el horizonte.
En la siguiente imagen se muestra el resultado obtenido:

![image](https://user-images.githubusercontent.com/72757217/111778938-bb6ef000-88b5-11eb-8a2c-8e4921ef82a6.png)

Ahora bien, ¿cómo podemos trabajar con ese punto de control? Es ahora cuando entran en juego los controladores proporcionales, derivativos e integrales. 
La función de estos controladores es "controlar" el error. Dicho error será la diferencia entre la posición del punto y el centro de nuestra imagen contemplando únicamente el eje horizontal. 
Es decir, se pretende que el punto se mantenga en la mitad del ancho de la imagen y para ello cada controlador cumplirá una funcionalidad diferente.


## Controlador proporcional
Un control proporcional consiste en la multiplicación de una constante o ganancia por el error calculado, en este caso la diferencia entre el centro de la imagen y el punto calculado en la línea roja (únicamente respecto al eje horizontal).

![image](https://user-images.githubusercontent.com/72757217/111780799-2d483900-88b8-11eb-9309-79dcd4ef2382.png)

Las primeras pruebas se realizaron con un controlador proporcional y un código muy sencillo en el que la velocidad era muy baja (1.2) y el giro variaba únicamente en función del error multiplicado por una Kp de 0.0025. Hay que destacar el hecho de que esta Kp es tan baja porque la diferencia se mide en píxeles y su valor puede ir de 0 a la mitad del ancho de nuestra imgen, em este caso 240.

De esta forma se puede conseguir que el coche siga la línea y complete el circuito, pero no lo hará completmente encima de la línea y el tiempo en el que completará la vuelta será de aproximadamente un minuto y medio.

Los resultados de dicha prueba se muestran en el siguiente vídeo:


## Controlador derivativo

