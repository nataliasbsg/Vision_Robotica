## Práctica 1
Esta práctica consiste en la realización de un control reactivo PID implementado en un coche que corra en un circuito de Fórmula 1 implementando un código de Python.
El circuito contará con una línea roja en el medio de la carretera que habrá que seguir con un controlador de línea.
Dicho control ha de permitir un buen seguimiento de la línea, tanto en rectas como en curvas, y a una velocidad que permita dar una vuelta al circuito en un tiempo inferior a 1 minuto.

# Introducción
La implementación de la práctica ha de realizarse en [Unibotics](https://unibotics.org/) y las instrucciones para su realización se encuentran en [jderobot](http://jderobot.github.io/RoboticsAcademy/exercises/AutonomousCars/follow_line/).
Para su realización es imprescindible la instalación de un Docker y seguir los pasos indicados para hacer el pull del Docker de Robotics Academy y poder comenzar a trabajar en el entorno de Unibotics.

Antes de comezar a enfrentarse a este reto, es imprescindible tener claros los conceptos en los que se van a basar nuestro código:
- En las rectas se correrá a la velocidad máxima y no se girará.
- En las curvas habrá que frenar y reducir esa velocidad, puesto que hay que girar y seguir manteniéndose en la línea.

La velocidad variará de 0 a 5 unidades, lo que se traducirá en el programa de 0 a 120 km/h aproximadamente. 

Por otra parte, la velocidad de giro seguirá la regla de la mano derecha, siendo positiva cuando se gire a izquierdas y negativa cuando se gire a derechas.

# Primeros pasos
Lo primero que se ha de hacer es la detección de la línea. Para ello se trabajará en el espacio de trabajo HSV y se realizará una máscara en los rangos en los que se encuentre el rojo. 

Una vez realizada la máscara se ha de tener en cuenta que la imagen puede contener rojos que no pertenezcan a esa línea. 
Por ejemplo, en algunas ocasiones se puede observar la parte delantera de nuestro vehículo, la cual también es roja.
Para evitar dichos errores se realizará una limpieza con operaciones de dilatación y erosión.

Tras detectar la línea hay que plantearse como realizar su seguimiento. En nuestro caso, se ha decidio trabajar con un punto colocado al horizonte de dicha línea para poder anticiparnos ante los cambio de líneas y curvas.
Para ello se han limitado los rangos de la línea a los más lejanos, es decir, los más cercanos al centro de nuestra imagen. Una vez establecidos dichos límites se ha realizado el controno de la máscara y se ha calculado su centro de masas para mostrar un punto central en la línea.
De esta forma, se obtiene un punto de control situado en el centro de la línea en el horizonte.

![image](https://user-images.githubusercontent.com/72757217/111555528-df80e300-8788-11eb-86c0-ee9040395d15.png)


Ahora bien, ¿cómo podemos trabajar con ese punto de control? Es ahora cuando entran en juego los controladores proporcionales, derivativos e integrales. 
La función de estos controladores es "controlar" el error. Dicho error será la diferencia entre la posición del punto y el centro de nuestra imagen contemplando únicamente el eje horicontal. 
Es decir, se pretende que el punto se mantenga en la mitad del ancho de la imagen y para ello cada controlador cumplirá una funcionalidad diferente.
