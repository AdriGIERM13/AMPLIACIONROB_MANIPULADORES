PRÁCTICA 2: Manipulator dynamics simulation.

Para comenzar con esta práctica, lo primero hay que hacer es configurar el entorno de ROS 2 Humble, así como crear nuestro repositorio y descargar los paquetes proporcionados por el profesor.

Una vez realizada la instalación y configuración, contamos con los siguiente:
- Un paquete uma_arm_description que entre otras cosas incluye un archivo launch.py que nos permitirá visualizar nuestro brazo manipulador gracias a la herramienta RViz.
- Un paquete uma_arm_control que cuenta con archivos .yaml para la configuración de parámetros, un archivo uma_arm_dynamics_launch.py para simular la dinámica del manipulador, y un archivo uma_arm_dynamics.cpp que contiene la implementación de las ecuaciones dinámicas y que modificaremos más adelante.

Subscripciones y Publicadores del Nodo
El nodo implementa dos mecanismos fundamentales de comunicación en ROS2: suscripciones a tópicos para recibir datos, y publicadores para enviar información al resto del sistema.

Subscripciones
joint_torques_subscription_
Esta suscripción está asociada al tópico joint_torques, el cual publica mensajes del tipo std_msgs::msg::Float64MultiArray. Estos mensajes contienen los valores de los torques aplicados en cada una de las articulaciones del manipulador. Cada vez que se recibe un nuevo mensaje, se ejecuta automáticamente la función de callback joint_torques_callback, encargada de procesar y almacenar los datos recibidos.

external_wrenches_subscription_
Similarmente, esta suscripción escucha el tópico external_wrenches, que emite mensajes del tipo geometry_msgs::msg::Wrench. Dichos mensajes representan fuerzas y torques externos aplicados sobre el manipulador. La función external_wrenches_callback se encarga de gestionar la información recibida para su posterior procesamiento o uso en cálculos de control.

Publicadores
publisher_acceleration_
Este publicador se encarga de emitir datos en el tópico joint_accelerations, utilizando mensajes del tipo std_msgs::msg::Float64MultiArray. A través de este canal, el nodo envía las aceleraciones calculadas para cada una de las articulaciones, lo que permite que otros nodos o sistemas puedan utilizarlas en tareas de simulación o control.

publisher_joint_state_
Por último, este publicador genera mensajes en el tópico joint_states, con datos estructurados según el tipo sensor_msgs::msg::JointState. Dichos mensajes proporcionan una descripción completa del estado del manipulador, incluyendo la posición, velocidad y esfuerzo de cada articulación. Esta información es fundamental para el seguimiento y supervisión del comportamiento del robot en tiempo real.

