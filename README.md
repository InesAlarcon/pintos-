# pintos-

Fase 1:

timer.c
Implementamos la función thread block para poder quitar el busy waiting en el alarm.
thread.c
Para pasar la prueba de prioridad del alarm, solo necesitamos mantener la Ready List como una lista de prioridad, ordenada de alta a baja prioridad en estas tres funciones: init_thread, thread_unblock, thread_yield.
Para la donación de prioridad descubrimos que al cambiar la prioridad de un hilo, el orden de ejecución de todos los hilos se ve afectado. Por lo tanto, solo podemos insertar el thread en la Ready List cuando la prioridad del thread ya haya sido cambiada, para que podamos reordenar todos los hilos. De manera similar, al crear un thread, si la prioridad es mayor que el thread actual, el thread actual también se insertará en la Ready List. Entonces modificamos thread_set_priority y thread_create, donde se determina si se inserta el thread actual en la Ready List según la prioridad del nuevo thread.
Para esta fase se inicialízan en init_thread las funciones lock_acquire y lock_release para poder manejar los bloqueos de los threads. 
synch.c
Implementamos donación recursiva, y modificamos el miembro max_priority del lock para poder actualizar la prioridad a través de la función thread_update_priority para lograr la donación prioritaria. Realizamos una comparación de prioridades dentro de la cola de bloqueo (thread_cmp_priority), luego cambiamos el comportamiento al liberar el bloqueo, es decir, modificar la función lock_release e  implementar la función thread_remove_lock. Finalmente, implementamos thread_update_priority, esta función maneja el cambio de prioridad al liberar el bloqueo: si el subproceso actual todavía tiene un bloqueo, adquiere la máxima prioridad del bloqueo que posee. Si es mayor que base_priority, actualice la prioridad donada. También implementamos una función  cond_sema_cmp_priority que compara las prioridades de los semáforos para poder ordenar las threads que se encuentren esperando al igual que sema_down y sema_up. De esta manera, la cola del semáforo se cambia a la cola de prioridad.

MLFQS:
Implementamos la función de incremento Recent_cpu y la función para actualizar el sistema load_avg y Recent_cpu de todos los threads. En init_thread revisamos si el thread es de mlfqs y se realiza una operación para determinar su prioridad en base al PRI_MAX, su recent CPU y su niceness. En thread_tick verificamos el thread y se realizan varios pasos, si el thread esta idle, se le suma 1 a su recent cpu y dependiendo del modulo de sus ticks 100 se calcula su load_avg por la frecuencia y cuando sus clock ticks son 4 se calcula y se actualiza su nueva prioridad. Agregamos un archivo llamado fixed-point porque desafortunadamente, Pintos no admite aritmética de punto flotante en su kernel, porque complicaría y afectaría el rendimiento del núcleo. Los núcleos reales a menudo tienen la misma limitación, por la misma razón. Esto significa que los cálculos sobre cantidades reales deben simularse utilizando números enteros. 

----------------------------------------------------------------------------------------------------------------------------------------
Fase 2:

thread.h
Agregamos la estructura de un process hijo y dentro de USERPROG agregamos estructuras de ayuda para los process hijos. Creamos una lista para llevar control de locks, una estructura para file actual y la descripción del file actual.
thread.c 
En init_process agregamos la inicialización de un process hijo y una lista una vacía en la que ingresamos el child_lock y la descripción del file.
exception.c
En page_fault() agregamos condicionales para verificar si el address es incorrecto o nulo y revisar las lecturas y escrituras, si se desea realizar una escritura cuando la operación no lo es, esto genera un error, por lo que se envía como un exit erróneo o tambien cuando accedemos a una pagina de solo lectura y queremos escribir en ella realizamos una salida del sistema. 
process.c
Se actualizan nuevas librerías, solo adicionamos syscall.h
process_execute
Implementamos en esta función el obtener una página libre para nuestro programa, archivo y proceso hijo, el programa es tokenizado y se guarda como argumentos. Se crea un nuevo proceso para poder ejecutar el programa y utilizamos un semáforo para poder esperar al proceso hijo, una vez verificamos que el proceso actual es el hijo entonces procedemos a bloquearlo y liberamos páginas en memoria ocupadas por el programa, de lo contrario, si no es el hijo entonces todas las páginas son liberadas. 
Start_process
En esta función implementamos el manejo del stack, lo inicializamos y asignamos el stack pointer a nuestro proceso actual. Utilizamos interrupciones para cargar los programas y verificamos los argumentos para poder guardarlos. 
process_wait
Implementamos una función que verifica si tenemos procesos hijos en la lista de lock y comenzamos a mover estos procesos al frente. Si no tenemos procesos o el hijo actual es el que está esperando, su espera termina o si bien el proceso padre se encuentra ejecutando, el hijo deberá esperar. 
process_exit
Implementamos una función que verifica el código de salida de los procesos, si lo tiene cerramos el proceso y si no etiquetamos el proceso como huérfano. 

syscall.c
En este archivo, agregamos muchas más librerías que son necesarias para poder trabajar en crear las funciones necesarias para los syscalls. Creamos varias funciones, las primeras para poder trabajar con la memoria del sistema, y las siguientes para poder realizar los syscalls. Los syscalls como tal funcionan dentro de un switch.

----------------------------------------------------------------------------------------------------------------------------------------
Materiales de referencia: 

http://www.ccs.neu.edu/home/amislove/teaching/cs5600/fall10/pintos/pintos_7.html

http://math.hws.edu/eck/cs431/f16/assg5/index.html

https://web.stanford.edu/class/cs140/projects/pintos/pintos.html#SEC_Top

http://www.ccs.neu.edu/home/amislove/teaching/cs5600/fall10/pintos/pintos.html#SEC_Top

https://static1.squarespace.com/static/5b18aa0955b02c1de94e4412/t/5b85fad2f950b7b16b7a2ed6/1535507195196/Pintos+Guide

http://dcclab.sogang.ac.kr/?module=file&act=procFileDownload&file_srl=1115677&sid=fe1dfeb23030c6fa1e57daf999746632

http://bits.usc.edu/cs350/assignments/project2.pdf

https://courses.cs.vt.edu/cs4284/spring2013/pintos/doc/pintos.pdf

https://zhuanlan.zhihu.com/p/104497182

https://slideplayer.com/slide/3997126/

https://slideplayer.com/slide/7988080/
