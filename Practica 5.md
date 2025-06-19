# Práctica 5 : Control de fuerza

## Introducción 
  En la última práctica de Manipuladores, se pretende desarrollar un sistema de control de fuerza con una realimentación interna de posición, 
con lo que se puede llegar a obtener la fuerza aplicada en el efector final deseada.Dicho comportamiento dinámico del robot se linealiza con 
la ley de control utilizada en las prácticas anteriores.
  
 **Ec. 1 :**
 
 $$ \boldsymbol{\tau} = \mathbf{M}(\mathbf{q}) \cdot \ddot{\mathbf{q}}d + {\mathbf{C} (\mathbf{q}, 
\dot{\mathbf{q}}) \cdot \dot{\mathbf{q}} + \mathbf{F}b\cdot \dot{\mathbf{q}} + \mathbf{g}}{\mathbf{n}(\mathbf{q}, 
\dot{\mathbf{q}})} + \mathbf{J}^T(\mathbf{q})\mathbf{f}{e} $$

Para esta práctica se consideran posiciones cartesianas y se asume que el entorno presenta un comportamiento elástico la cual la podemos modelar 
mediante la siguiente expresión. Donde $x_e$ representa la posición cartesiana del manipulador y $x_r$ representa el punto de equilibrio. 

 **Ec. 2 :**

$$
F_{mi} = \mathbf{K}_P \cdot (x_F - x_e)
$$

La dinámica del sistema del manipulador se describe mediante la siguiente expresión, donde $x_e$ representa la posición en régimen estacionario, 
la cual debe alcanzar la posición de consigna $x_d$ .Esta posición de consigna se define a través de una ley de control orientada al seguimiento de fuerza.
Para implementar dicho control, se propone utilizar un controlador proporcional, a partir del cual se puede obtener la siguiente deducción.

 **Ec. 3 :**
 
$$
\mathbf{M}_d \ddot{x}_e + \mathbf{K}_D \dot{x}_e + \mathbf{K}_P x_e = \mathbf{K}_P x_F
$$

Donde el controlador proporcional viene definido por :

 **Ec. 4 :**
 
$$
x_F = \mathbf{C}_f \cdot (f_d - f_e)
$$

La expresión resultante viene definida por:

 **Ec. 5 :**
 
$$
\mathbf{M}_d \ddot{x}_e + \mathbf{K}_D \dot{x}_e + \mathbf{K}_P (\mathbf{I}_3 + \mathbf{C}_F \mathbf{K}) x_e = \mathbf{K}_P \mathbf{C}_F (\mathbf{K} x_r + f_d)
$$


Dicha expresión representa un sistema de tipo 0 para el control de fuerza, por tanto presenta un error en regimen permanente frente a una consigna de entrada. Por lo que para solucinarlo, se implementa en $C_f$ un integrador y obtener un sistema de tipo 1. 

 **Ec. 6 :**
 
$$
\mathbf{C}_F = \mathbf{K}_F + \mathbf{K}_I \int_0^t (\cdot)\ d\zeta
$$

El diagrama equivalente del sistema por tanto quedaría: 
<p align="center">
  <img src="https://github.com/user-attachments/assets/c3088f1c-52bc-4823-ba9c-30e5eab8b20e" alt="image" width="600"><br>
  <em>Figura 1. Diagrama de bloques equivalente del sistema .</em>
</p>

En el diagrama de bloques se pueden distinguir tres partes.
La primera corresponde al conjunto de bloques encargados de convertir el error de fuerza en términos de posición, utilizando la constante $C_f$ ​como la acción proporcional. Esta acción solo afecta a la dirección X, ya que existe una restricción impuesta por una pared.

La segunda parte corresponde al conjunto de bloques responsables del control de posición, el cual incluye un bucle interno que calcula la posición actual del manipulador.
Por último, la tercera parte se encarga del cálculo de la fuerza de contacto, con el fin de implementar el control externo de fuerzas.

## Simulación con controlador P
Para la simulación del sistema que implementa un controlador P, se ha desarrollado el sisguinte sistema de bloques en Matlab-Simulink.

<p align="center">
  <img src="https://github.com/user-attachments/assets/fc9703bd-5a6b-4d12-aad4-3c0ef06d1d49" alt="image" width="600"><br>
  <em>Figura 2. Diagrama de bloques en Simulink con controlador P .</em>
</p>

Donde el bloque de control de posicion viene formado por: 
<p align="center">
  <img src="https://github.com/user-attachments/assets/ba23b054-93ca-4ad9-b37f-c486df6febbe" alt="image" width="600"><br>
  <em>Figura 3. Bloque de control de movimiento .</em>
</p>

El sistema parte de una posición inicial de $x_{e_inicial}=[1.3,0.7]$ y donde los parámetros del sistema son los siguintes:

* Constante K del sensor:        $K=[1000, 0; 0, 1000] $
* Controlador $C_F$  :           $C_f=[0.05, 0; 0,0] $
* Matriz de inercia $M_d$  :     $M_d=[1000, 0; 0,1000] $
* Ganancia derivativa $K_d$ :    $K_d=[5000, 0; 0,5000] $
* Ganancia derivativa $K_d$  :   $K_d=[5000, 0; 0,5000] $

En la simulación obtenemos la siguinte comportamiento:
*Posición*
<p align="center">
  <img src="https://github.com/user-attachments/assets/f764cbb6-ba26-4cc6-8e5d-e5fbf0254c4e" alt="image" width="800"><br>
  <em>Figura 4. Variación de Posición  .</em>
</p>

Se puede observar que apenas existe error en la consecución de la posición en el eje X, con un pequeño error de aproximadamente 0.15 m. Sin embargo, la posición en el eje Y se mantiene en 0. Esto se debe a que no se aplica ninguna fuerza en la dirección del eje Y, por lo que el controlador recibe una consigna nula, estableciéndose así en dicha posición final.

*Fuerza*
<p align="center">
  <img src="https://github.com/user-attachments/assets/0e34613a-97b6-49ef-9781-15e4be4ffd7f" alt="image" width="800"><br>
  <em>Figura 5. Variación de Fuerza</em>
</p>

Como se puede observar en las fuerzas obtenidas a partir de la simulación, el sistema presenta una ganancia de aproximadamente -1.37 en estado estacionario, lo cual se aleja de la consigna establecida de [10 0] N. Esto indica la necesidad de implementar un controlador PI para eliminar el error en régimen permanente.
Otra propiedad que se puede analizar a través de la simulación es que el sistema presenta un tiempo de establecimiento de 2.5 segundos.

*Concluiones*

La aplicación de un controlador proporcional (P) en el control de fuerza del manipulador no permite lograr un control adecuado del sistema. Esto se debe a que, al no incorporar una acción integral, se genera un error en estado estacionario.
Por otro lado, dado que el controlador solo aplica una ganancia en el eje X, la consigna en el eje Y se establece como nula, lo que provoca que el manipulador no ejerza ninguna fuerza en esa dirección. 

## Simulación con controlador PI
del sistema que implementa un controlador PI, se ha desarrollado el sisguinte sistema de bloques en Matlab-Simulink.

<p align="center">
  <img src="https://github.com/user-attachments/assets/54fef522-22ea-4351-9af6-0b73b65cd95f" alt="image" width="600"><br>
  <em>Figura 6. Diagrama de bloques en Simulink con controlador PI .</em>
</p>

Donde el bloque de controlador PI está formado por el siguinte cojunto de bloques: 

<p align="center">
  <img src="https://github.com/user-attachments/assets/3be5a4db-83cd-4927-a83b-ba0e5047f0eb" alt="image" width="200"><br>
  <em>Figura 7. Diagrama de bloques del bloque d control PI .</em>
</p>

Donde los bloques están formados por:

* Control proporcional $K_p$:  $K_p=[0.03, 0; 0, 0] $
* Control integral $K_i$       $K_i=[0.03, 0; 0, 0] $

En la simulación obtenemos la siguinte comportamiento:
*Posición*
<p align="center">
  <img src="https://github.com/user-attachments/assets/fc62962d-6e6b-4e16-a730-1aaf8c38378f" alt="image" width="800"><br>
  <em>Figura 8. Variación de Posición  .</em>
</p>

Con la incorporación del controlador PI, se ha logrado reducir el error en régimen estacionario que se presentaba con el uso del controlador proporcional.
Sin embargo, al igual que en el caso del controlador proporcional, no se aplica una ganancia en el eje Y, por lo que la posición en dicho eje sigue siendo nula.

*Fuerza*
<p align="center">
  <img src="https://github.com/user-attachments/assets/5810214b-835e-431d-b6a5-9086e8bb2082" alt="image" width="800"><br>
  <em>Figura 9. Variación de Fuerza  .</em>
</p>

Como se puede observar, se ha logrado reducir el error en estado estacionario, alcanzando una ganancia unitaria. Sin embargo, el sistema presenta un tiempo de establecimiento de 5.4 segundos, lo que indica que el control mediante un controlador PI es más lento en comparación con el controlador proporcional.

*Conclusión*
Implementando el controlador PI se ha logrado mejorar el control de fuerza del manipulador, obteniendo un error nulo ante una entrada escalón, no obstante se empeora la velocidad de respuesta, generando un sistema más lento. Si nos fijamos existe una diferencia de tiempo entre el establecimiento de la posición y de la fuerza. 

*Mejoras*
Para poder mejorar la respuesta del sistema será necesario ajustar las ganancias del controlador PI. Para ello vamos a reducir el valor de las ganancias del controlador PI. Para ver cómo mejora la respuesta se hará un estudio con las nuevas ganancias.

Nuevas ganancias del controlador:

* Control proporcional $K_p$:  $K_p=[0.015, 0; 0, 0] $
* Control integral $K_i$       $K_i=[0.02, 0; 0, 0] $

Donde la simulación con las nuevas ganancias del controlador otorga el siguiente comportamiento al sistema:

*Posición*
<p align="center">
  <img src="https://github.com/user-attachments/assets/e8943f0e-943a-4de8-9afd-10edd128c715" alt="image" width="800"><br>
  <em>Figura 8. Variación de Posición  .</em>
</p>

*Fuerza*
<p align="center">
  <img src="https://github.com/user-attachments/assets/41fc9f4b-a94c-454f-bdf5-32e91d6fe8a4" alt="image" width="800"><br>
  <em>Figura 9. Variación de Fuerza  .</em>
</p>

Como se puede observar, el ajuste del controlador PI permite que la ganancia del sistema sea unitaria y que el error en régimen permanente ante una entrada escalón sea nulo.
Además, esta mejora en el controlador reduce el tiempo de establecimiento a 4.4 segundos, lo que representa una mejora en la respuesta del sistema en comparación con el caso anterior.
