# Práctica: Controlador de Máquina Expendedora

## Breve explicación del código

El programa está estructurado alrededor de una máquina de estados controlada mediante un switch(estadoActual), ya que en mi caso me ha parecido la forma más limpia y sostenible de gestionar un sistema donde cada fase es mutuamente exclusiva. 
Usar un switch en vez de varios if evita mezclar lógicas, hace que el flujo sea mucho más claro y permite añadir o modificar estados sin riesgo de romper otros comportamientos. 
Cada estado tiene su propio bloque bien definido (espera, lectura de temperatura, menú, preparación, retirada…), y al entrar en ellos hago uso de distintos FLAGS junto con temporizadores basados en micros() para ejecutar ciertas acciones solo una vez. 
La principal ventaja de esto es que no bloquea el programa con delays, de modo que puedo seguir leyendo el joystick, actualizando el LCD o detectando pulsaciones largas del botón mientras los tiempos avanzan en segundo plano.

El código también está dividido en funciones específicas como mostrarTempHum(), tomarDistancia(), mostrarMenu(), editarPrecio(), etc., para evitar repetir código y hace que cada funcionalidad quede aislada, siendo mucho más fácil de depurar. 
En cuanto a las interrupción hardware hago uso de una ISR que mide el tiempo exacto en microsegundos para distinguir entre una pulsación media (reinicio del servicio) y una larga (entrada o salida del modo administrador). 
Considero que esto es más preciso y robusto que intentar medir la pulsación desde el loop principal. Cuando el sistema entra en modo Admin, lo gestiono con una bandera (enAdmin) que permite "desacoplar" completamente esa interfaz de la lógica de servicio normal.

En general, toda la estructura del programa sigue una organización modular y basada en estados, lo que facilita controlar sensores, actuadores y menús sin que unos interfieran en otros, y permitiendo modificar partes del proyecto sin tener que tocar el núcleo principal.


## Dificultades encontradas y soluciones

**1. El estado PREPARANDO no duraba 4–8 s: se interrumpía antes**

Mi programa saltaba constantemente a ESPERANDO sin dejar que PREPARANDO terminara su temporización.
Esto se producía porque el botón del joystick estaba configurado como INPUT_PULLUP, por lo que realmente si estaba funcionando como se esperaba: HIGH = NO pulsado, LOW = pulsado

Pero en mi código trataba HIGH como pulsación, provocando reinicios constantes. Es por ello que si no pulsaba el botón, esté se reiniciaba sin parar.

Solución:
Invertir la lógica:

if (buttonState == LOW) {   // botón realmente pulsado
    estadoActual = ESPERANDO;
}

**2. El LED no aumentaba progresivamente: saltaba directo al brillo máximo**

Esto se producía por dos causas combinadas:

El estado PREPARANDO se abandonaba antes de tiempo (problema 1), por lo que no daba tiempo al PWM a incrementarse.

En algunos puntos se calculaba el progreso así:  progreso = (float) (elapsed / durante);

pero si ambos datos son unsigned long, la división ocurre antes del cast, resultando en 0.

Solución:
Asegurar conversión correcta: progreso = (float)elapsed / (float)durante;

**3. Declaración de variables dentro de un Switch**

Durante varios días dedique bastante tiempo a entender porque no se actualizaban de manera correcta mis variables, inicializadas dentro de este, ya que la medición del sensor DHT 11 mostraba datos que no correspondían con la situación actual.

Solución:
Gracias a la ayuda del profesor pudimos entender que el problema residía en tratar de no inicializar varuables en este ámbito, dado que no son correctamnete gestionabas, devolviendo basura en algunos casos.

**4. Muestra de opciones del Admin**

A la hora de mostrar las opciones posibles en la pantalla, y acceder a sus funcionalidades se solapaban ambos menús. Haciendo así que el muestreo de datos fuera ilegible, pero debía procurar mantener que este fuera de manera dinámica.

Solución:
Tuve que incluir unos delays para que la visualización fuera más cómoda a nivel de usuario.

**5. Acceso al cambio de precios**

Inicialmente solo podía acceder a cambiar mis precios de manera dinámica, pero sin poder navegar correctamente por mi manú, quedandome siempre en la misma opción (Café Solo).

Solución:
Gestionarlo de manera independiente, teniendo en cuenta todos los casos de movimientos de mi joystick y evitando interferencias en el trascurso de mi código.

## Vídeo que muestra la ejecucción

## License

This work is submitted under the CC-BY-SA 4.0 license, meaning it can be shared and adapted as long as it is properly attributed and shared under the same terms.

## Autor

Marina Antolínez Cabrero
