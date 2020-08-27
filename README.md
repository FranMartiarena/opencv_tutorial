# Tutorial Opencv

## ¿Que es opencv?

Como sus dos ultimas siglas lo indican(computer vision), opencv es una libreria de vision por computadora  
que se puede programar con c, c++, python y java

## ¿Uso?

Esta libreria nos va a permitir  programar análisis y procesamiento de imágenes o vídeos, seguimiento/detección/reconocimiento de objetos, rostros, formas y un monton de cosas mas.

## Instalacion

Opencv hace uso de una libreria muy conocida llamada numpy, que nos va a facilitar las operaciones con imagenes, por lo que vamos a instalarlo a continuacion.

:warning: **Aviso**:Esta tutorial esta puramente orientado al lenguaje de programacion python y la instalacion solo esta explicada para los sistemas windows y linux.
(Aca yo recomiendo que si tienen un sistema operativo de distro linux lo usen, ya que en windows la Instalacion se puede volver un tanto tediosa debido a requerimientos)


### Windows
<pre>
pip install opencv-python
pip install numpy
</pre>
### Linux
<pre>
sudo apt-get install python-opencv
pip install numpy
</pre>

## Objetivo

Voy a cubrir un simple tutorial y desarrollar un programa que detecta el movimiento de personas, para ello van a necesitar un video el cual pueden descargar atravez de mi github o usar uno propio.

## Explicacion

Como ya se pueden imaginar, la vision por computadora es totalmente diferente a la de un ser humano.
La computadora representa toda imagen en forma de pixeles, en la que cada pixel va a tener un valor segun la la escala de colores que use.
Una de las escalas mas usada y conocidas es el espectro rgb(red, green, blue), que tiene un limite de 255 tonalidades por color, que si se combinan con los demas nos da 16.7 millones de combinaciones.
Opencv usa la escala bgr que cambia de lugar el rojo y azul, pero mas adelante explicare como jugar con esto.
Ahora bien, la computadora va a almacenar el valor del pixel en un array y nosotros vamos a poder manipularlo atravez de opencv.

## Mostrando Video

Lo primero que vamos a hacer es importar la libreria de opencv:
```python
import cv2
```
Despues le indicamos a opencv que capture el video seleccionado en una variable llamada cap (capture):
```python
cap = cv2.VideoCapture('video.avi')
```
Para poder reproducir el video lo que se hace es tomar una captura por cada frame del mismo y luego mostrarla, por lo que vamos a necesitar un bucle while e indicar que lea un frame de cap para luego mostrarlo:
```python
while cap.isOpened():#Mientras la captura de video este abierta, es decir, no termina
    bool, frame = cap.read() #El metodo read nos da 2 variables, un boolean y el frame que no es mas que un array con los valores de la imagen que capturo
    cv2.imshow(frame) #Aca le indicamos que nos muestre el frame
```
Ahora tenemos que llamar una funcion llamada waitKey(n) en la que se indica la cantidad de milisegundos que se mostrara la imagen, a su vez esta funcion lee en ascii las teclas presionadas durante su ejecucion, lo que aprovechamos a hacer un hotkey y asi salir del video:
```python
while cap.isOpened():
    bool, frame = cap.read()
    cv2.imshow("ventana", frame)

    if cv2.waitKey(40) == 27:#El numero 27 en ascii es la tecla escape
        break

cv2.destroyAllWindows()#Se eliminan las ventanas
cap.release()#Se termina la captura del video
```

## Background Substraction

Ahora bien, lo que queremos es detectar movimiento de personas en la imagen pero para eso necesitamos traer todo lo que no sea el fondo de la imagen(background), es decir queremos mostrar lo objetos de la superficie (foreground), esto se llama substraccion de fondo (background substraction) lo cual se logra tomando una imagen inicial y comparando los valores de los pixeles con la imagen siguiente restando asi los valores y obteniendo los que cambian. Para hacer esto existe un metodo de cv2 llamado createBackgroundSubtractorKNN() con el que modificaremos el frame:
```python
import cv2

cap = cv2.VideoCapture('video.avi')
substraction = cv2.createBackgroundSubtractorKNN()#se crea la substraccion de fondo

while cap.isOpened():
    bool, frame = cap.read()

    mask = substraction.apply(frame) #Aplicamos la substraccion al frame

    cv2.imshow("ventana1", frame) #Muestra el frame original
    cv2.imshow("ventana2", mask) #Muestra la substraccion


    if cv2.waitKey(40) == 27:
        break

cv2.destroyAllWindows()
cap.release()
```

## Morphological transformations

Al comparar las 2 ventanas, vemos que funciono pero que varias de las figuras estan deformadas y tienen manchas negras en su interior. Para sacar esto vamos a usar lo llamado  "transformacion morfologica de cierre" o "closing morphological transformation", que aplican una operacion sobre una imagen binaria para cambiarla, en nuestro caso seria para eliminar ese sonido dentro de las figuras:
```python
import cv2

cap = cv2.VideoCapture('video.avi')
substraction = cv2.createBackgroundSubtractorKNN()

while cap.isOpened():
    bool, frame = cap.read()

    mask = substraction.apply(frame)

    closing = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, None)#Aplicamos el closing sobre la substraccion

    cv2.imshow("ventana", closing) #Muestra el closing



    if cv2.waitKey(40) == 27:
        break

cv2.destroyAllWindows()
cap.release()
```
## Blur & Kernel

Si lograron ejecutar el codigo, veran que funciono, pero que todavia queda mucho sonido dispersado. Este tipo de sonido en especifico se lo denomina como "sonido sal y pimienta".
Para deshacer este sonido se suelen usar metodos de difuminado(blur). Existen muchos, pero el que mejor aplica a este tipo de sonido es el llamado "mediana de filtro" o "median filter", que recorre pixel por pixel y lo cambia a la media del valor de sus pixeles vecinos usando un "kernel". El kernel es una matriz con un tamaño determinado que tendra un valor en cada una se sus cuadriculas, el cual al ponerlo sobre la imagen, multiplicara el valor de cada cuadricula por el valor del pixel que tiene encima, y luego lo sumara obteniendo un nuevo valor para el pixel en el centro de la cuadricula. Lo aplicamos de la siguiente manera:
```python
import cv2

cap = cv2.VideoCapture('video.avi')
substraction = cv2.createBackgroundSubtractorKNN()

while cap.isOpened():
    bool, frame = cap.read()

    mask = substraction.apply(frame)

    closing = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, None)

    blur = cv2.medianBlur(closing, 5)#Aplicamos el blur al closing, con un kernel 5x5(los valores de las cuadriculas dependeran del blur)

    cv2.imshow("ventana", blur) #mostramos el blur

    if cv2.waitKey(40) == 27:
        break

cv2.destroyAllWindows()
cap.release()
```


## Thresholding

Ahora si podemos ver que se fue la mayoria del sonido, pero tenemos otro problema, se puede ver un sombreado gris devajo de las personas en movimiento. Aca es donde entra el thresholding, que sirve para realizar operaciones sobre la imagen y dividir los pixeles en 2 grupos. El threshold va a ir pixel por pixel realizando determinada operacion sobre el mismo. En este caso usariamos uno denominado "threshold to zero", y le indicamos que si el valor es menor a 150, osea gris o mas oscuro, que le cambie el valor a 0(negro):
```python
import cv2

cap = cv2.VideoCapture('video.avi')
substraction = cv2.createBackgroundSubtractorKNN()

while cap.isOpened():

    bool, frame = cap.read()

    mask = substraction.apply(frame)

    closing = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, None)

    blur = cv2.medianBlur(closing, 5)

    val, thresh = cv2.threshold(blur, 150 , 0, cv2.THRESH_TOZERO)#Nos devuelve 2 valores, thresh es el array de la imagen
    cv2.imshow("ventana", thresh) #mostramos el thresholding

    if cv2.waitKey(40) == 27:
        break

cv2.destroyAllWindows()
cap.release()
```

## Dilation

Luego del threshold vemos que las sombras se fueron, por lo que nos vamos a enfocar en dilatar cada figura para que sea mas facil de detectarla, para esto vamos a necesitar crear un kernel haciendo uso de la libreria numpy en el siguiente paso:
```python
import cv2
import numpy as np #Importamos numpy

cap = cv2.VideoCapture('video.avi')
substraction = cv2.createBackgroundSubtractorKNN()

while cap.isOpened():

    bool, frame = cap.read()

    mask = substraction.apply(frame)

    closing = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, None)

    blur = cv2.medianBlur(closing, 7)

    val, thresh = cv2.threshold(blur, 150 , 0, cv2.THRESH_TOZERO)

    kernel = np.ones((5, 5), np.uint8)#Creamos un kernel que sera de 5x5 y tendra un valor de uno en cada cudricula
    dilated = cv2.dilate(thresh, kernel=kernel, iterations=1)#dilatamos el threshold y hacemos uso del kernel

    cv2.imshow("ventana", dilated) #mostramos dilated

    if cv2.waitKey(40) == 27:
        break

cv2.destroyAllWindows()
cap.release()
```

## Contours

Cv2 nos da la opcion de trabajar con contornos atravez de una funcion. Los contornos nos permiten encerrar determinada seccion de la imagen, asi detectando figuras y luego aproximar su tamaño para saber si son personas:
```python
import cv2
import numpy as np

cap = cv2.VideoCapture('video.avi')
substraction = cv2.createBackgroundSubtractorKNN()

while cap.isOpened():

    bool, frame = cap.read()

    mask = substraction.apply(frame)

    closing = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, None)

    blur = cv2.medianBlur(closing, 7)

    val, thresh = cv2.threshold(blur, 150 , 0, cv2.THRESH_TOZERO)

    kernel = np.ones((5, 5), np.uint8)
    dilated = cv2.dilate(thresh, kernel=kernel, iterations=1)

    contours, _ = cv2.findContours(dilated, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)#Asi encontramos los contornos y los almacenamos en contours
    cv2.drawContours(frame, contours, -1, (0,255,0), 2)#Esta linea nos da la opcion de dibujar los contornos sobre la imagen original llamada frame

    cv2.imshow("ventana", frame)#Mostramos frame y podremos ver los contornos sobre la imagen

    if cv2.waitKey(40) == 27:
        break

cv2.destroyAllWindows()
cap.release()
```

## Rectangles

Para darle un toque final, vamos a ubicar un rectangulo sobre cada uno de los contornos(usaremos un bucle for para recorrer la variable contours) y le vamos a especificar que lo haga solamente si su area medida en pixeles es mayor a 500(area apartir de la cual se identifican las personas en este video).
El codigo final quedaria asi:
```python
import cv2
import numpy as np

cap = cv2.VideoCapture('video.avi')
substraction = cv2.createBackgroundSubtractorKNN()

while cap.isOpened():

    bool, frame = cap.read()

    mask = substraction.apply(frame)

    closing = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, None)

    blur = cv2.medianBlur(closing, 7)

    val, thresh = cv2.threshold(blur, 150 , 0, cv2.THRESH_TOZERO)

    kernel = np.ones((5, 5), np.uint8)
    dilated = cv2.dilate(thresh, kernel=kernel, iterations=1)

    contours, _ = cv2.findContours(dilated, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
    #No mostramos los contorno, los reemplazamos por los rectangulos(queda mejor)
    #cv2.drawContours(frame, contours, -1, (0,255,0), 2)


    for contour in contours:#Vamos contorno por contorno

        (x, y, w, h) = cv2.boundingRect(contour) #Delimitamos nuestro contorno con coordenadas  
        if cv2.contourArea(contour)<500: #si el area del contorno es menor a 500, pasa al siguiente contorno
            continue
        cv2.rectangle(frame, (x,y), (x+w, y+h), (0,255,0),2)#si no se cumple la condicion anterior, se dibuja un rectangulo en la imagen inicial sobre el contorno

    cv2.imshow("ventana", frame)#Mostramos frame y podremos ver los contornos sobre la imagen

    if cv2.waitKey(40) == 27:
        break

cv2.destroyAllWindows()
cap.release()
```
