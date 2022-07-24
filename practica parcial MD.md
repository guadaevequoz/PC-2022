## Practica parcial - Memoria distribuida

### PMA - _Pasaje de Parametros Asincronico_

#### 1) Resolver con PMA (Pasaje de Mensajes ASINCRÓNICOS) el siguiente problema. En una oficina hay 3 empleados y P personas que van para ser atendidas para realizar un trámite. Cuando una persona llega espera hasta ser atendido por cualquiera de los empleados, le indica el trámite a realizar y espera a que le den el resultado. Los empleados atienden las solicitudes en orden de llegada; si no hay personas esperando, durante 5 minutos resuelven trámites pendientes (simular el proceso de resolver trámites pendientes por medio de un delay). Nota: no generar demora innecesaria; cada persona hace sólo un pedido y termina; los empleados no necesitan terminar su ejecución.

_Mi solucion:_

```java
chan tramite(int,text);
chan pedido(int);
chan resultado[P](text);
chan siguiente[3](text, int);

Process Empleado[id: 0..2]{
    text res, tram;
    int idP;

    while(true){
        send pedido(id)
        receive siguiente[id](tram, idP)
        if(!tram==="VACIO"){
            res = tram.resolver();
            send resultado[idP](res);
        }else{
            delay(500);
        }
    }
}

Process Persona[id: 0..P-1]{
    text tram, res;

    send tramite(id, tram);
    receive resultado[id](res);
}

Process Coordinador{
    text tram;
    int idP, idE;

    while(true){
        receive pedido(idE);
        if(empty(tramite)) tram = "VACIO"
        else receive(idP, tram);
        send siguiente[idE](tram,idP);
    }

}
```

_Solución del profesor:_

```java
chan solicitud(id);
chan respuestaTramite[P](text);
chan respuesta[P](int);
chan siguiente(int);
chan respuestaEmpleado[3](int);
chan datosTramite[3](text);

Process Empleado[id: 0..2]{
    text res, tram;
    int idP;

    while(true){
        send siguiente(id)
        receive respuestEmpleado[id](idP)
        if(idP > -1){
            send respuesta[idP](id);
            receive datosTramite[id](tram);
            res = tram.resolver();
            send respuestTramite[idP](res);
        } else delay(500);
    }
}

Process Persona[id: 0..P-1]{
    text tram, res;
    int idE;

    send solicitud(id);
    receive respuesta[id](idE);
    send datosTramite[idE](tramite);
    receive respuestaTramite[id](res);
}

Process Coordinador{
    int idP, idE;

    while(true){
        receive siguiente(idE);
        if(not empty(tramite)) receive solicitud(idP)
        else idP = -1;

        send respuestEmpleado[idE](idP);
    }

}
```

**Errores comunes:**

- No usar un coordinador.
- No responder el administrador al empleado cuando no habia solicitudes para atender.
- No usar arreglos de canales, usar un mismo canal para todos los empleados.

#### 2) Resolver con PMA (Pasaje de Mensajes ASINCRÓNICOS) el siguiente problema. En un negocio hay 5 empleados que atienden a N personas que van a pedir un presupuesto, de acuerdo al orden de llegada. Cuando el cliente sabe que empleado lo va a atender le entrega el listado de productos que necesita, y luego el empleado le entrega el presupuesto del mismo. Nota: maximizar la concurrencia. Existe una función HacerPresupuesto(lista) que simula la elaboración del presupuesto por parte de los empleados

_Mi solucion:_

```java
chan listado(int, text);
chan presupuesto[N](text);

Process Empleado[id: 0..4]{
    text lista, presupuesto;
    int idP;

    while(true){
        receive listado(idP, lista);
        presupuesto = HacerPresupuesto(lista);
        send presupuesto[idP](presupuesto);
    }
}

Process Persona[id: 0..N-1]{
    text lista, presupuesto;

    send listado(id, lista);
    receive presupuesto[id](presupuesto);
}


```

_Solución del profesor:_

```java
chan solicitud(int);
chan respuesta[N](int);
chan datosListado[5](text);
chan respuestaPresupuesto[N](text);

Process Empleado[id: 0..4]{
    text lista, presupuesto;
    int idP;

    while(true){
        receive solicitud(idP);
        send respuesta[idP](id);
        receive datosListado[id](lista);
        presupuesto = HacerPresupuesto(lista);
        send respuestaPresupuesto[idP](presupuesto);
    }
}

Process Persona[id: 0..N-1]{
    text lista, presupuesto;
    int idE;

    send solicitud(id);
    receive respuesta[id](idE);
    send datosListado[idE](lista);
    receive respuestaPresupuesto[id](presupuesto);
}

```

**Errores comunes:**

- Utilizar un proceso coordinador (problema conceptual aunque funcione el programa).
- No hacer interaccion directa con el empleado de enviar el listado recien cuando sé qué empleado es el que me atienda. Se hace justo cuando se hace la solicitud. (acá me equivoqué xd)
- No utilizar los arreglos de canales para cada empleado/persona y de esa manera se pisaban.

#### 3) Resolver con PASAJE DE MENSAJES ASINCRÓNICOS (PMA) el siguiente problema. Se debe simular la atención en un peaje con 7 cabinas para atender a N vehículos (algunos de ellos son ambulancias). Cuando el vehículo llega al peaje se dirige a la cabina con menos vehículos esperando y se queda ahí hasta que lo terminan de atender y le dan el ticket de pago. Las cabinas atienden a los vehículos que van a ella de acuerdo al orden de llegada pero dando prioridad a las ambulancias; cuando terminan de atender a un vehículo le dan el ticket de pago. Nota: maximizar la concurrencia.

_Solución del profesor:_

```java
chan pedirNum(int);
chan darNum[N](int);
chan salir(int);
chan avisoA();
chan llegaA[7](int);
chan llegaV[7](int);
chan avisoV[7]();
chan darTicket[N](text);
chan libre[7]();

Process Vehiculo[id:0..N-1]{
    int cabina;
    boolean esAmbulancia;
    text ticket;

    send pedirNum(id);
    send avisoA();
    receive darNum[id](cabina);
    if(esAmbulancia) send llegaA[cabina](id)
    else send llegaV[cabina](id);
    send avisoV[cabina]();
    receive darTicket[id](ticket);
    send libre[cabina]();
    send salir(cabina);
    send avisoA();
}

Process Cabina[id: 0..6]{
    text ticket;
    int idV;
    cola c;

    while(true){
        receive avisoV[id]();
        if(not empty(llegaA[id])) receive llegaA[id](idV);
        else receive llegaV[id](idV);
        ticket = idV.generarTicket();
        send darTicket[idV](ticket);
        receive libre[id]();
    }
}

Process Admin{
    int cabinas[7] = ([7] 0), idC, idV;

    while(true){
        receive avisoA();
        if(empty(salir) && (not empty(pedirNum))) -->
            receive pedirNum(idV);
            idC = cabinas.min();
            send darNum(idV)(idC);
            cabinas[idC]++;
        * (not empty(salir)) -->
            receive salir(idC);
            cabinas[idC]--;
        fi
    }

}

```

**Errores comunes:**

- Tener mal las condiciones de los if.
- Se generaba busy waiting (se soluciona agregando los avisos).
- Mandar la condicion de si era ambulancia o no en la condicion y despues agregarse a una cola. No tiene sentido esto, los canales ya de por si son colas.

#### 4) Resolver con PASAJE DE MENSAJES ASINCRÓNICOS (PMA) el siguiente problema. Se debe simular la atención en una planta para la VTV con 5 puestos para atender a N vehículos (que pueden ser sanitarios o comunes). Cuando el vehículo llega a la planta se dirige al puesto con menos vehículos esperando y se queda ahí hasta que le dan el comprobante de la verificación. Los puestos atienden a los vehículos que van a él de acuerdo al orden de llegada pero dando prioridad a los vehículos sanitarios; cuando terminan de atender a un vehículo le debe entregar un comprobante con los detalles de la verificación. Nota: maximizar la concurrencia.

_Mi solucion:_

```java

```

#### 5) Resolver con PASAJE DE MENSAJES ASINCRÓNICOS (PMA) el siguiente problema. Se debe simular la atención en un banco con 3 cajas para atender a N clientes que pueden ser especiales (son las embarazadas y los ancianos) o regulares. Cuando el cliente llega al banco se dirige a la caja con menos personas esperando y se queda ahi hasta que lo terminan de atender y le dan el comprobante de pago. Las cajas atienden a las personas que van a ella de acuerdo al orden de llegada pero dando prioridad a los clientes especiales; cuando terminan de atender a un cliente le debe entregar un comprobante de pago. Nota: maximizar la concurrencia.

_Mi solucion:_

```java

```

### PMS - _Pasaje de Parametros Sincronico_

#### 1) Resolver con PMS (Pasaje de Mensajes SINCRÓNICOS) el siguiente problema. En una carrera hay C corredores y 3 Coordinadores. Al llegar los corredores deben dirigirse a los coordinadores para que cualquiera de ellos le dé el número de "chaleco" con el que van a correr y luego se va. Los coordinadores atienden a los corredores de acuerdo al orden de llegada (cuando un coordinador está libre atiende al primer corredor que está esperando). Nota: maximizar la concurrencia.

_Mi solucion:_

```java
Process Corredores[id: 0..C-1]{
    int chaleco = -1;
    Admin!llegue(id);
    Coordinadores[id]?chaleco(chaleco);
    //correr
}

Process Coordinadores[id: 0..2]{
    int idAux;

    while(true){
        Admin!pedido(id);
        Admin[id]?siguiente(idAux);
        Corredores[id]!chaleco(random());
    }
}

Process Admin{
    int idA, idB;
    cola buffer;

    do Corredores[*]?llegue(idA) --> buffer.push(idA)
    * !buffer.isEmpty(); Coordinadores[*]?pedido(idB) --> Coordinadores[idB]!siguiente(buffer.pop())
    od

}
```

_Solucion del profesor_

```java
Process Corredor[id:0..C-1]{
    int num;

    AdminOrden!quieroNumero(id);
    Coordinador[*]?tomaNumero(num);
}

Process Coordinador[id:0..2]{
    int idC, num;

    while(true){
        AdminOrden!siguiente(id);
        AdminOrden?tomaCorredor(idC);
        num = generarNumero();
        Corredor[idC]!tomaNumero(num);
    }

}

Process AdminOrden{
    int idC, idE;
    queue cola;

    do Corredor[*]?quieroNumero(idC) --> cola.push(idC);
    * (!cola.isEmpty()); Coordinador[*]?siguiente(idE) --> idC=cola.pop();
                                                           Coordinador[idE]!tomaCorredor(idC);
    od
}

```

**Errores comunes:**

- Uso de variables compartidas.
- No usar el Admin. Es necesario porque el corredor no sabría a qué coordinador pedirle un número y no puedo usar comodín para enviar en la práctica.
- No poner el \* con los [] para tomar mensajes de todos.
- No consultar si la cola esta vacia antes de recibir el pedido de los corredores por el trabajo.

#### 2) Resolver con PMS (Pasaje de Mensajes SINCRÓNICOS) el siguiente problema. Simular la atención de una estación de servicio con un único surtidor que tiene un empleado que atiende a los N clientes de acuerdo al orden de llegada. Cada cliente espera hasta que el empleado lo atienda y le indica qué y cuánto cargar; espera hasta que termina de cargarle combustible y se retira. Nota: cada cliente carga combustible sólo una vez; todos los procesos deben terminar.

_Mi solucion:_

```java

Process Cliente[id:0..N-1]{
    Empleado!llegar(id, queCargar, cuantoCargar);
    Empleado[id]?irse();
}

Process Empleado{
    String queCargar;
    int cuantoCargar,idC;
    while(true){
        Cliente[*]?llegar(idC, queCargar, cuantoCargar);
        //atender
        Cliente[idC]!irse();
    }
}

```

_Solucion del profesor_

```java

Process Cliente[id:0..N-1]{
    AdminOrden!solicitaPaso(id);
    Empleado[id]?pasar();
    Empleado!cargar(tipo,cantidad);
    Empleado[id]?salir();

}

Process Empleado{
    int idC, cant;
    boolean terminar = false;
    text tipo;

    while(!terminar){
        AdminOrden!siguiente();
        AdminOrden?tomaCliente(idC, terminar);
        Cliente[idC]!pasar();
        Cliente[idC]?cargar(tipo,cant);
        //cargar
        Cliente[idC]!salir();
    }
}

Process AdminOrden{
    queue cola;
    int idC, idE, cantEnviados = 0;
    boolean fin = false;

    while(!fin){
        if Cliente[*]?solicitarPaso(idC) --> push (cola, idC);
        * (!cola.isEmpty()); Empleado?siguiente() --> cantEnviados++;
            if(cantEnviados == N) fin = true;
            idC = cola.pop();
            Empleado!tomaCliente(idC,fin);
        fi

    }

}

```

**Errores comunes:**

- No usar el administrador y hacer que la comunicación entre cliente y empleado sea directa. El probnlema es que como el empleado va a estar parado en una recepcion recibiendo con comodín, EL COMODIN NO RESPETA EL ORDEN.
- Sacar de la cola del admin sin que haya nada adentro.

#### 3) Resolver con PMS (Pasaje de Mensajes SINCRÓNICOS) el siguiente problema. En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con exclusión mutua) y un empleado encargado de administrar el uso del mismo. A su vez hay P personas que van a la exposición y solicitan usar el simulador, cada una de ellas espera a que el empleado lo deje acceder, lo usa por un rato y se retira para que el empleado deje pasar a otra persona. El empleado deja usar el simulador a las personas respetando el orden en que hicieron la solicitud. Nota: cada persona usa sólo una vez el simulador.

_Mi solucion:_

```java
Process Persona[id:0..P-1]{
    Admin!llegue(id);
    Empleado[id]?pasar();
    //usar
    Empleado!terminar();
}

Process Empleado{
    int idP;
    while(true){
        Admin!siguiente();
        Admin?tomaPersona(id);
        Persona[id]!pasar();
        Persona[*]?terminar();
    }
}

Process Admin{
    int idP, idE;
    queue cola;

   do Persona[*]?llegue(idP) -> cola.push(idP);
   * (!cola.isEmpty()); Empleado?siguiente() -> Empleado!tomaPersona(cola.pop());
   od

}


```

_Solucion del profesor:_ Dijo que se puede hacer con o sin admin. Se puede hacer sin admin porque el empleado no debe trabajar con la persona, sólo administra el acceso.

```java

Process Persona[id:0..P-1]{
    Empleado!solicitarPaso(id);
    Empleado?pasar();
    UsarSimulador();
    Empleado!terminar();
}

Process Empleado{
    queue cola;
    int idP;
    bool libre;

    do Persona[*]?solicitarPaso(idP) ->
        if(!libre) push (cola,idP)
        else {
            libre = false;
            Persona[idP]!pasar();
        }
    * Persona[*]?salir() ->
        if(!cola.isEmpty()) libre = true
        else {
            idP = cola.pop();
            Persona[idP]!pasar();
        }
    od
}

```

**Errores comunes:**

- No nombrar al proceso cuando le mando mensaje. Por ejemplo poner Persona!algo no es un proceso Persona, si hago Persona[algo] o Persona[*] si es un proceso.

### ADA

#### Resolver el siguiente problema ADA. Simular el con funcionamiento de un Entrenamiento de Básquet donde hay 20 jugadores y UN entrenador. El entrenador debe distribuir a los jugadores en 2 canchas. Cuando un jugador llega el entrenador le indica la cancha a la cual debe ir para que se dirija a ella y espere hasta que lleguen los 10; en ese momento comienzan a jugar el partido que dura 40 minutos. Cuando ambos partidos han terminado, el entrenador les da a los 20 jugadores una charla de 10 minutos y luego todos se retiran. El entrenador asigna el número de cancha en forma cíclica 1, 2, 1, 2 y así sucesivamente.

_Solucion:_

```ada
Procedure parcial{
    TASK ENTRENADOR IS
        ENTRY asignar(CANCHA: OUT int);
        ENTRY terminoPartido;
        ENTRY salir;
    END ENTRENADOR;
    TASK TYPE JUGADOR;
    TASK TYPE CANCHA IS
        ENTRY llegue;
        ENTRY iniciar;
        ENTRY terminar;
    END CANCHA;

    JUGADORES: ARRAY[0..19] OF JUGADOR;
    CANCHAS: ARRAY[0..2] OF CANCHA;

    TASK BODY ENTRENADOR IS
    BEGIN
        for C in 0..19 loop
            ACCEPT ASIGNAR(CANCHA: OUT int) do
                CANCHA = (C MOD 2)+1;
            end ASIGNAR;
        end loop;
        for i in 0..1 loop
            ACCEPT terminoPartido;
        end loop;
        delay(10 minutos);
         for i in 0..19 loop
            ACCEPT salir;
        end loop;
    END ENTRENADOR;

    TASK BODY JUGADOR IS
    VAR
        CANCHA: int;
    BEGIN
        ENTRENADOR.asignar(CANCHA);
        CANCHAS[CANCHA].llegue;
        CANCHAS[CANCHA].iniciar;
        CANCHAS[CANCHA].terminar;
        ENTRENADOR.salir;
    END JUGADOR;

    TASK BODY CANCHA IS
    BEGIN
        for i in 0..9 loop
            ACCEPT llegue;
        end loop;
        for i in 0..9 loop
            ACCEPT iniciar;
        end loop;
        delay(60 minutos);
        for i in 0..9 loop
            ACCEPT terminar;
        end loop;
        ENTRENADOR.terminoPartido;
    END CANCHA;
}

```

**Errores comunes:**

- El entrenador o el jugador hacian el delay del tiempo de partido.
- No aprovechar la conexión bidireccional.
- Poner siempre parametros de entrada.

#### Resolver el siguiente problema con ADA. Existen N personas que van a realizar un pago en la caja de un banco. Las personas son atendidas de acuerdo al orden de llegada, aunque aquellos que sean jubilados tienen prioridad sobre los que no lo sean. Todas las personas esperan a lo sumo 15 minutos para ser atendidas. Si pasado ese tiempo no fueron atendidas, le dejan una nota de reclamo al responsable de Informes y se retiran.

_Mi solucion:_

```ada
Procedure banco{
    TASK TYPE PERSONA;
    TASK EMPLEADO IS
        ENTRY atender(PAGO: IN int);
        ENTRY atenderJubilado(PAGO: IN int);
    END;
    TASK INFORME IS
        ENTRY reclamo(NOTA: IN text);
    END;

    PERSONAS: array [0..N-1] of PERSONA;

    TASK BODY PERSONA IS
    VAR
        esJubilado: boolean;
        pago: int;
    BEGIN
        SELECT
            if(esJubilado){
                EMPLEADO.atenderJubilado(pago);
            }else{
                EMPLEADO.atender(pago);
            }
        OR DELAY 15minutos
            INFORME.reclamo(notaReclamo);
        END SELECT;
    END PERSONA;

    TASK BODY EMPLEADO IS

    BEGIN
        loop
            SELECT
                ACCEPT atenderJubilado(PAGO: IN INT);
            OR
                when(atenderJubilado'count == 0) => ACCEPT atender(PAGO: IN INT);
            END SELECT;
        end loop;
    END EMPLEADO;

    TASK BODY INFORME IS
    BEGIN
        loop
            ACCEPT reclamo(NOTA: IN text);
        end loop;
    END INFORME;
}
```

_Solucion del profesor:_

```ada
Procedure banco{
    TASK TYPE PERSONA;
    TASK EMPLEADO IS
        ENTRY atencionJub(t: IN int; r: OUT text);
        ENTRY atencionNoJub(t: IN int; r: OUT text);
    END;
    TASK RESPONSABLEINFORMES IS
        ENTRY realizarReclamo(NOTA: IN text);
    END;

    PERSONAS: array [0..N-1] of PERSONA;

    TASK BODY PERSONA IS
    VAR
        esJubilado: boolean;
        pago: int;
        r: text;
        nota: text;
    BEGIN
        if(esJubilado){
            SELECT
                EMPLEADO.atencionJub(pago,r);
            OR DELAY(15*60)
                RESPONSABLEINFORMES.realizarReclamo(reclamo);
            END SELECT;
        }else{
            SELECT
                EMPLEADO.atencionNoJub(pago,r);
            OR DELAY(15*60)
                RESPONSABLEINFORMES.realizarReclamo(reclamo);
            END SELECT;
        }


        SELECT

        OR DELAY 15minutos
            INFORME.reclamo(notaReclamo);
        END SELECT;
    END PERSONA;

    TASK BODY EMPLEADO IS
    BEGIN
        loop
            SELECT
                ACCEPT atencionJub(t: IN INT; r: OUT text) do
                    r = resolverTramite(t):
                end atencionJub;
            OR
                when(atenderJubilado'count == 0) => ACCEPT atencionNoJub(t: IN INT; r: OUT text) do
                    r = resolverTramite(t):
                end atencionNoJub;
            END SELECT;
        end loop;
    END EMPLEADO;

    TASK BODY INFORME IS
    VAR
        reclamos: queue;
    BEGIN
        loop
            ACCEPT reclamo(NOTA: IN text) do
                reclamos.push(NOTA);
            end reclamo;
        end loop;
    END INFORME;
}
```

**Errores comunes:**

- No puede ir un if luego del SELECT, tiene que ir si o si un entry call.
- No modelaban al empleado ni al responsable del informe.
- No implementaron la parte de los informes.
- Usar un sólo entry en el empleado/No dar prioridad a los jubilados.
- No aprovechar la bidireccionabilidad.

#### Resolver el siguiente problema con ADA. Se quiere modelar un puente colgante de un solo sentido, el cual resiste hasta 100 personas. Suponga que hay una cantidad innumerable de personas que quieren cruzar y que sólo lo podrán hacer si no superan la cantidad máxima. Las personas cruzan el puente de acuerdo al orden de llegada, aunque los jubilados tienen prioridad sobre los no jubilados.

_Mi solucion:_

```ada
Procedure Puente{
    TASK PUENTE IS
        ENTRY pasarJub;
        ENTRY pasarNoJub;
        ENTRY salir;
    END PUENTE;

    TASK TYPE PERSONA;
    PERSONAS: array[0..N-1] of PERSONA;

    TASK BODY PERSONA IS
    BEGIN
        if(esJubilado){
            PUENTE.pasarJub;
        }else{
            PUENTE.pasarNoJub;
        }
        //pasar por el puente
        PUENTE.salir;
    END;

    TASK BODY PUENTE IS
    VAR
        cant = int;
    BEGIN
        loop
            SELECT
                when(cant  < 100) => ACCEPT pasarJub() do
                    cant ++;
                end ACCEPT;
            OR
                when(cant  < 100) AND (pasarJub'Count == 0) => ACCEPT pasarNoJub() do
                    cant ++;
                end ACCEPT;
            OR
                ACCEPT salir do
                    cant--;
                end do;
            END SELECT;
        end loop;
    END;
}

```

_Solucion del profesor:_

```ada
Procedure  {
    TASK TYPE JUBILADO;
    TASK TYPE NOJUBILADO;
    TASK ADMIN IS
        ENTRY entrada_jub;
        ENTRY salida;
        ENTRY entrada_nojub;
    END ADMIN;

    JUBILADOS: array [1..A] of JUBILADO;
    NOJUBILADOS: array [1..A] of NOJUBILADO;

    TASK BODY JUBILADO IS
    BEGIN
        ADMIN.entrada_jub;
        //pasar
        ADMIN.salida;
    END JUBILADO;

    TASK BODY NOJUBILADO IS
    BEGIN
        ADMIN.entrada_nojub;
        //pasar
        ADMIN.salida;
    END NOJUBILADO;

    TASK BODY ADMIN IS
    VAR
        cant: int;
    BEGIN
        loop
            SELECT
                ACCEPT salida;
                cant--;
            OR
                WHEN(cant < 100) => ACCEPT entrada_jub;
                                    cant++;
            OR
                WHEN(cant < 100) AND (entrada_jub'count == 0)=> ACCEPT entrada_nojub;
                                    cant++;
            END SELECT;
        end loop;
    END;
}

```

#### Resolver el siguiente problema con ADA. En una empresa de organización de recitales cuentan con un sitio web para la venta de entradas. Existen N personas que quieran comprar una de las E entradas disponibles. Las personas se dividen en pacientes e impacientes. Cuando las personas pacientes intentan hacer la compra en el sitio web, esperan a lo sumo 5 minutos a que el servidor responda. Si pasado ese tiempo no fueran atendidas, lo vuelven a intentar. En el caso de las personas impacientes, si no son atendidas inmediatamente por el servidor, esperan 10 segundos y vuelvan a intentarlo. En todos los casos, el sitio web les dice si pudieron venderle la entrada o no.

_Solucion:_

```ada
Procedure practica{
    TASK TYPE PACIENTE;
    TASK TYPE IMPACIENTE;
    TASK SW IS
        ENTRY atencion(r: OUT boolean);
    END SW;

    PACIENTES: array[1..N] of PACIENTE;
    IMPACIENTES: array[1..N] of IMPACIENTES;

    TASK BODY PACIENTE IS
    VAR
        res: bool;
        atendido: bool;
    BEGIN
        atendido = false;
        while(!atendido) loop
            SELECT
                SW.atencion(res);
                atendido = true;
            OR DELAY 300
                null;
            END SELECT;
        end loop;
    END PACIENTE;

    TASK BODY IMPACIENTE IS
    VAR
        res: bool;
        atendido: bool;
    BEGIN
        while(!atendido) loop
            SELECT
                SW.atencion(res);
                atendido = true;
            ELSE
                DELAY(10);
            END SELECT;
        end loop;
    END IMPACIENTE;

    TASK BODY SW IS
    VAR
        cantV: int;
    BEGIN
        loop
            ACCEPT atencion(r: OUT bool)do
                if(cantV <= E) {
                    cantV++;
                    r = true;
                }
                else r = false;
            end atencion;
        end loop;
    END SW;
}
```

**Errores comunes:**

- Usar SELECT OR DELAY en vez de SELECT ELSE en el impaciente.
- Hacer pedidos diferentes segun el tipo de paciente.

#### Resolver el siguiente problema con ADA. Simular el funcionamiento de un Complejo de Canchas de Paddle que posee 10 canchas y donde hay UN encargado que se encarga de distribuir a las personas en las canchas. Al complejo acuden 40 personas a jugar. Cuando una persona llega el encargado le indique el número de cancha a la cual debe ir, y se dirige a ella; cuando han llegado los 4 jugadores a la cancha, comienzan a jugar el partido que dura 60 minutos; al terminar el partido las 4 personas se retiran. El encargado asigna el número de cancha según el orden de llegada (los 4 primeros a la cancha 1, los siguientes 4 a la 2 y así sucesivamente). Nota: cuando el encargado ya atendió a las 40 personas debe terminar su ejecución.

_Solucion:_

```ada
Procedure practica{
    TASK TYPE JUGADOR;
    TASK ENCARGADO IS
        ENTRY asignar(CANCHA: OUT int);
    END ENCARGADO;
    TASK TYPE CANCHA IS
        ENTRY llegue;
        ENTRY empezar;
        ENTRY terminar;
    END CANCHA;

    JUGADORES: array [0..39] of JUGADOR;
    CANCHAS: array [0..9] of CANCHA;

    TASK BODY JUGADOR IS
    VAR
        cancha: int;
    BEGIN
        ENCARGADO.asignar(cancha);
        CANCHA[cancha].llegue;
        CANCHA[cancha].empezar;
        CANCHA[cancha].terminar;
    END JUGADOR;

    TASK BODY ENCARGADO IS
    BEGIN
        for C in 1..10 loop
            for P in 1..4 loop
                ACCEPT asignar(CANCHA: OUT int) do
                    CANCHA = C;
                end asignar;
            end loop;
        end loop;
    END ENCARGADO;

    TASK BODY CANCHA IS
    BEGIN
        for i in 1..4 loop
            ACCEPT llegue;
        end loop;
        for i in 1..4 loop
            ACCEPT empezar;
        end loop;
        delay(60 minutos);
        for i in 1..4 loop
            ACCEPT terminar;
        end loop;
    END CANCHA;

}
```
