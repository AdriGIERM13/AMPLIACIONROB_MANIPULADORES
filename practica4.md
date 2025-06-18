# PRÁCTICA 4: IMPEDANCE CONTROL

Para esta última práctica de ROS, implementaremos un controlador de impedancia (Figura 1).

<p align="center">
  <img src="https://github.com/user-attachments/assets/79e9dc34-6cf1-4fd4-b17c-b745a22cb481" alt="image" width="600"><br>
  <em>Figura 1.Esquema controlador de impedancia completo.</em>
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

Para implementar este método se ha utilizado la fórmula de la figura 2. En la figura 3 se puede ver el código resultante.

<p align="center">
  <img src="https://github.com/user-attachments/assets/9c078de4-e3f8-478f-8563-2425b0c417c6" alt="image" width="200"><br>
  <em>Figura 2.Fórmula para el cálculo de la posición.</em>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/6b605f58-e11f-4989-8e63-022e61ab9a85" alt="image" width="700"><br>
  <em>Figura 3.Código del método para calcular cinemática directa.</em>
</p>

#### Cálculo del jacobiano

Una vez tenemos estos datos, el siguiente paso es calcular el Jacobiano J(q). Esta matriz relaciona las velocidades articulares (q̇) con las velocidades cartesianas del efector final (ẋ). 

Vamos a implementar el método update_jacobians siguiendo las fórmulas de la figura 4. El código de este método se observa en la figura 5.

<p align="center">
  <img src="https://github.com/user-attachments/assets/320064a7-8286-4dfb-9b6a-7cb17cdda2d4" alt="image" width="400"><br>
  <em>Figura 4.Fórmula para el cálculo de jacobianos.</em>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/e3e9f6e5-5c85-482a-bf99-cfee300f7bd4" alt="image" width="700"><br>
  <em>Figura 5. Código del método para calcular jacobianos.</em>
</p>

#### Velocidades cartesianas

Una vez que se tiene J(q), el controlador puede calcular las velocidades cartesianas actuales del efector final (ẋ) mediante la cinemática diferencial de primer orden:

ẋ=𝐽(𝑞)⋅q̇ (1)

Este paso permite conocer cómo se está moviendo el manipulador en ese instante.
En la figura 6 se muestra el código con la ecuación 1 implementada.

<p align="center">
  <img src="https://github.com/user-attachments/assets/baf97b4d-0a9d-4232-8c1e-626784ae2cb8" alt="image" width="700"><br>
  <em>Figura 6. Código del método para calcular velocidades cartesianas.</em>
</p>

