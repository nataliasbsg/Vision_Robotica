# Práctica 1
Esta práctica consiste en la realización de un control reactivo PID implementado en un coche que corre en un circuito de Fórmula 1 implementando un código de Python.
El circuito contará con una línea roja en el medio de la carretera que habrá que seguir con un controlador de línea.
Dicho control se realizará gracias a la imagen obtenida por una cámara situada en la parte frontal de coche, aun asi esta imagen no estará completamente centrada. 
El objetivo principal es un buen seguimiento de la línea, tanto en rectas como en curvas, y a una velocidad que permita dar una vuelta al circuito en un tiempo inferior a 1 minuto.




## Introducción
La implementación de la práctica ha de realizarse en [Unibotics](https://unibotics.org/) y las instrucciones para su realización se encuentran en [jderobot](http://jderobot.github.io/RoboticsAcademy/exercises/AutonomousCars/follow_line/).
Para su realización es imprescindible la instalación de un Docker y seguir los pasos indicados para hacer el pull del Docker de Robotics Academy y poder comenzar a trabajar en el entorno de **Unibotics**.

Antes de comezar a enfrentarse a este reto, es imprescindible tener claros los conceptos en los que se van a basar nuestro código:
- En las rectas se correrá a la velocidad máxima y no se girará.
- En las curvas habrá que frenar y reducir esa velocidad, puesto que hay que girar y seguir manteniéndose en la línea.

A continuación, se muestra el circuito donde correrá nuestro coche:

![image](https://user-images.githubusercontent.com/72757217/111838213-4b825900-88f9-11eb-8ed7-8be5a4fc1aa9.png)


La velocidad variará de 0 a 5 unidades, lo que se traducirá en el programa de 0 a 120 km/h aproximadamente. 
Por otra parte, la velocidad de giro seguirá la regla de la mano derecha, siendo positiva cuando se gire a izquierdas y negativa cuando se gire a derechas, variando de -0.5 a 0.5 aproximadamente. 

En la siguiente imagen se muestra un velocidad de 65 km/h y una velocidad de giro nula:

![image](https://user-images.githubusercontent.com/72757217/111838081-1aa22400-88f9-11eb-8d1f-e442829b0ae0.png)
![image](https://user-images.githubusercontent.com/72757217/111837907-de6ec380-88f8-11eb-9371-c4551cf837bd.png)

## Primeros pasos
Lo primero que se ha de hacer es la detección de la línea, para ello se trabajará en el espacio de trabajo HSV y se realizará una máscara en los rangos en los que se encuentre el rojo. 

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

![image](https://user-images.githubusercontent.com/72757217/111822463-1c152180-88e4-11eb-9e76-b4028b353a56.png)

Las primeras pruebas se realizaron con un controlador proporcional y un código muy sencillo en el que la velocidad era muy baja (1.2) y el giro variaba únicamente en función del error multiplicado por una Kp de 0.0025. Hay que destacar el hecho de que esta Kp es tan baja porque la diferencia se mide en píxeles y su valor puede ir de 0 a la mitad del ancho de nuestra imagen, em este caso 240.

De esta forma se puede conseguir que el coche siga la línea y complete el circuito, pero no lo hará completamente encima de la línea y el tiempo en el que completará la vuelta será de aproximadamente un minuto y medio.

Los resultados de dicha prueba se muestran en el siguiente vídeo:

<iframe width="560" height="315" src="https://www.youtube.com/embed/lsyvYMFcFj4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>




## Controlador proporcional derivativo
Un control derivativo consiste en el control sobre el cambio del error en valor absoluto, suavizando la respuesta. 
Esto se hará multiplicando una constante, Kd, por la diferencia entre el error actual y el anterior, tal y como se muestra en la siguiente imagen:

![image](https://user-images.githubusercontent.com/72757217/111822359-fc7df900-88e3-11eb-97d6-78b65db88aa7.png)

En este paso se observó una mejoría notoria en los resultados alcanzando valores inferiores a 1 minuto y con mayor control sobre la línea. 
Sin embargo, se decidió optimizar más el código puesto que el cambio de recta a curva era demasiado brusco existiendo solo esta dos condiciones. Además, cabe destacar que en el circuito en el que se ha trabajado había curvas mucho más cerradas que otras, por lo que no se aprovechaba bien la velocidad en las curvas más abiertas.
Partiendo de esta premisa se decidió partir de una velocidad máxima, Vmax, que se reduciría al aumentar el error para disminuirlo más fácil, para controlar dicho cambio se introdujo otra constante,Kp_v. 
De esta forma quedaría una ecuación como la siguiente:

*velocidad = Vmax - abs (dif) * Kp_v*

Tras haber optimizado el código e integrado un controlador PD se obtuvieron los siguientes resultados al aumentar gradualmente la velocidad. Se ha de destacar que estos tiempos fueron tomados como media de tres vueltas que el coche fue capaz de dar. Sin embargo, al aumentar las velocidades el seguimiento de línea se ve también dificultado.

|velocidad|tiempo/vuelta|
|---------|-------------| 
|    3    |      37     | 
|   3.2   |      34     | 
|   3.4   |      33     | 
|   3.6   |      31     | 
|   3.8   |      30     | 
|    4    |      28     | 
|   4.2   |      30     | 
|   4.4   |  inestable  | 

Si analizamos la tabla se puede observar que al aumentar las velocidad los tiempos de cada vuelta aumentarán hasta llegar al tiempo récord de 28 segundos. 
Pero, si se aumenta más la velocidad el tiempo no mejorará debido a todas las inestabilidades que se crean. 
En una velocidad de 4.4 se crean tantas inestabilidades que el coche no podrá correr el circuito sin salirse en alguna de sus vueltas.

A continuación se presenta un vídeo en el que se muestra el funcionamiento con el controlador PD:

<iframe width="560" height="315" src="https://www.youtube.com/embed/oaHpta7cris" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>




## Controlador proporcional derivativo integral
Un control integral consiste en disminuir y eliminar el error en estado estacionario.
Esto se hará multiplicando una constante, Ki, por la suma del error actual y del anterior, tal y como se muestra en la siguiente imagen:

![image](https://user-images.githubusercontent.com/72757217/111822313-ec661980-88e3-11eb-9a95-e72708c5454b.png)

Al integrar un controlador PID se puede observar que los tiempos realmente no presentan mucha mejoría, pero sí la adhesión a la línea y la disminución del error.
A contincuación, se presenta una tabla resumen de varias pruebas que se realizaron modificando la velocidad máxima de nuestro coche.


|velocidad|tiempo/vuelta|
|---------|-------------| 
|    3    |      34     | 
|   3.2   |      32     | 
|   3.4   |      30     | 
|   3.6   |      29     | 
|   3.8   |      28     | 
|    4    |      28     | 
|   4.2   |      28     | 
|   4.4   |  inestable  | 

En este caso, a partir de 3.8 la velocidad máxima se estanca puesto que aunque le pidas al coche que corra más las inestabilidades que se producen no lo permiten.

Tras un pequeño reajuste se obtuvieron los siguientes resultados:

- Modo **seguro**: realiza el circuito de una forma segura, encima de la línea, priorizando la calidad frente a la velocidad. Es capaz de realizar una vuelta completa en 38 segundos.

<iframe width="560" height="315" src="https://www.youtube.com/embed/beoGeE6gHhw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

- Modo **standard**: realiza la vuelta en un buen tiempo, pero en las curvas es más brusco. Es capaz de realizar una vuelta completa en 32 segundos.

<iframe width="560" height="315" src="https://www.youtube.com/embed/KXr_CEd40U8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

- Modo **rápido**: velocidad máxima que se puede alcanzar sin salirse del circuito dando varias vueltas. Es capaz de realizar una vuelta completa en 24 segundos.

<iframe width="560" height="315" src="https://www.youtube.com/embed/94zmJf2hE4g" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>




## Sugerencias de mejoras en Unibotics
La principal dificultad que presenta Unibotics es que no funciona correctamente en cualquier ordenador ya que es bastante exigente con la CPU.
Aun contando mi ordenador con un Intel Core i7-4712MQ a 2.3GHz con 4 núcleos y 8 hilos, Gazebo era incapaz de iniciarse, al igual que el código con una Brain Frequency superior a 10. De hecho, partiendo de estas limitaciones el código solo funcionaba en raras ocasiones.
Debido a estas dificultades opté por pedir a un compañero correr mi código en su ordenador en remoto y ahí es cuando pude comprobar el buen funcionamiento de mi código y fui capaz de optimizarlo.




