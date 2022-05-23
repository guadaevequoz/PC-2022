### 1) Se requiere modelar un puente de un solo sentido, el puente solo soporta el peso de 5 unidades de peso. Cada auto pesa 1 unidad, cada camioneta pesa 2 unidades y cada camión 3 unidades. Suponga que hay una cantidad innumerable de vehículos (A autos, B camionetas y C camiones).

#### a) Realice la solución suponiendo que todos los vehículos tienen la misma prioridad.

```ada
PROCEDURE PUENTE {

TASK PUENTE IS
    ENTRY ENTRAAUTO();
    ENTRY ENTRACAMION();
    ENTRY ENTRACAMIONETA();
    ENTRY SALEAUTO();
    ENTRY SALECAMION();
    ENTRY SALECAMIONETA();
END PUENTE;

TASK TYPE CAMION {}
TASK TYPE AUTO {}
TASK TYPE CAMIONETA {}

CAMIONES: ARRAY[0..C-1] OF CAMION;
CAMIONETAS: ARRAY[0..B-1] OF CAMIONETA;
AUTOS: ARRAY[0..A-1] OF AUTO;

TASK BODY CAMION {
    PUENTE.ENTRACAMION();
    PUENTE.SALECAMION();
}
TASK BODY AUTO {
    PUENTE.ENTRAAUTO();
    PUENTE.SALEAUTO();
}
TASK BODY CAMIONETA {
    PUENTE.ENTRACAMIONETA();
    PUENTE.SALECAMIONETA();
}

TASK BODY PUENTE {
    INT PESO = 0;
    WHILE(TRUE)LOOP{
        SELECT
            WHEN(PESO + 1 <= 5) ACCEPT ACCEPT ENTRAAUTO()
                PESO += 1;
            END ENTRAAUTO()
        OR
            WHEN(PESO + 2 <= 5) ACCEPT ACCEPT ENTRACAMIONETA()
                PESO += 2;
            END ENTRACAMIONETA()
        OR
            WHEN(PESO + 3 <= 5) ACCEPT ACCEPT ENTRACAMION()
                PESO += 3;
            END ENTRACAMION()
        OR
            ACCEPT SALEAUTO()
                PESO -=1
            END SALEAUTO()
        OR
            ACCEPT SALECAMIONETA()
                PESO -=2
            END SALECAMIONETA()
        OR
            ACCEPT SALECAMION()
                PESO -=3
            END SALECAMION()
        END
    }
}

}
```

#### b) Modifique la solución para que tengan mayor prioridad los camiones que el resto de los vehículos.

```ada

PROCEDURE PUENTE {

TASK PUENTE IS
    ENTRY ENTRAAUTO();
    ENTRY ENTRACAMION();
    ENTRY ENTRACAMIONETA();
    ENTRY SALEAUTO();
    ENTRY SALECAMION();
    ENTRY SALECAMIONETA();
END PUENTE;

TASK TYPE CAMION;
TASK TYPE AUTO;
TASK TYPE CAMIONETA;

CAMIONES: ARRAY[0..C-1] OF CAMION;
CAMIONETAS: ARRAY[0..B-1] OF CAMIONETA;
AUTOS: ARRAY[0..A-1] OF AUTO;

TASK BODY CAMION IS
    PUENTE.ENTRACAMION();
    PUENTE.SALECAMION();
END CAMION;

TASK BODY AUTO IS
    PUENTE.ENTRAAUTO();
    PUENTE.SALEAUTO();
END AUTO;

TASK BODY CAMIONETA IS
    PUENTE.ENTRACAMIONETA();
    PUENTE.SALECAMIONETA();
END CAMIONETA;

TASK BODY PUENTE IS
    INT PESO = 0;
    WHILE(TRUE)LOOP
        SELECT
            WHEN(PESO + 1 <= 5) AND ((ENTRACAMION'COUNT > 0) AND (PESO + 3 + 1 > 5)) OR (ENTRACAMION'COUNT == 0) ACCEPT ENTRAAUTO()
                PESO += 1;
        END ENTRAAUTO()
        OR
            WHEN(PESO + 2 <= 5) AND ((ENTRACAMION'COUNT > 0) AND (PESO + 3 + 2 > 5)) OR (ENTRACAMION'COUNT == 0) ACCEPT ENTRACAMIONETA()
                PESO += 2;
            END ENTRACAMIONETA()
        OR
            WHEN(PESO + 3 <= 5) ACCEPT ACCEPT ENTRACAMION()
                PESO += 3;
            END ENTRACAMION()
        OR
            ACCEPT SALEAUTO()
                PESO -=1
            END SALEAUTO()
        OR
            ACCEPT SALECAMIONETA()
                PESO -=2
            END SALECAMIONETA()
        OR
            ACCEPT SALECAMION()
                PESO -=3
            END SALECAMION()
        END
    END LOOP
END PUENTE;

}

```

### 2) Se quiere modelar la cola de un banco que atiende un solo empleado, los clientes llegan y si esperan más de 10 minutos se retiran.

```ada
PROCEDURE BANCO{
    TASK EMPLEADO IS
        ENTRY PEDIDO(D: IN TEXTO; R: OUT TEXTO)
    END EMPLEADO;
    TASK TYPE CLIENTE;

    CLIENTES: ARRAY[0..N-1] OF CLIENTE;

    TASK BODY CLIENTE IS
        TEXTO RES;
    BEGIN
        SELECT
            EMPLEADO.PEDIDO("DATOS", RES);
        OR DELAY 600.0
            NULL;
        END SELECT;
    END CLIENTE;

    TASK BODY EMPLEADO IS
    BEGIN
        WHILE(TRUE) LOOP
            ACCEPT PEDIDO(D: IN TEXTO; R: OUT TEXTO) DO
                R = RESOLVERPEDIDO(D);
            END PEDIDO;
        END LOOP;
    END EMPLEADO;

}
```

### 3) Se dispone de un sistema compuesto por 1 central y 2 procesos. Los procesos envían señales a la central. La central comienza su ejecución tomando una señal del proceso 1, luego toma aleatoriamente señales de cualquiera de los dos indefinidamente. Al recibir una señal de proceso 2, recibe señales del mismo proceso durante 3 minutos. El proceso 1 envía una señal que es considerada vieja (se deshecha) si en 2 minutos no fue recibida. El proceso 2 envía una señal, si no es recibida en ese instante espera 1 minuto y vuelve a mandarla (no se deshecha).

```ada
PROCEDURE PROGRAM {
    TASK CENTRAL IS
        ENTRY SEÑAL1();
        ENTRY SEÑAL2();
    END CENTRAL;
    TASK PROCESO1;
    TASK PROCESO2;

    TASK BODY CENTRAL IS
        ACCEPT SEÑAL1();
        END SEÑAL1;

        WHILE(TRUE) LOOP
            SELECT
                ACCEPT SEÑAL1();
            OR
                ACCEPT SEÑAL2();
                SELECT
                    SEÑAL2();
                OR DELAY 180
                END SELECT;
            END SELECT;
        END LOOP;
    END CENTRAL;

    TASK BODY PROCESO1 IS
        SELECT
            CENTRAL.SEÑAL1()
        OR DELAY 180
        END SELECT;
    END PROCESO1;

    TASK BODY PROCESO2 IS
        SELECT
            CENTRAL.SEÑAL2()
        ELSE
            DELAY 60
                CENTRAL.SEÑAL2()
            END
        END SELECT;
    END PROCESO2;

}
```

### 4) En una clínica existe un médico de guardia que recibe continuamente peticiones de atención de las E enfermeras que trabajan en su piso y de las P personas que llegan a la clínica ser atendidos. Cuando una persona necesita que la atiendan espera a lo sumo 5 minutos a que el médico lo haga, si pasado ese tiempo no lo hace, espera 10 minutos y vuelve a requerir la atención del médico. Si no es atendida tres veces, se enoja y se retira de la clínica. Cuando una enfermera requiere la atención del médico, si este no lo atiende inmediatamente le hace una nota y se la deja en el consultorio para que esta resuelva su pedido en el momento que pueda (el pedido puede ser que el médico le firme algún papel). Cuando la petición ha sido recibida por el médico o la nota ha sido dejada en el escritorio, continúa trabajando y haciendo más peticiones. El médico atiende los pedidos dándole prioridad a los enfermos que llegan para ser atendidos. Cuando atiende un pedido, recibe la solicitud y la procesa durante un cierto tiempo. Cuando está libre aprovecha a procesar las notas dejadas por las enfermeras.

```ada
PROCEDURE CLINICA {
    TASK MEDICO IS
        ENTRY PERSONAENFERMA();
        ENTRY PERSONANORMAL();
        ENTRY ENFERMERA();
        ENTRY NOTAENFERMERA();
    END MEDICO;

    TASK TYPE PERSONA;
    TASK TYPE ENFERMERA;

    PERSONAS: ARRAY[0..P-1] OF PERSONA;
    ENFERMERAS: ARRAY[0..E-1] OF ENFERMERA;

    TASK BODY MEDICO IS
    BEGIN
        WHILE(TRUE) LOOP
            SELECT
                WHEN (PERSONENFERMA'COUNT == 0) ACCEPT PERSONANORMAL()
                    DELAY X; //LA ATIENDE
                END PERSONANORMAL();
            OR
                ACCEPT PERSONAENFERMA()
                    DELAY X; //LA ATIENDE
                END PERSONAENFERMA();
            OR
                WHEN (PERSONENFERMA'COUNT == 0) AND (PERSONNORMAL'COUNT == 0) ACCEPT ENFERMERA()
                    DELAY X; //ATENDER ENFERMERA
                END ENFERMERA();
            ELSE
                ACCEPT NOTAENFERMERA(P: IN STRING)
                    P.ATENDER()
                END NOTAENFERMERA():
            END SELECT;
        END LOOP;
    END MEDICO;

    TASK BODY ENFERMERA IS
    BEGIN
        WHILE(TRUE) LOOP
            SELECT
                MEDICO.ENFERMERA();
            ELSE
                MEDICO.NOTAENFERMERA();
            END SELECT;
        END LOOP;
    END ENFERMERA;

    TASK PERSONA IS
    VAR
        ESENFERMA: BOOLEAN;
        ATENTIDO: BOOLEAN;
        CANT: INT;
    BEGIN
        ATENTIDO = FALSE;
        CANT = 0;
        WHILE(!ATENDIDO && CANT < 3) LOOP
            CANT++;
            SELECT
                IF(ESENFERMA) MEDICO.PERSONAENFERMA()
                ELSE MEDICO.PERSONANORMAL()
            OR DELAY 5
                DELAY(10);
            END SELECT;
        END LOOP;
    END PERSONA;
}
```

### 5) En un sistema para acreditar carreras universitarias, hay UN Servidor que atiende pedidos de U Usuarios de a uno a la vez y de acuerdo con el orden en que se hacen los pedidos. Cada usuario trabaja en el documento a presentar, y luego lo envía al servidor; espera la respuesta de este que le indica si está todo bien o hay algún error. Mientras haya algún error, vuelve a trabajar con el documento y a enviarlo al servidor. Cuando el servidor le responde que está todo bien, el usuario se retira. Cuando un usuario envía un pedido espera a lo sumo 2 minutos a que sea recibido por el servidor, pasado ese tiempo espera un minuto y vuelve a intentarlo (usando el mismo documento).

```ada
PROCEDURE SISTEMA {
    TASK SERVIDOR IS
        ENTRY PEDIDO(P: IN text, R: OUT boolean);
    END SERVIDOR;

    TASK TYPE USUARIO;

    USUARIOS: ARRAY [0..U-1] OF USUARIO;

    TASK BODY SERVIDOR IS
    BEGIN
        WHILE(TRUE) LOOP
            ACCEPT PEDIDO(P: IN text, R: OUT boolean)
                R = P.atender();
            END PEDIDO();
        END LOOP;
    END SERVIDOR;

    TASK BODY USUARIO IS
    VAR
        R: boolean;
        DOC: text;
    BEGIN
        R = false;
        WHILE(!R) LOOP
            DOC.Modificar();
            SELECT
                SERVIDOR.PEDIDO(DOC, R);
            OR DELAY 2
                DELAY(1);
            END SELECT;
        END LOOP;
    END USUARIO;

}

```

### 6) En una playa hay 5 equipos de 4 personas cada uno (en total son 20 personas donde cada una conoce previamente a que equipo pertenece). Cuando las personas van llegando esperan con los de su equipo hasta que el mismo esté completo (hayan llegado los 4 integrantes), a partir de ese momento el equipo comienza a jugar. El juego consiste en que cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser de 1, 2 o 5 pesos) y se suman los montos de las 60 monedas conseguidas en el grupo. Al finalizar cada persona debe conocer el grupo que más dinero junto. Nota: maximizar la concurrencia. Suponga que para simular la búsqueda de una moneda por parte de una persona existe una función Moneda() que retorna el valor de la moneda encontrada.

```ada

```

### 7) Hay un sistema de reconocimiento de huellas dactilares de la policía que tiene 8 Servidores para realizar el reconocimiento, cada uno de ellos trabajando con una Base de Datos propia; a su vez hay un Especialista que utiliza indefinidamente. El sistema funciona de la siguiente manera: el Especialista toma una imagen de una huella (TEST) y se la envía a los servidores para que cada uno de ellos le devuelva el código y el valor de similitud de la huella que más se asemeja a TEST en su BD; al final del procesamiento, el especialista debe conocer el código de la huella con mayor valor de similitud entre las devueltas por los 8 servidores. Cuando ha terminado de procesar una huella comienza nuevamente todo el ciclo. <br> Nota: suponga que existe una función Buscar(test, código, valor) que utiliza cada Servidor donde recibe como parámetro de entrada la huella test, y devuelve como parámetros de salida el código y el valor de similitud de la huella más parecida a test en la BD correspondiente. Maximizar la concurrencia y no generar demora innecesaria.

```ada

PROCEDURE SISTEMA {
    TASK ESPECIALISTA;
    TASK ADMINISTRADOR;

    TASK TYPE SERVIDOR IS
        ENTRY PEDIDO(TEST: IN text, CODIGO: OUT int, VALOR: OUT int);
    END SERVIDOR;

    SERVIDORES: ARRAY [0..7] OF SERVIDOR;

    TASK BODY ESPECIALISTA IS
    VAR
        CODIGO: ARRAY[8] OF INT;
        VALOR: ARRAY[8] OF INT;
        HUELLA: INT;
    BEGIN
        WHILE(TRUE) LOOP
            FOR(INT i = 0; i < 8; i++){
                SERVIDORES[i].PEDIDO(TEST, CODIGO[i], VALOR[i]);
            }
            HUELLA = CODIGO[VALOR.indexOf(VALOR.max())];
        END LOOP;

    END ESPECIALISTA;

    TASK BODY SERVIDOR IS
    BEGIN
        WHILE(TRUE) LOOP
            ACCEPT PEDIDO(TEST, CODIGO, VALOR)
                Buscar(TEST, CODIGO, VALOR);
            END PEDIDO;
        END LOOP;
    END SERVIDOR;
}

```
