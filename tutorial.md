¿Que es opencv?

Como sus ultimas siglas lo indican(computer vision), opencv es una libreria de vision por computadora desarrollada por intel,
que se puede programar con c, cpp, python y java

¿Uso?

Esta libreria nos va a permitir  programar análisis y procesamiento de imágenes o vídeos, seguimiento, detección y reconocimiento de objetos, rostros, formas y un monton de cosas mas.

Instalacion

AVISO:Esta tutorial esta puramente orientado al lenguaje de programacion python y la instalacion solo esta explicada para los sistemas windows y linux.
(Aca yo recomiendo que si tienen un sistema operativo de distro linux lo usen, ya que en windows la Instalacion se puede volver un tanto tediosa debido a requerimientos)

Opencv hace uso de una libreria muy conocida llamada numpy para hacer operaciones numericas avanzadas con imagenes, por lo que tendremos que instalarlo

Windows

pip install opencv-python

Linux

sudo apt-get install python-opencv

Objetivo

Que aprendan lo simple de opencv.
Voy a cubrir un simple tutorial y desarrollar un programa para la deteccion de movimiento de personas, por eso van a necesitar un video el cual pueden descargar atravez de mi github o usar uno suyo.

Explicacion

Como ya se pueden imaginar, la vision por computadora es totalmente diferente a la de un ser humano.
La computadora representa toda imagen en forma de pixeles, en la que cada pixel va a tener un valor segun la la escala de colores que use.
Una de las escalas mas usada y conocidas es el espectro rgb(red, green, blue), que tiene un limite de 255 tonalidades por color, que si se combinan con los demas nos da 16.7 millones de combinaciones.
Opencv usa la escala bgr que cambia de lugar el rojo y azul, pero mas adelante explicare como jugar con esto.
Ahora bien, la computadora va a almacenar el valor del pixel en un array y nosotros vamos a poder manipularlo atravez de opencv.

Mostrando Video

Lo primero que vamos a hacer es importar la libreria de opencv:

import cv2

Despues le indicamos a opencv que capture el video seleccionado en una variable llamada cap (capture):

cap = cv2.VideoCapture('video.avi')

Para poder reproducir el video lo que se hace es tomar una captura por cada frame del mismo y luego mostrarla, por lo que vamos a necesitar un bucle while e indicar que lea un frame de cap para luego moostrarlo.

while cap.isOpened():#Mientras la captura de video este abierta, es decir, no termina
    bool, frame = cap.read() #El metodo read nos da 2 variables, un boolean y el frame que no es mas que un array con los valores de la imagen que capturo
    cv2.imshow(frame) #Aca le indicamos que nos muestre el frame

Ahora tenemos que llamar una funcion llamada waitKey(n) en la que se indica la cantidad de milisegundos que se mostrara la imagen, a su vez esta funcion lee en ascii las teclas presionadas durante su ejecucion, lo que aprovechamos a hacer un hotkey para salir del video.

while cap.isOpened():#Mientras la captura de video este abierta, es decir, no termina
    bool, frame = cap.read() #El metodo read nos da 2 variables, un boolean y el frame que no es mas que un array con los valores de la imagen que capturo
    cv2.imshow(frame) #Aca le indicamos que nos muestre el frame

    if cv2.waitKey(40) == 27:#El numero 27 en ascii es la tecla escape
        break

cv2.destroyAllWindows()#Se eliminan las ventanas
cap.release()#Se termina la captura del video

Deteccion

Ahora bien, lo que queremos es detectar movimiento de personas en la imagen pero para eso necesitamos traer todo lo que no sea el fondo de la imagen(background), es decir queremos mostrar lo objetos de la superficie (foreground), esto se llama substraccion de fondo (background substraction) lo cual se logra tomando una imagen inicial y comparando los valores de los pixeles con la imagen siguiente restando asi los valores y obteniendo los que cambian. Para hacer esto existe un metodo de cv2 llamado createBackgroundSubtractorKNN() con el que modificaremos el frame.

import cv2

cap = cv2.VideoCapture('video.avi')
substraction = cv2.createBackgroundSubtractorKNN()#se crea la substraccion de fondo

while cap.isOpened():#Mientras la captura de video este abierta, es decir, no termina
    bool, frame = cap.read() #El metodo read nos da 2 variables, un boolean y el frame que no es mas que un array con los valores de la imagen que capturo

    mask = substraction.apply(frame) #Aplicamos la substraccion al frame

    cv2.imshow(frame) #Aca le indicamos que nos muestre el frame


    if cv2.waitKey(40) == 27:#El numero 27 en ascii es la tecla escape
        break

cv2.destroyAllWindows()#Se eliminan las ventanas
cap.release()#Se termina la captura del video

Si lograron ejecutar el codigo, veran que funciono, pero que hay mucho sonido dispersado:

![Alt text](sonido.png)
