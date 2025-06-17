PRÁCTICA 2: Manipulator dynamics simulation.

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

M(q) * q̈ + C(q,q̇) * q̇ + Fb * q̇ +g(q) = τ+τext

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

![image](https://github.com/user-attachments/assets/eaa46905-3f2b-4996-b87d-5a36a4fc1b0c)

La ecuación para calcular la ecuación es la siguiente:



