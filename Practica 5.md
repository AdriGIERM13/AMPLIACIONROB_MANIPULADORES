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

