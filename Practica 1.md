# Práctica 1 : Planificación de trayectorias carteasianas 

El objetivo de esta práctica es representar la trayectoria cartesiana de un manipulador robótico, la cual ha sido generada utilizando interpolación de la orientación mediante cuaterniones, basada en el método de Taylor. La planificación se encarga de generar las referencias de posición y orientación para el control del movimiento del manipulador, de forma que se obtiene una secuencia temporal de las diferentes posturas del efector final, desde un punto inicial hasta un punto final.

Además, se garantiza que durante el movimiento desde la postura inicial hasta la final no se saturen los actuadores, asegurando así un comportamiento dinámicamente factible del sistema.

**Funciones aportadas**

Para la siguinte práctica se hacen uso de las siguientes funciones:
* cartesian_planning: El codigo se encarga de realizar la simulación de la trayectoria del manipulador ABB IRB120
* zyz2tr : Se encarga de convertir el vector [ α ,β ,γ ] de ZYZ (ángulos de Euler) a una Matriz homogénea T.
* tr2zyz : Obtiene la representación [ α ,β ,γ ] de ZYZ (ángulos de Euler) a partir de una transformación T.
* tr2q   : Convierte la matriz homogénea T al cuaternión q.
* q2tr   : Calcula la matriz homoegenea T correspondiente al cuaternión q.

  
**Funciones incompletas**

Para esta práctica además se aportan una serie de funciones que se encuentran incompletas y que se han de completar.
* qpinter
* generate_smooth_path

**Cuaternión**

Un cuaternión es una representación matemática utilizada para describir rotaciones y orientaciones en el espacio tridimensional. Es especialmente útil en robótica, gráficos por computadora, navegación y control de movimiento, ya que permite interpolar rotaciones de forma suave.

En aplicaciones como el control de movimiento de manipuladores robóticos o sistemas de navegación inercial, los cuaterniones se emplean para representar orientaciones de manera estable y eficiente.
<p align="center">
  <img src="https://github.com/user-attachments/assets/6e9da30e-5140-41db-91ef-d540b4b17d67" alt="Trayectoria suave del sistema" width="250">
</p>
<p align="center"><em>Figura 1 :Rrepresentaciuón de Cuaternión </em></p>

Dados dos vectores 𝑢 y 𝑣 se puede construir un cuaternión 𝑞 tal que, al aplicarlo como operador rotacional, se alinee 
𝑢 con 𝑣. Este tipo de operación es fundamental en tareas de orientación o alineación de referencias espaciales.

Una propiedad importante de los cuaterniones es que, al aplicarlos sobre un vector, pueden provocar no solo una rotación sino también un escalado si su norma no es igual a uno. Para evitar este efecto no deseado, se utilizan cuaterniones unitarios, es decir, cuaterniones con norma igual a uno, los cuales garantizan que la magnitud del vector original no se vea alterada, realizándose únicamente una rotación.

**Interpolación Cartesiana** 

La interpolación cartesiana se caracteriza por lograr una variación lineal de la posición y la orientación; esta última utiliza la representación de la orientación mediante cuaterniones. Por lo tanto, al vincular dos desplazamientos rectilíneos, se produce una discontinuidad de velocidad en el punto de transición.

Para el caso de estudio de la práctica, se considera una trayectoria formada por dos desplazamientos: el primero va desde una ubicación P<sub>0</sub> hasta P<sub>1</sub>, y el segundo desde P<sub>1</sub> hasta P<sub>2</sub>. Para evitar las discontinuidades de velocidad que pueden surgir en el punto P<sub>1</sub> debido al cambio de trayectoria, se emplea una aceleración constante que permite ajustar la velocidad durante el desplazamiento. 

<p align="center">
  <img src="https://github.com/user-attachments/assets/e501f716-24d2-475a-8925-0c789489933c" alt="Trayectoria suave del sistema" width="500">
</p>
<p align="center"><em>Figura 2 : Diagrama de posición y velocidad </em></p>

En la tryectoria trazada, se realiza un ajuste gradual en la velocidad  durante un intervalo desde un instante de tiempo desde −τ (antes de alcanzar P<sub>1</sub>) hasta τ (después de alcanzar P<sub>1</sub>). La velocidad en este intervalo cambiará linealmente la velocidad desde ΔP<sub>1</sub> / T<sub>1</sub> hasta ΔP<sub>2</sub> / T<sub>2</sub>.  Para hacer el cambio de trayectoria más suave, se necesita encontrar una función matemática que describa como debe camabiar la posicion en el intervalo [−τ , τ]. Dicha función es una parabola o una funcinión cuadratica. 

Aplicando las condiciones de borde en ambos extremos del segmento y definiendo la aceleración en el área, la posición se obtiene como:

  **p(t)** = P<sub>1</sub> − ((τ − t)<sup>2</sup> / 4τT<sub>1</sub>) · ΔP<sub>1</sub> + ((τ + t)<sup>2</sup> / 4τT<sub>2</sub>) · ΔP<sub>2</sub>

Y la orientación como:

  **q(t)** = q<sub>1</sub> · q[−((τ − t)<sup>2</sup> / 4τT<sub>1</sub>) · θ<sub>1</sub>, n̂<sub>1</sub>] · q[((τ + t)<sup>2</sup> / 4τT<sub>2</sub>) · θ<sub>2</sub>, n̂<sub>2</sub>]   


## Apartado 1
Para el primer apartado se pide completar la función qpinter, esta debe ralizar la interpolación de cuaterniones basandose en el métdodo de Tayler que calcula el cuaternión intermedio entre el punto inicial y el final. El valor del parametro lambda debe de satisfacer  0≤la≤1. 

Para obtener la interpolación,  primero se obtienen las posiciones de las matrices homogeneas, para realizar una interpolación lineal entre p1 y p2.

$pr = (1-lambda)⋅p1+lambda⋅p2 = p1 + lambda⋅(p2-p1)$

Con las funciones tr2q se puede obtener el valor de los cuaterniones nomralizados de las matrices homogeneas. Los cuaterniones obtenidso 𝑞1 y 𝑞2 representan las orientaciones de las poses P1 y P2. En la interpolación para obtener el camino más corto en el espacio articular, se verifica si el producto entre 𝑞1 y 𝑞2 es negativo.

Luego se calcula la rotación relativa  entre los cuaterniones mediante el producto (función qqmul) y a partir de esta rotación se obtiene el eje y ángulo de rotación. Con esto podemos generar un cuternión de rotación intermedio. El cuaternión interpolado 𝑞𝑟 se obtiene aplicando esta rotación intermedia al cuaternión original 𝑞1, dando como resultado una orientación suavemente interpolada entre P1 y P2.

El codigo desarrollado es el siguinte: 

        function [pr,qr] = qpinter(P1, P2, t)
            % Posición
            p1 = P1(1:3,4);
            p2 = P2(1:3,4);
            lambda = t; % Asumimos t ∈ [0,1]
            pr = p1 + lambda * (p2 - p1);
        % Cuaterniones
            q1 = tr2q(P1);
            q2 = tr2q(P2);
            q1 = q1 / norm(q1);
            q2 = q2 / norm(q2);
        
            % Asegura el camino corto
        if dot(q1, q2) < 0
            q2 = -q2;
        end
            inv_q1 = [q1(1), -q1(2:4)];
            q = qqmul(inv_q1, q2);
        
            theta = 2 * acos(q(1));
            if abs(sin(theta/2)) < 1e-6
                u = [0 0 0];
            else
                u = q(2:4) / sin(theta / 2);
            end
        
            q_giro = [cos(lambda * theta / 2), u * sin(lambda * theta / 2)];
            qr = qqmul(q1, q_giro);
        end 


## Apartado 2

Para el segundp apartado de la ptactica se pide completar la función **generate_smooth_path** la cual debe de calcular la transformaciçon correspondiente al moviemiento de las diferentes trayectorias desde P<sub>0</sub>  a P<sub>2</sub>, pasando por el punto intermedio P<sub>1</sub>. Como se comentado antes el paso por el punto P<sub>1</sub>  debe de estar suavizado por el método de Taylor. En los parametros de entrada de la funcion, τ y T deben de corresponder al intervalo de transición y al timepo total utilizado para recorrer el camino. 

Para trazar el trayectoria deseada, primero se obtienen las posiciones y lo cuaterniones normalizados de cada uno de los puntos de interes de la trayectoria. Una vez obtenido se obtiene el incremento de posición entre los puntos P<sub>0</sub> y P<sub>1</sub> y P<sub>1</sub> y P<sub>2</sub>. Para el primer tramo de la tryactoria se establece un rango de tiempo de (t <= -tau), antes del tramo de la trayectoria que se suaviza. 

Para ello se ha desarrollado el siguinte codigo:

    function [P, Q] = generate_smooth_path(P0, P1, P2, tau, T, t)
        % Calcula la posición P (1x3) y orientación Q (cuaternión [w x y z])
        % suavizando la trayectoria entre P0, P1 y P2 usando el método de Taylor
    
        % Extraer posiciones y cuaterniones
        p0 = P0(1:3,4)';
        p1 = P1(1:3,4)';
        p2 = P2(1:3,4)';
    
        q0 = tr2q(P0);
        q1 = tr2q(P1);
        q2 = tr2q(P2);
    
        % Normalizar cuaterniones
        q0 = q0 / norm(q0);
        q1 = q1 / norm(q1);
        q2 = q2 / norm(q2);
    
        dP1 = p1 - p0;
        dP2 = p2 - p1;
    
        if (t < -T || t > T)
            disp('Parameter t out of range');
            P = [NaN NaN NaN];
            Q = [1 0 0 0];
            return;
        end
    
        if (t <= -tau)
            % Primer segmento: P0 → P1 (lineal)
            %lambda = (t + T) / (T - tau);
            lambda = (t + T) /T;
            lambda = max(0, min(1, lambda));  % Asegurar que lambda ∈ [0,1]
            [P, Q] = qpinter(P0, P1, lambda);
    
        elseif (t >= tau)
            % Tercer segmento: P1 → P2 (lineal)
            %lambda2 = (t - tau) / (T - tau);
            lambda2 = (t) / (T);
            lambda2 = max(0, min(1, lambda2));
            [P, Q] = qpinter(P1, P2, lambda2);
    
        else
            % Segmento intermedio (suavizado entre P0, P1, P2)
    
            % Posición (Taylor)
            P = p1 - dP1 * (tau - t)^2 / (4 * tau * T) + dP2 * (tau + t)^2 / (4 * tau * T);
    
            % Orientación (Taylor con cuaterniones)
            inv_q0 = [q0(1), -q0(2:4)];
            inv_q1 = [q1(1), -q1(2:4)];
    
            q_rot1 = qqmul(inv_q0, q1);
            q_rot2 = qqmul(inv_q1, q2);
    
            % Extraer ángulos y ejes de rotación
            theta1 = 2 * acos(min(max(q_rot1(1), -1), 1));
            theta2 = 2 * acos(min(max(q_rot2(1), -1), 1));
    
            if abs(sin(theta1/2)) < 1e-6
                u1 = [0 0 0];
            else
                u1 = q_rot1(2:4) / sin(theta1 / 2);
            end
    
            if abs(sin(theta2/2)) < 1e-6
                u2 = [0 0 0];
            else
                u2 = q_rot2(2:4) / sin(theta2 / 2);
            end
    
            angulo1 = -theta1 * (tau - t)^2 / (4 * tau * T);
            angulo2 =  theta2 * (tau + t)^2 / (4 * tau * T);
    
            q_giro1 = [cos(angulo1/2), u1 * sin(angulo1/2)];
            q_giro2 = [cos(angulo2/2), u2 * sin(angulo2/2)];
    
            q = qqmul(qqmul(q1, q_giro1), q_giro2);
            Q = q / norm(q);
        end
    end


 
