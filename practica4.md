# PRÁCTICA 4: IMPEDANCE CONTROL

Para esta última práctica de ROS, implementaremos un controlador de impedancia (Figura 1).

<p align="center">
  <img src="https://github.com/user-attachments/assets/79e9dc34-6cf1-4fd4-b17c-b745a22cb481" alt="image" width="600"><br>
  <em>Figura 1. Esquema controlador de impedancia completo.</em>
</p>

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

Esto sirve para conocer la posición del robot en el espacio cartesiano y poder comparar esa posición con la deseada (xd).

Para implementar este método se ha utilizado la fórmula de la ecuación 1. En la figura 2 se puede ver el código resultante.

<p align="center">
  <img src="https://github.com/user-attachments/assets/9c078de4-e3f8-478f-8563-2425b0c417c6" alt="image" width="200"><br>
  <em>Ecuación 1. Fórmula para el cálculo de la posición.</em>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/6b605f58-e11f-4989-8e63-022e61ab9a85" alt="image" width="700"><br>
  <em>Figura 2. Código del método para calcular cinemática directa.</em>
</p>

#### Cálculo del jacobiano

Una vez tenemos estos datos, el siguiente paso es calcular el Jacobiano J(q). Esta matriz relaciona las velocidades articulares (q̇) con las velocidades cartesianas del efector final (ẋ). 

Vamos a implementar el método update_jacobians siguiendo las fórmulas de la ecuación 2. El código de este método se observa en la figura 3.

<p align="center">
  <img src="https://github.com/user-attachments/assets/320064a7-8286-4dfb-9b6a-7cb17cdda2d4" alt="image" width="400"><br>
  <em>Ecuación 2. Fórmula para el cálculo de jacobianos.</em>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/e3e9f6e5-5c85-482a-bf99-cfee300f7bd4" alt="image" width="700"><br>
  <em>Figura 3. Código del método para calcular jacobianos.</em>
</p>

#### Velocidades cartesianas

Una vez que se tiene J(q), el controlador puede calcular las velocidades cartesianas actuales del efector final (ẋ) mediante la cinemática diferencial de primer orden:

ẋ=𝐽(𝑞)⋅q̇ (3)

Este paso permite conocer cómo se está moviendo el manipulador en ese instante.
En la figura 4 se muestra el código con la ecuación 3 implementada.

<p align="center">
  <img src="https://github.com/user-attachments/assets/baf97b4d-0a9d-4232-8c1e-626784ae2cb8" alt="image" width="700"><br>
  <em>Figura 4. Código del método para calcular velocidades cartesianas.</em>
</p>

#### Aceleraciones cartesianas deseadas

Una vez que tenemos disponibles todos los datos necesarios, podemos calcular las aceleraciones cartesianas deseadas (ẍd) de acuerdo al comportamiento deseado del controlador de impedancia. Esto se realiza mediante el método impedance_controller():

Estas aceleraciones se obtienen siguiendo el modelo de impedancia de segundo orden, el cual  se puede ver en la ecuación 4 y representa el comportamiento mecánico deseado del sistema:

<p align="center">
  <img src="https://github.com/user-attachments/assets/780af940-fde5-4855-af2a-522488fa6a59" alt="image" width="150"><br>
  <em>Ecuación 4. Fórmula del comportamiento dinámico del sistema.</em>
</p>

donde: 

- M es la metriz de masa virtual
- B es la matriz de amortiguamiento
- K es la matriz de rigidez
- fext es la fuerza externa
- x̂ = x - xd es el error de posición

A partir de esta fórmula se puede despejar la aceleraciónd deseada ẍd tal y como se ve en la ecuación 5. En la figura 5 vemos el código que se ha implementado.

<p align="center">
  <img src="https://github.com/user-attachments/assets/3dad1cb1-733f-4a9d-a5fc-e3903cc1e662" alt="image" width="150"><br>
  <em>Ecuación 5. Fórmula cálculo de la aceleración deseada.</em>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/c005a7ba-fbbc-480b-9e83-2b9ffc3242b8" alt="image" width="700"><br>
  <em>Figura 5. Código del método para calcular aceleraciond deseada.</em>
</p>

### Aceleraciones articulares deseadas

Una vez que hemos obtenido las aceleraciones cartesianas deseadas, el siguiente paso es transformar estas aceleraciones al espacio articular, es decir, calcular las aceleraciones articulares deseadas (desired_joint_accelerations).

La cinemática diferencial de segundo orden se ha calculado de acuerdo a la ecuación 6.
El código implementado es el que se ve en la figura 6. 

<p align="center">
  <img src="https://github.com/user-attachments/assets/9883bb66-bf31-4779-b00b-b87c2fc441f7" alt="image" width="250"><br>
  <em>Ecuación 6. Fórmula cálculo de la aceleración articular.</em>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/863591a7-1105-42db-95c7-ec9d12808a2f" alt="image" width="500"><br>
  <em>Figura 6. Código del cálculo de la aceleración articular.</em>
</p>

El video del experimento 1, aplicando fuerzas al robot con el control de impedancia se puede ver en la carpeta videos/practica4/EXPERIMENT01. Recalcar que ha sido grabado con el móvil ya que el ordenador no era capaz de grabar la pantalla correctamente con tantos nodos en funcionamiento.
