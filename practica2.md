# Reconstrucción 3D

En esta práctica se pretende generar una reconstrucción 3D a partir de las imágenes obtenidas por dos cámaras, denominadas durante la práctica izquierda y derecha. 
Con las imágenes obtenidas por estas cámaras se va a reconstruir en 3D la siguiente escena:


![image](https://user-images.githubusercontent.com/72757217/124360113-3b805200-dc28-11eb-8b68-6d1a0ac6c668.png)


La escena es fotografiada pos las cámaras obteniendo las siguientes imágenes:


![image](https://user-images.githubusercontent.com/72757217/124362824-b7ce6180-dc37-11eb-8993-42f932887b7e.png)


En la siguiente página de [jdetobot](https://jderobot.github.io/RoboticsAcademy/exercises/ComputerVision/3d_reconstruction#theory) 
se encuentran explicados todos los pasos a seguir para poder trabajar con [Unibotics](https://unibotics.org/academy/exercise/3d_reconstruction/).


Lo primero que se hace es transformar las imágenes a escala de grises para poder aplicar el algoritmo de Canny y poder extraer los bordes.


![image](https://user-images.githubusercontent.com/72757217/124365549-1b14bf80-dc49-11eb-8209-f0828a49d96e.png)


Con el fin de limpiar un poco los datos reduciendo el número de puntos se aplicará un filtro bilateral.
De esta forma se obtienen las siguientes imágenes:


![image](https://user-images.githubusercontent.com/72757217/124358878-a038ae00-dc22-11eb-8f9c-797909083172.png)


Las imágenes originales también se transformarán al espacio de color HSV, que es más robusto ante los cambios de iluminación.
![image](https://user-images.githubusercontent.com/72757217/124365519-f02a6b80-dc48-11eb-8e60-2d3949cfdead.png)


A continuación, se calcula el rayo de retroproyección de la cámara izquierda a la cámara derecha utilizando las funciones facilitadas por Unibotics:
- HAL.getCameraPosition: devuelve la posición de la cámara.
- HAL.backproject: vuelve a proyectar un punto 2D al sistema de referencia 3D.
- HAL.graficToOptical: transforma el sistema de coordenadas de la imagen en el sistema de coordenadas de la cámara.
- HAL.project: proyecta un punto 3D de la escena sobre un punto 2D del sistema de imágenes.
- HAL.opticalToGrafic: transforma un punto en el sistema 3D de la cámara al sistema de imagen.

A partir del rayo de retroproyección se crea una máscara que contiene la línea epipolar. Para ello, se toman dos puntos del rayo de retroproyección y se proyectan sobre la imagen de la derecha.


Para encontrar los puntos homólogos a partir de esta epipolar se utiliza la función matchTemplate. Con la función minMaxLoc se encuentran los puntos con más correspondencia.
Además, para considerar homólogos dos puntos su coeficiente de correspondecia ha de ser superior al 90%.
Una vez obtenidos se pueden calcular los rayos de retroproyección y ver dónde se cruzan. Debido a que las líneas de retroproyección no son perfectas habrá una distancia b entre estos rayos. Para minimizar y solventar ese error se calcula una solución por mínimos cuadrados a partir del punto medio de la distancia que separa ambas líneas (b/2).

![image](https://user-images.githubusercontent.com/72757217/124365799-9a56c300-dc4a-11eb-843b-fa963330f325.png)
Fuente: José Miguel Buenaposada Biencinto.


## Resultados


Se han pintado 10.000 puntos al azar (random) obteniendo los siguientes resultados:


![image](https://user-images.githubusercontent.com/72757217/124365972-3f25d000-dc4c-11eb-8225-5d43c5395490.png)
![image](https://user-images.githubusercontent.com/72757217/124365998-75634f80-dc4c-11eb-8192-c680ec77d652.png)


Los resultados anteriores se pueden mejorar pintando más puntos, pero el tiempo de cómputo es mayor. En este caso se pintaron 15.000:


![image](https://user-images.githubusercontent.com/72757217/124366043-02a6a400-dc4d-11eb-95f9-56c8f92ed256.png)
![image](https://user-images.githubusercontent.com/72757217/124366059-2669ea00-dc4d-11eb-95d6-bbeeb1059054.png)


Por último, se intentaron pintar 20.000 pero el programa no fue capaz debido a que se  pierde la conexión con el Docker antes de poder pintar los puntos.



