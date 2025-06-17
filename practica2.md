# PRÁCTICA 2: Manipulator dynamics simulation.

Para comenzar con esta práctica, lo primero hay que hacer es configurar el entorno de ROS 2 Humble, así como crear nuestro repositorio y descargar los paquetes proporcionados por el profesor.

Una vez realizada la instalación y configuración, contamos con los siguiente:

Un paquete uma_arm_description que entre otras cosas incluye un archivo launch.py que nos permitirá visualizar nuestro brazo manipulador gracias a la herramienta RViz.
Un paquete uma_arm_control que cuenta con archivos .yaml para la configuración de parámetros, un archivo uma_arm_dynamics_launch.py para simular la dinámica del manipulador, y un archivo uma_arm_dynamics.cpp que contiene la implementación de las ecuaciones dinámicas y que modificaremos más adelante.

Subscripciones y Publicadores del Nodo:

Este nodo implementa suscripciones a topics para recibir datos y publicadores para enviar información al resto del sistema.

Subscripciones:
 - joint_torques_subscription_: Esta suscripción está asociada al topic joint_torques, el cual publica mensajes del tipo std_msgs::msg::Float64MultiArray. Estos mensajes contienen los valores de los torques aplicados en cada una de las articulaciones del manipulador. Cada vez que se recibe un nuevo mensaje, se ejecuta automáticamente la función de callback joint_torques_callback, que procesa y almacena los datos recibidos.

- external_wrenches_subscription_: esta suscripción lee el topic external_wrenches, que emite mensajes del tipo geometry_msgs::msg::Wrench. Representan fuerzas y torques externos aplicados sobre el manipulador. La función external_wrenches_callback se encarga de gestionar la información recibida para su posterior procesamiento o uso en cálculos de control.

Publicadores:

- publisher_acceleration_: Este publicador se encarga de emitir datos en el topic joint_accelerations, utilizando mensajes del tipo std_msgs::msg::Float64MultiArray. A través de este canal, el nodo envía las aceleraciones calculadas para cada una de las articulaciones, lo que permite que otros nodos o sistemas puedan utilizarlas en tareas de simulación o control.

- publisher_joint_state_: Por último, este publicador genera mensajes en el topic joint_states, con datos de tipo sensor_msgs::msg::JointState. Dichos mensajes proporcionan una descripción completa del estado del manipulador, incluyendo la posición, velocidad y esfuerzo de cada articulación. Esta información es fundamental para el seguimiento y supervisión del comportamiento del robot en tiempo real.

A continuación vamos a implementar la dinámica del manipulador, la cuál viene dada por la ecuación:

M(q) * q̈ + C(q,q̇) * q̇ + Fb * q̇ +g(q) = τ+τext (1)

donde:

q: Vector de posiciones articulares (joint_positions_).

q̇: Vector de velocidades articulares (joint_velocities_).

q̈: Vector de aceleraciones articulares (joint_accelerations_).

M(q): Matriz de inercia.

C(q, q̇): Matriz de fuerzas de Coriolis y centrífugas.

Fb: Matriz de fricción viscosa.

g: Vector de fuerzas debidas a la gravedad.

τ: Vector de torques articulares comandados (joint_torques_).

τext: Vector de torques articulares debidos a fuerzas externas.
​
Además, en nuestro caso, tenemos un manipulador de 2 grados de libertad.

Para calcular las aceleraciones, necesitamos calcular las matrices y vectores aplicando las formulaciones de Lagrange y de Newton Euler:

<p align="center">
  <img src="https://github.com/user-attachments/assets/eaa46905-3f2b-4996-b87d-5a36a4fc1b0c" alt="image" width="600"><br>
  <em>1. Ecuaciones para la matriz de inercia, matriz de fuerzas de Coriolis, matriz de fricción viscosa y vector de fuerzas gravitatorias</em>
</p>


La ecuación para calcular la ecuación es la siguiente:

q̈(k+1) = M⁻¹(q_k) * [ τ_k + τ_ext(k) - C(q_k, q̇(k)) * q̇(k) - Fb * q̇(k) - g(q_k) ]

Haciendo la integral de la aceleración obtengo la velocidad, y de igual forma integrando la velocidad obtengo la posición.

Al lanzar todos los nodos y ejectutar el rqt_graph nos queda de la siguente forma:

<p align="center">
  <img src="https://github.com/user-attachments/assets/ca4d9a38-58b3-4d4d-9b8e-bd1586e68cb8" alt="image" width="400"><br>
<em> 2. Conexión de nodos mediante rqt_graph</em>
</p>

Vamos a realizar la simulación y capturar los resultados utilizando plotjuggler.

En primer lugar utilizamos los parámetros por defecto que se pueden observar en la imagen 3:

<p align="center">
  <img src="https://github.com/user-attachments/assets/5227fa31-345c-4989-88de-7ede906589d8" alt="image" width="400"><br>
<em> 3. Parámetros iniciales</em>
</p>

El resultado de la simulación con estos parámetros se observa en la figura 4: 

<p align="center">
  <img src="https://github.com/user-attachments/assets/5e8c8345-71e5-4041-833c-d171125e4963" alt="image" width="400"><br>
<em> 3. Resultados de la simulación</em>
</p>

*--PREGUNTAS--*

¿Cuales son los efectos de modificar los parámetros de la dinámica del brazo?

Los parámetros que podemos modificar en el archivo dynamics_params.yaml son los siguientes:

- m1: Masa del primer eslabón del manipulador.

- m2: Masa del segundo eslabón del manipulador.

- l1: Longitud del primer eslabón del manipulador.

- l2: Longitud del segundo eslabón del manipulador.

- b1: Coeficiente de amortiguamiento de la primera articulación.

- b2: Coeficiente de amortiguamiento de la segunda articulación.

- g: Aceleración debida a la gravedad.

En esta simulación el manipulador está sometido únicamente a la fuerza de la gravedad, por lo que si modificmoos los parámetros ocurrirá lo siguiente: 

Masa de los eslabones (m1 y m2):

Al aumentar la masa, la fuerza gravitatoria sobre cada eslabón será mayor, por lo que el brazo tenderá a caer o desplazarse con más rapidez y fuerza debido al peso aumentado. Por otro lado, reduciendo la masa, el efecto de la gravedad será menor, y el brazo caerá más lentamente y con menos fuerza.

Longitud de los eslabones (l1 y l2):

Cambiar la longitud afecta el momento gravitacional. Si el eslabón es más largo genera un mayor par (torque) debido a la gravedad, porque el peso actúa a mayor distancia del eje de rotación. Esto puede hacer que el brazo se mueva más rápido o con mayor esfuerzo bajo su propio peso. Si acortamos los eslabones, el torque gravitacional será menor.

Coeficientes de amortiguamiento (b1 y b2):

El amortiguamiento no cambia la fuerza gravitatoria, pero sí afecta cómo se mueve el brazo bajo esa fuerza. Un amortiguamiento alto frenará el movimiento, haciendo que el brazo descienda más lentamente y sin oscilar. Un amortiguamiento bajo permitirá movimientos más rápidos y posibles oscilaciones.

Aceleración gravitatoria (g):

Como es lógico, al cambiar este valor modifica directamente la intensidad de la fuerza de gravedad. A mayor gravedad, mayor será la fuerza que actúa sobre el manipulador, aumentando el torque y la velocidad con la que se mueven los eslabones. Si disminuimos este valor, el efecto será el contrario, el brazo tardará más en caer.

El ejemplo de la simulación se encuentra en la carpeta videos con el nombre "PRACTICA2_PARAMETROS_CAMBIADOS"

Para realizar la simulación hemos aumentado tanto las masas de los eslabones m1 y m2 así como hemos aumentado los coeficientes de amortiguamiento. El resultado es que el brazo comienza con una aceleración mayor debido al aumento del peso de los eslabones, los cuáles están sometidos a las fuerzas gravitatorias. Por otra parte, se observa que las oscilaciones son un poco más lentas y tarda algo menos en llegar a la posición de reposo. 
