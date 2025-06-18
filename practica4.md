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

El video del experimento 1, aplicando fuerzas al robot con el control de impedancia se puede ver en la carpeta videos/practica4/EXPERIMENT01. 

*NOTA: el video ha sido grabado con el móvil ya que el ordenador no era capaz de grabar la pantalla correctamente con tantos nodos en funcionamiento.

*--PREGUNTAS--*

*¿Cuales son los efectos de cambiar los parametros de impedancia (M,B,K)?*

Cambiar estos valores tiene un impacto directo en el comportamiento del efector final:

 -M (matriz de masa virtual): Afecta a la inercia del sistema. Si aumentamos M, el brazo se comportará como si tuviese más masa, reaccionando mas lentamente a perturbaciones y cambios. Si, por otra parte, disminumimos M, la respuesta será más ágil y más sensible a ruidos y fuerzas pequeñas.

 - B (matriz de amortiguamiento): Controla cuánto se atenúan las oscilaciones/vibraciones. Si B es alto, el movimiento será más estable y suave, pero en consecuencia también más lento. Si B es bajo, el sistema responderá con mayor rapidez, pero puede generar unas oscilaciones mayores o inestabilidad.

 - K (matriz de rigidez): Define cuanto "empuja" el robot hacia la posición indicada. Si K es alta entonces el robot mantendrá meor su trayectoria frente a perturbaciones, aunque también puede provocar movimientos bruscos si hay errores. Si K es baja, el comportamiento será más flexible.

En conclusión, ajustar los parámetros de impedancia permite encontrar un equilibrio entre precisión, suavidad y robustez frente a perturbaciones externas. Estos parámetros dependerán de las necesidades de la tarea.

*¿Qué efectos tiene tener una "impedancia alta" en el eje X y una "impedancia baja" en el eje Y?*

Tener distintos niveles de impedancia en los ejes implica que el comportamiento del robot no será igual en todas las direcciones. 

Una alta impedancia en el eje X supone que en esa dirección el manipulador resistirá más cualquier desviación o fuerza externa en esa dirección y el movimiento en X será más preciso y rígido, manteniéndose cercano a la pose deseada.

Una baja impedancia en el eje Y implica que se comportará de manera mas flexible y permitirá mayor desplazamiento ante fuerzas externasa en esa dirección.

Esta combinación puede suponer movimientos indeseados en el eje de baja impedancia (Y), ya que al tener poca rigidez o amortiguamiento, se podrá mover más fácilmente en esa dirección ante pequeñas fuerzas o ruidos mientras que no ocurrirá en el eje X, lo que podría provocar inestabilidad.

*¿Las fuerzas aplicadas en el eje X generan movimiento en el eje Y, y viceversa?*

Sí, en ambos casos se produce esta situación.

*Explica por qué ocurre*

Las matrices de impedancia utilizadas (masa, amortiguamiento y rigidez) son estrictamente diagonales, lo que indica que, idealmente, el comportamiento en cada eje (X e Y) debería ser independiente. Este problema puede provenir de que existen acoplamientos dinámicos en el sistema debido a la estructura del robot lo cual hace que no se aisle con total precisión un eslabón de otro, generando fuerzas en un eje donde no se le aplica fuerza.

*¿Cómo se podría mitigar este efecto?*

Aunque el sistema utiliza matrices de impedancia diagonales, se observa acoplamiento entre ejes en la simulación. Este comportamiento podría mitigarse usando estrategias como pueden ser: mejorar el modelado del Jacobiano y su derivada, proyectar las fuerzas sobre los ejes deseados  o introducir compensaciones dinámicas. Además, podríamos ajustar los parámetros de impedancia con el objetivo de mejorar el aislamiento entre ejes y obtener un comportamiento más preciso del efector final.

*¿Hace el robot algún movimiento raro y por qué?*

Sí, como se puede ver en los vídeos "EXPERIMENTO2" y "EXPERIMENTO2_2", siempre que el robot trabaje dentro de su espacio operacional funcionará correctamente. Sin embargo, en posiciones muy lejanas (cuando la posición en X o Y está al máximo), el robot entra en un bloqueo donde ni llega a esa posición y se desconfigura ya que después no se puede publicar ninguna otra pose. Esto puede deberse a que al intentar alcanzar una posición pero no llegar, la secuencia en el código no detecta que se ha llegado a la posición deseada y por tanto no continúa procesandose.

### CONCLUSIÓN

En esta última práctica de ROS hemos logrado un control completo de nuestro brazo manipulador teniendo en cuenta todos los efectos que actúan sobre él (fuerza de gravedad, Coriolis) utilizando nodos como la cancelación de dinámica así como el controlador de impedancias. Además hemos comprobado mediante diversas herramientas el resultado de aplicar fuerzas en las direcciones X e Y en el brazo, y seleccionar una pose objetivo para que el brazo se coloque en la posición que deseemos.
