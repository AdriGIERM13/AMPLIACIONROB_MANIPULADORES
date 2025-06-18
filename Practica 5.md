# Práctica 5 : Control de fuerza

## Introducción 
  En la última práctica de Manipuladores, se pretende desarrollar un sistema de control de fuerza con una realimentación interna de posición, 
con lo que se puede llegar a obtener la fuerza aplicada en el efector final deseada.Dicho comportamiento dinámico del robot se linealiza con 
la ley de control utilizada en las prácticas anteriores.
  
$$ \boldsymbol{\tau} = \mathbf{M}(\mathbf{q}) \cdot \ddot{\mathbf{q}}d + {\mathbf{C} (\mathbf{q}, 
\dot{\mathbf{q}}) \cdot \dot{\mathbf{q}} + \mathbf{F}b\cdot \dot{\mathbf{q}} + \mathbf{g}}{\mathbf{n}(\mathbf{q}, 
\dot{\mathbf{q}})} + \mathbf{J}^T(\mathbf{q})\mathbf{f}{e} $$

Para esta práctica se consideran posicones cartesianas y se asume que el entonto presenta un comportamiento elástico la cual la podemos modelar 
mediante la siguinte expresión. Donde $x_e$ representa la posición cartesiana del amnipualdor y $x_r$ representa el punto de equilibrio. 

$$
F_{mi} = \mathbf{K}_P \cdot (x_F - x_e)
$$

La dinámica del sistema del manipulador se describe mediante la siguiente expresión, donde $x_e$ representa la posición en régimen estacionario, 
la cual debe alcanzar la posición de consigna $x_d$ .Esta posición de consigna se define a través de una ley de control orientada al seguimiento de fuerza.
Para implementar dicho control, se propone utilizar un controlador proporcional, a partir del cual se puede obtener la siguiente deducción.

$$
\mathbf{M}_d \ddot{x}_e + \mathbf{K}_D \dot{x}_e + \mathbf{K}_P x_e = \mathbf{K}_P x_F
$$

Donde el controlador proporcional vine definido por :

$$
x_F = \mathbf{C}_f \cdot (f_d - f_e)
$$

La expresión resultante viene definida por:

$$
\mathbf{M}_d \ddot{x}_e + \mathbf{K}_D \dot{x}_e + \mathbf{K}_P (\mathbf{I}_3 + \mathbf{C}_F \mathbf{K}) x_e = \mathbf{K}_P \mathbf{C}_F (\mathbf{K} x_r + f_d)
$$


Dicha expresión representa un sistema de tipo 0 para el control de fuerza, por tanto presenta un error en regimen permanente frente a una consigna de entrada. Por lo que para solucinarlo, se implementa en $C_f$ un integrador y obtener un sistema de tipo 1. 

$$
\mathbf{C}_F = \mathbf{K}_F + \mathbf{K}_I \int_0^t (\cdot)\ d\zeta
$$

El diagrama equivalente dek sistema por tanto quedaría: 
<p align="center">
  <img src="https://github.com/user-attachments/assets/c3088f1c-52bc-4823-ba9c-30e5eab8b20e" alt="image" width="600"><br>
  <em>Figura 1. Diagrama de bloques equivalente del sistema .</em>
</p>

## Simulación con controlador P
Para la simualcion del sistema que implementa un controlador P, se ha desarrollado el sisguinte sistema de bloques en Matlab-Simulink.

<p align="center">
  <img src="https://github.com/user-attachments/assets/fc9703bd-5a6b-4d12-aad4-3c0ef06d1d49" alt="image" width="600"><br>
  <em>Figura 2. Diagrama de bloques en Simulink con controlador P .</em>
</p>

Donde el bloque de control de posicion viene formado por: 
<p align="center">
  <img src="https://github.com/user-attachments/assets/ba23b054-93ca-4ad9-b37f-c486df6febbe" alt="image" width="600"><br>
  <em>Figura 3. Bloque de control de movimiento .</em>
</p>

El sistema parte de una posición incial de $x_{e_inicial}=[1.3,0.7]$ y donde los parametros del sistema son los siguintes:

* Constante K del sensor:    $K=[1000, 0; 0, 1000] $
* Controlador $C_F$  :        $C_f=[0.05, 0; 0,0] $
* Matriz de inercia $M_d$  :   $M_d=[1000, 0; 0,1000] $
* Ganancia derivativa $K_d$ :   $K_d=[5000, 0; 0,5000] $
* Ganancia derivativa $K_d$  :   $K_d=[5000, 0; 0,5000] $

En la simualción obtenemos la siguinte comportamiento:
*Posición*

![image](https://github.com/user-attachments/assets/128d2d59-f848-4e34-9839-71137aa38b75)

<p align="center">
  <img src="(https://github.com/user-attachments/assets/6c18d715-c3a9-421d-9e79-f1172d3819f7" alt="image" width="600"><br>
  <em>Figura 3. Bloque de control de movimiento .</em>
</p>

