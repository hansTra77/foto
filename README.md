# Proyecto 3
# Face Morphing
Universidad ICESI  
Curso: Fotografia Computacional
Profesor: Carlos Alberto Arce Lopera
Estudiantes: Juan Pablo Medina Mora, Nelson David Padilla Hernández, David Ernesto Quiñónez Bolaños

Link: 

## Introduccion

En el presente proyecto se busca lograr el morphing entre las diferentes caras de los integrantes del grupo desarrollador, todo por medio del API de tratamiento de imagenes OpenCV. El proceso que se siguio para realizar la tarea propuesta fue encontrar los puntos principales de los rostros a combina haciendo uso de la demo del API Face ++ que puede encontrarse en http://www.faceplusplus.com/demo-landmark/.

Con los puntos guia encontrados se procedio a realizar la triangulacion de los mismos, esto se realizo aplicando Delaunay Triangulation, por medio de la clase Subdiv2D de la libreria OpenCV. como resultado del paso anterior tenemos la lista de triangulos que serviran para realizar el warping de las caras, el warping se realizo por medio del metodo getAffineTransform de Subdiv2D.
Finalmente haciendo uso de warpAffine se logra la serie de imagenes de las caras mezclandose.

Con las imagenes ya combinadas se procedio a realizar el gif del resultado. En el presente escrito se expone con mayor detalle cada uno de los pasos anteriormente descritos, y adicionalmente se presentan los resultados, problemas encontrados en el desarrollo y una serie de conclusiones.

A continuacion, se observan las nueve fotos utilizadas en el proyecto.


## Desarrollo

### Encontrar los puntos mas sobresalientes para realizar la deteccion facial

El primer paso que se realizo para desarrollar el proyecto fue encontrar los puntos sobresalientes de todas las immagenes utilizadas. Con este fin se hizo uso del demo de la libreria Face ++, la cual se especializa en la deteccion de rostros en fotografias, y una de sus opciones consiste en encontrar los puntos sobresalientes de los rostros (Face Landmarks). En las figuras se observan los resultados obtenidos con el programa.



los resultados obtenidos para procesar los puntos se encontraban en formato JSON y tenian el siguiente formato.

```json
{
  "result": [
    {
      "face_id": "c8767e6e274cc048ac07263041c80eae",
      "landmark": {
        "contour_chin": {
          "x": 49.493067,
          "y": 70.684
        },
        "contour_left1": {
          "x": 27.565067,
          "y": 41.2646
        },
       
```

Los valores generados por Face ++ fueron almacenados en archivos txt, que posteriormente fueron procesados para quedar en un formato mas facil de leer, el cual se muestra a continuacion.

```
49.493067 70.684
27.565067 41.2646
28.277067 45.4024
28.889333 49.7924
29.501067 53.7798
30.578133 57.4448
32.8728 60.5324
35.619733 63.9434
39.1368 67.1922
43.621867 69.6314
.
.
.
```

En este nuevo formato cada linea hace referencia a un punto y se organiza de forma: x y, con un espacio entre las componentes. En este punto de el proceso aun no se podia hacer uso de estos puntos, dado que no hace uso de coordenadas relativas  a la imagen utilizada, sino que usa coordenadas absolutas con valores del 1 al 100. Para obtener los valores reales de los pixeles que ocupaba cada punto en la imagen se hizo uso del siguiente codigo en Java.

```java
package aPixel;

import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.util.ArrayList;

import processing.core.PApplet;

public class principal extends PApplet{
	
	ArrayList<String> x = new ArrayList<String>();
	ArrayList<String> y = new ArrayList<String>();
	
	public void setup() {
		
		String cadena;
			
		FileReader read;
		try {
			read = new FileReader("../8.txt");
			BufferedReader buf = new BufferedReader(read);
			int i = 0;
			while((cadena = buf.readLine()) != null) {
				
				String [] actual = cadena.split(" ");
				System.out.println(i + " " + actual[0] + " " + actual[1]);
				String a = actual[0];
				String b = actual[1];
				x.add(a);
				y.add(b);
				i ++;
				
			}
			
			buf.close();

		} catch (FileNotFoundException e1) {
			// TODO Auto-generated catch block
			e1.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		System.out.println(x.size());
		System.out.println(y.size());
		
		for(int i = 0; i <= x.size() - 1; i ++) {
			
			float iy = Float.valueOf(y.get(i));
			float ix = Float.valueOf(x.get(i));
			
			int yy = (int) (map(iy, 1, 100, 0, 500));
			int xx = (int) (map(ix, 1, 100, 0, 375));
			
			System.out.println(xx + " " + yy);
			
		}

		//

	}

}

```

El anterior codigo se encarga de transformar los puntos de un valor porcentual a un valor en pixeles por medio de un mapeo de valores del uno a cien a otro de cero al tamaño de la imagen en pixeles (imagenes de 375 x 500 pixeles). Haciendo uso del programa Java se consiguio que los txt de los puntos quedasen de la siguiente forma.

```
183 351
100 203
103 224
105 246
107 266
112 285
120 300
.
.
.
```

Para finalizar con este primer paso se agregaron puntos extra a cada imagen, tres para la nariz, dos para los hombros, dos para las orejas y otros que corresponden al borde de las fotos, en total 97 puntos por foto.

### Triangulacion

Con los nueve archivos de puntos se procedio a realizar la triangulacion con el metodo Delaunay Triangulation que se encuentra en la clase Subdiv2D de OpenCV, pero para que las triangulaciones de las imagenes fueran las correctasse realizo un preprocesamiento, este consistio en promediar los puntos encontrados en cada rostro, con el fin de que para la triangulacion solo se hiciera uso de una lista de puntos y que por tanto las triangulaciones resultantes fueran iguales para cada rostro y no se generaran malformaciones.

Con el nuevo grupo de puntos se procedio a efectuar la triangulacion. En primer lugar se creo un arreglo con los puntos que se encontraban en el txt de puntos, con los puntos ya guardados en el arreglo se creo un espacio que representaria la imagen y seria posteriormente dividido. Despues se creo la instancia de Subdiv2D al que se insertaron los puntos del arreglo, para finalmente llamar el metodo getTriangleList() que retorna una lista con los triangulos encontrados usando el indice de los puntos que lo conforman como se muestra a continuacion.

```
15 94 76
27 12 16
41 1 56
7 16 9
14 27 34
.
.
.
```

### Warping de las imagenes

Con la triangulacion de los puntos reaizada se procede a hacer el warping de las imagenes, para tal fin se hizo uso de la ecuacion 1, con la cual se pueden localizar la posicion de las caracteristicas sobresalientes en la imagen combinada. 



Con los puntos sobresalientes de la imagen combinada mas los puntos encontrados con Face ++ para las imagenes y el resultado de la triangulacion se proedio a calcular las transformaciones afines, es decir mapear los puntos que conforman cada triangulo encoontrado de la imagen original a la imagen combinada. En OpenCV esto se realiza por medio del metodo getAffineTransform(), que encuentra la transformacion afin para cada uno de los triangulos que conforman la imagen, este mismo proceso se realizo con la segunda y tercera imagen a combinar.

Finalmente se procede a realizar el warping como tal y unirlas de forma que no se vean cambios muy agresivos, esto se realiza por medio de la ecuacion 2 y de los metodos warpAffine() y fillConvexPoly().

El anterior proceso se realizo dentro de un for que modificaba el valor del alfa con el fin de generar asi 45 imagenes que mostraran la transformacion de las tres caras a mas detalle. Las imagenes resultantes se crearon una a una en una carpeta.

### Creacion del GIF

Finalmente se realizaron los gif animados.


## Problemas encontrados en el desarrollo

Durante el desarrollo del proyecto no hubo grandes complicaciones, pero si se debio investigar como se habia resuelto con anterioridad el problema propuesto y que soluciones brindaba OpenCV para poder resolverlo. El segundo problema que se encontro y no se esperaba era que los puntos generados por la libreria Face ++ no eran de inmediato utiles, y se necesitaba de un preprocesamiento para que pudieran ser utilizados.

## Conclusiones

Con la realizacion del proyecto pudimos encontrar que el campo de la fotografia computacional es muy amplio y complejo, pero en la actualidad casi todo ya se encuentra hecho y a disposicion de quienes lo necesitan, lo unico que se necesita es aber buscar y aprendeer a usar las herramientas.
Es importante tambien mencionar que por medio de este proyecto pudimos observar las posibilidades de este campo de la computacion, que mas alla de la creacion de efectos muy interesantes pueden ser usados para otras cosas, como por ejemplo la seguridad al intentar encontrar caras o poder comprender las facciones de los rostros humanos.

[1]: images/get1.JPG
[2]: images/post.JPG
[3]: images/delete.JPG
[4]: images/get2.JPG
