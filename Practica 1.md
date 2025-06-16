# Práctica 1 : Planificación de trayectorias carteasianas 

El objetivo de esta práctica es representar la trayectoria cartesiana de un manipulador robótico, la cual ha sido generada utilizando interpolación de la orientación mediante cuaterniones, basada en el método de Taylor. La planificación se encarga de generar las referencias de posición y orientación para el control del movimiento del manipulador, de forma que se obtiene una secuencia temporal de las diferentes posturas del efector final, desde un punto inicial hasta un punto final.

Además, se garantiza que durante el movimiento desde la postura inicial hasta la final no se saturen los actuadores, asegurando así un comportamiento dinámicamente factible del sistema.

**Funciones aportadas**
Para la siguinte práctica se hacen uso de las siguientes funciones:
* cartesian_planning: El codigo se encarga de realizar la simulación de la trayectoria del manipulador ABB IRB120
* zyz2tr : Se encarga de convertir el vector [ α ,β ,γ ] de ZYZ (ángulos de Euler) a una Matriz homogénea T.
* tr2zyz : Obtiene la representación [ α ,β ,γ ] de ZYZ (ángulos de Euler) a partir de una transformación T.
* tr2q   : Convierte la matriz homogénea T al cuaternión q.
* tr2q   : Calcula la matriz homoegenea T correspondiente al cuaternión q.

  
**Funciones incompletas**

Para esta práctica además se aportan una serie de funciones que se encuentran incompletas y que se han de completar.
* qpinter
* generate_smooth_path

## Cuaternión
Un cuaternión es una representación matemática utilizada para describir rotaciones y orientaciones en el espacio tridimensional. Es especialmente útil en robótica, gráficos por computadora, navegación y control de movimiento, ya que permite interpolar rotaciones de forma suave y sin ambigüedades como el bloqueo de gimbal (gimbal lock).

En aplicaciones como el control de movimiento de manipuladores robóticos o sistemas de navegación inercial, los cuaterniones se emplean para representar orientaciones de manera estable y eficiente.

Dados dos vectores 𝑢 y 𝑣 se puede construir un cuaternión 𝑞 tal que, al aplicarlo como operador rotacional, se alinee 
𝑢 con 𝑣. Este tipo de operación es fundamental en tareas de orientación o alineación de referencias espaciales.

Una propiedad importante de los cuaterniones es que, al aplicarlos sobre un vector, pueden provocar no solo una rotación sino también un escalado si su norma no es igual a uno. Para evitar este efecto no deseado, se utilizan cuaterniones unitarios, es decir, cuaterniones con norma igual a uno, los cuales garantizan que la magnitud del vector original no se vea alterada, realizándose únicamente una rotación.

## Interpolación Cartesiana 
La interpolación cartesiana se caracteriza por lograr una variación lineal de la posición y la orientación; esta última utiliza la representación de la orientación mediante cuaterniones. Por lo tanto, al vincular dos desplazamientos rectilíneos, se produce una discontinuidad de velocidad en el punto de transición.
