PRÁCTICA 3:

*-- PREGUNTAS--*

¿Qué ocurre si el modelo dinámico de compensación no es exactamente igual al modelo dinámico real del manipulador?

Si el modelo dinámico utilizado para calcular la compensación no coincide exactamente con la dinámica real del manipulador, se producirán errores a la hora de calcular la compensación ya que las matrices M, C, y g tendrán valores distintos, y por tanto el brazo no podrá nunca mantener el equilibrio.

Vamos a comprobarlo modificando los parámetros en gravity compensation dentro de dynamics_params_yaml:

Los videos con los resultados de la simulaciones se pueden ver en la carpeta videos.

"PRACTICA3_1": En este caso vamos a modificar la masa y la longitud del primer eslabón de tal forma, según los cálculos realizados para el torque el brazo se eleva, llegando a una posición fija donde se mantiene estable.

"PRACTICA3_2": Aquí se ha probado a cambiar todos los parámetros, el resultado es el mismo, tiende a la estabilidad pero con ambos eslabones hacia arriba.

En el video "PRACTICA3_DYNAMICS" se muestra la simulación cambiando los parámetros de todas las variables en la parte de "dynamics_cancellation". Como resultado vemos que los eslabones del manipulador adoptan una posición fija rapidámente que es variable según los parámetros.

En la figura siguiente también se muestra otro ejemplo de la posición de los brazos con otros parámetros.

<p align="center">
  <img src="https://github.com/user-attachments/assets/4a6207a1-f90f-43ca-b0bc-89d00e7ae9b7" alt="image" width="600"><br>
  <em>1.Posición manipulador con parámetros del video PRACTICA3_DYNAMICS.</em>
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/67dd0ba2-9248-4168-ab6b-f788272b93d6" alt="image" width="600"><br>
  <em>1.Posición manipulador con otros parámetros.</em>
</p>

