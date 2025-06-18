# PRÁCTICA 4: IMPEDANCE CONTROL

Para esta última práctica de ROS, implementaremos un controlador de impedancia (Figura 1).

![image](https://github.com/user-attachments/assets/79e9dc34-6cf1-4fd4-b17c-b745a22cb481)


En primer lugar tenemos la compensación dinámica realizada en la práctica 3. A este esquema se le añade el controlador de impedancia cartesiano. Ya que este controlador actúa en el espacio operacional, es necesario realizar transformaciones entre las variables articulares y cartesianas. 
Para ello, usamos el modelo cinemático del robot junto con las derivadas de primer y segundo orden de la cinemática, conocidas como cinemática diferencial.

Para implementar este controlador, se ha creado un nuevo nodo llamado impedance_controller.cpp.

Este nodo se suscribe a los siguientes topics: 

Pose de equilibrio deseada: equilibrium_pose (xd).
Este tópico proporciona la posición y orientación final que el efector final del robot debe alcanzar.

Estados articulares actuales: joint_states (q, q̇).
Aquí se obtienen las posiciones articulares actuales del robot (q) y sus velocidades articulares (q̇), necesarias para transformar a variables cartesianas y calcular las aceleraciones deseadas.

Fuerzas externas: external_wrenches (f_ext).
Este tópico contiene las fuerzas y torques que actúan sobre el efector final, normalmente provenientes de sensores de fuerza/par.

El nodo también utiliza los parámetros de impedancia, los cuales son leídos desde un archivo de configuración llamado impedance_params.yaml. Este archivo contiene las siguientes matrices, que modelan el comportamiento dinámico del controlador:

mass_matrix_ (M) – matriz de masa virtual

damping_matrix_ (B) – matriz de amortiguamiento

stiffness_matrix_ (K) – matriz de rigidez

Estas matrices definen cómo reacciona el robot a desviaciones entre la posición actual y la deseada, así como a perturbaciones externas.

### Funcionamiento
En primer lugar el controlador verifica que haya recibido correctamente los tres datos principales:

- joint_states (estados articulares),

- external_wrenches (fuerzas externas),

- equilibrium_pose (pose deseada del efector final).

#### Cinemática directa
Una vez que se validan los datos, el controlador utiliza el método forward_kinematics() para calcular la cinemática directa, es decir, obtiene la posición y orientación actual del efector final (x) a partir del estado articular (q).

Esto es fundamental para saber dónde se encuentra el robot en el espacio cartesiano y poder comparar esa posición con la deseada (xd).
