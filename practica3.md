# PRÁCTICA 3: INVERSE DYNAMICS CONTROL

En esta práctica vamos a implementar dinámica inversa para compensar los efectos no lineales que pueden producirse en el manipulador.

## Compensación de la gravedad

Los efectos de la gravedad sobre el manipulador provocan que este caiga por su propio peso. Es por ello que es necesario que el controlador tenga en cuenta estos efectos a la hora de calcular los torques articulares que serán enviados a los actuadores.

El controlador más sencillo dentro de los controladores por dinámica inversa es el controlador de compensación gravitatoria, este se basa en generar unos torques de control que sean iguales a los torques producidos por la gravedad sobre el manipulador. De esta forma, se contrarresta el efecto del peso, manteniendo el manipulador en equilibrio o facilitando su control para otras tareas.

Matemáticamente, el torque que genera el controlador se define como:

𝜏 = 𝑔(𝑞) (1)

Donde:

τ es el vector de torques que se aplicará en las articulaciones.

g(q) es el vector de torques producidos por el efecto de la gravedad, en función de la configuración articular 𝑞

Para implementar este controlador, es necesario realizar los siguientes pasos:

Obtener la configuración actual del manipulador, es decir, las posiciones articulares 
𝑞.
Calcular el vector gravitatorio g(q) utilizando el modelo dinámico del manipulador.

Asignar el torque calculado a la señal de control que se enviará a los actuadores.

En la figura 1 se muestra el código para calcular este torque, que es igual que el vector gravitacional:

<p align="center">
  <img src="https://github.com/user-attachments/assets/4bec5c24-ae3b-4b96-92d3-02d6b66eaed6" alt="image" width="600"><br>
  <em>Figura 1.Código compensador de la gravedad.</em>
</p>

El gráfico de los nodos se puede observar en la figura 2.

<p align="center">
  <img src="https://github.com/user-attachments/assets/cfbd6155-8704-40b0-9890-d61e9e91f1d6" alt="image" width="600"><br>
  <em>Figura 2. Gráfico de nodos mediante rqt_graph.</em>
</p>

Como resultado, el brazo del manipulador se mantiene estático. 

### Simulación de fuerzas

Utilizando la herramienta wrench_trackbar_publisher.py incluida dentro del paquete uma_control podemos aplicar fuerzas en X e Y a través de un interfaz(en Z no porque el manipulador es de dos dimensiones). Esto se comprueba en el video "SIMULACION_APLICACION_FUERZAS" en la carpeta de videos.

*¿Cuál es el comportamiento del robot cuando aplico fuerzas virtuales?*

Como se puede ver al principio vídeo, al aplicarle fuerza postiva en la coordenada x, este se mueve en esta dirección, intentando llegar a un punto de reposo pero no de una manera fija y lineal, sino que va oscilando alrededor de esa posición. Al igual que aplicando fuerza negativa en la dirección Y, el brazo baja, pero no de una manera controlado, oscila de un lado a otro debido a las fuerzas no lineales que también están actuando en el brazo.

## Linealización por control dinámico inverso

Una vez implementado el controlador de compensación gravitatoria, es posible compensar la dinámica no lineal completa del manipulador utilizando linealización por realimentación.

De esta manera forzamos al manipulador a comportarse según una dinámica deseada, eliminando los efectos que producen su propia dinámica interna, como la inercia, las fuerzas de Coriolis, la fricción y la gravedad.

Para ello debemos calcular el torque no solo para contrarrestar la gravedad, sino para compensar toda la dinámica del sistema.

Es importante destacar que esta compensación completa solo funciona si el modelo dinámico utilizado es una representación exacta del manipulador real, lo cual nunca se cumple en la realidad. El esquema de compensación se puede ver en la figura 3.


<p align="center">
  <img src="https://github.com/user-attachments/assets/587aee69-7e18-44a7-a1d9-bde6a33bdac6" alt="image" width="600"><br>
  <em>Figura 3. Diagrama de compensación completa.</em>
</p>

Para realizar la compensación debemos crear el nodo dynamics_cancellation.cpp. En este programaremos los torques deseados en cancel_dynamics. El código completo se puede ver en la figura 4. En este código hemos inicializado todas las matrices y vectores que vamos a utilizar y hemos aplicado la fórmula de la ecuación 2 para calcular el torque.



<p align="center">
  <img src="https://github.com/user-attachments/assets/2c090886-408f-4908-b94c-fe1f035a9b9d" alt="image" width="600"><br>
  <em>Figura 4. Código compensación competa. Cálculo del torque.</em>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/6d79708a-88a6-4bbb-b25d-5581ca126b88" alt="image" width="600"><br>
  <em>Ecuación 2. Ecuación cálculo torque completa.</em>
</p>

Para comprobar que funciona correctamente se ha generado una trayectoria cúbica incluida en el paquete uma_arm_control. El resultado de la simulación en plotjuggler se puede ver en la figura 5.

<p align="center">
  <img src="https://github.com/user-attachments/assets/613d9f76-7534-4d17-b459-ad4ce7e7a20f" alt="image" width="600"><br>
  <em>Figura 5. Gráficas de simulación trayectoria cúbica en plotjuggler.</em>
</p>

*-- PREGUNTAS--*

*¿Qué ocurre si el modelo dinámico de compensación no es exactamente igual al modelo dinámico real del manipulador?*

A

Si el modelo dinámico utilizado para calcular la compensación de la gravedad no coincide exactamente con la dinámica real del manipulador, se producirán errores a la hora de calcular la compensación ya que las matrices M, C, y g tendrán valores distintos, y por tanto el brazo no podrá nunca mantener el equilibrio.

Vamos a comprobarlo modificando los parámetros en gravity compensation dentro de dynamics_params_yaml:

Los videos con los resultados de la simulaciones se pueden ver en la carpeta videos.

"PRACTICA3_1": En este caso vamos a modificar la masa y la longitud del primer eslabón de tal forma, según los cálculos realizados para el torque el brazo se eleva, llegando a una posición fija donde se mantiene estable.


https://github.com/user-attachments/assets/e4e5f097-2d02-4b52-94e0-59d683eec735

/a237da0c-efcc-4fd2-af9c-e7f79fd77865



"PRACTICA3_2": Aquí se ha probado a cambiar todos los parámetros, el resultado es el mismo, tiende a la estabilidad pero con ambos eslabones hacia arriba.


https://github.com/user-attachments/assets/35f66d16-b8ba-4edb-88af-00b3bc10c339

B

En el video "PRACTICA3_DYNAMICS" se muestra la simulación cambiando los parámetros de todas las variables en la parte de "dynamics_cancellation". Como resultado vemos que los eslabones del manipulador adoptan una posición fija rapidámente que es variable según los parámetros.


https://github.com/user-attachments/assets/e68be221-425c-4573-887c-8122aaa17d92

En la figura 7 también se muestra otro ejemplo de la posición de los brazos con otros parámetros.

<p align="center">
  <img src="https://github.com/user-attachments/assets/4a6207a1-f90f-43ca-b0bc-89d00e7ae9b7" alt="image" width="600"><br>
  <em>Figura 6.Posición manipulador con parámetros del video PRACTICA3_DYNAMICS.</em>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/67dd0ba2-9248-4168-ab6b-f788272b93d6" alt="image" width="600"><br>
  <em>Figuta 7.Posición manipulador con otros parámetros.</em>
</p>

*¿Cuál es el comportamiento del robot bajo el control de dinámica inversa cuando aplicamos fuerzas virtuales?*

Tal y como se puede observar en el video "Fuerzas_compensador_dinamico" en la carpeta videos/practica3, el brazo robot aplica esa fuerza sin ningún tipo de control, el manipulador comienza a oscilar presentando una gran inestabilidad en sus dos eslabones, llegando a un punto donde comienza a girar sin control. Esto se debe a que al aplicar la aceleración deseada  el controlador supone que proporciono directamente una trayectoria suficientemente suave y que el modelo dinámico es perfecto, sin fuerzas externas inesperadas. Falta el acoplamiento entre la fuerza virtual y la aceleración deseada. Si aplicamos una fuerza Fext ​en el espacio cartesiano sin un modelo de impedancia, esa fuerza no se traduce en una aceleración deseada incoherente. Es decir, el sistema no sabe cómo responder físicamente a esa fuerza.

### CONCLUSIÓN

En esta tercera sesión, hemos partido del brazo robot y le hemos añadido un compensador de la gravedad implementado a través del modo gravity_compensation.cpp. También hemos visualizado el comportamiento del manipulador frente a la aplicación de fuerzas virtuales tanto en el eje X como en el eje Y.
