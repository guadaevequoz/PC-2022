## TP4

## PMA

### 1)

```java
chan Atencion(int);

Process Persona[id:0..N-1]{
    llegar();
    send Atencion(id);
}

Process Empleado[id:0..1]{
    int idP;
    while(true){
        receive Atencion(idP);
        idP.atender();
    }
}
```

### 2)

```java
chan Atencion[5](int);
chan BuscarCaja(int);
chan Atendido(int);
chan MenorCola[5](int);
chan Comprobante[P](text);

```

## PMS

### 1) Suponga que existe un antivirus distribuido en él hay R procesos robots que continuamente están buscando posibles sitios web infectados; cada vez que encuentran uno avisan la dirección y continúan buscando. Hay un proceso analizador que se encargue de hacer todas las pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si están o no infectados.

```java
Process Robot[id:0..R-1]{
    String sitio;
    while (true){
        if(sitio.buscarVirus()) Analizador!(sitio);
    }
}

Process Analizador {
    while (true){
        Robot[*]?(sitio);
        sitio.analizar();
    }
}
```

### 2)En un laboratorio de genética veterinaria hay 3 empleados. El primero de ellos continuamente prepara las muestras de ADN; cada vez que termina, se la envía al segundo empleado y vuelve a su trabajo. El segundo empleado toma cada muestra de ADN preparada, arma el set de análisis que se deben realizar con ella y espera el resultado para archivarlo. Por último, el tercer empleado se encarga de realizar el análisis y devolverle el resultado al segundo empleado.

```java
Process Preparador {
    String muestra;
    while (true){
        muestra = prepararMuestra();
        Admin!enviar(muestra);
    }
}

Process Armador {
    String muestra;
    String set;
    String resultados;
    boolean listo = true;

    while (true){
        Admin!estoy(listo);
        listo = false;
        Admin?muestra(muestra);
        set = armarSet(prueba);
        Analizador!set(set);
        Analizador?resultados(resultados);
        archivar(resultados);
        listo = true;
    }
}

Process Analizador {
    String set;
    String resultados;
    while (true){
        Armador?set(set);
        resultados = analizar(set);
        Armador!resultados(resultados);
    }
}


Process Admin {
    cola buffer;
    String muestraAct;

    do Preparador?enviar(muestraAct) -> buffer.push(muestraAct);
    * !buffer.isEmpty(); Armador?estoy(listo) -> Armador!muestra(buffer.pop())
}
```

### 3)En un examen final hay N alumnos y P profesores. Cada alumno resuelve su examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes respectando el orden en que los alumnos van entregando.

#### a) Considerando que P=1.

```java
Process Admin {
    cola buffer;

    do Alumno?entregar(examen,id) -> buffer.push(examen,id);
    * !(buffer.isEmpty()); Profesor?pedir(); -> Profesor!corregir(buffer.pop());
}

Process Alumno[id:0..N-1]{
    text examen;
    int nota;

    examen = hacerExamen();
    Admin!entregar(examen,id);
    Profesor?recibir(nota);
}

Process Profesor{
    text examen;
    int nota;
    int idAux;

    while (true){
        Admin!pedir();
        Admin?corregir(examen,idAux);
        nota = corregirExamen(examen);
        Alumno[idAux]!recibir(nota);
    }
}

```

#### b) Considerando que P>1.

```java
Process Alumno[id:0..N-1]{
    text examen;
    int nota;

    examen = hacerExamen();
    Administrador!corregir(examen,id);
    Profesor[*]?recibir(nota);
}

Process Profesor[id:0..P-1] {

    while (true){
        text examen;
        int idAlumno;
        int nota;
        while (true){
            Administrador!libre(id);
            Administrador?corregir(examen,idAlumno);
            nota = corregirExamen(examen);
            Alumno[idAlumno]!recibir(nota);
        }
    }
}

Process Administrador{
    text examen;
    int idAlumno;
    int idProfesor;
    cola examenes(text,int);

    do Alumno[*]?corregir(examen,idAux) -> examenes.push(examen,idAlumno);
    * !empty(fila); Profesor[*]?libre(idProfesor) -> examenes.pop();
    Profesor[idProfesor]!corregir(examen,idAlumno);
    od
}
```

#### c) Ídem b) pero considerando que los alumnos no comienzan a realizar su examen hasta que todos hayan llegado al aula.

```java
Process Alumno[id:0..N-1]{
    text examen;
    int nota;

    Administrador!llegue();
    Administrador?comenzar();
    examen = hacerExamen();
    Administrador!corregir(examen,id);
    Profesor[*]?recibir(nota);
}

Process Profesor[id:0..P-1] {

    while (true){
        text examen; int idAlumno; int nota;
        while (true){
            Administrador!libre(id);
            Administrador?corregir(examen,idAlumno);
            nota = corregirExamen(examen);
            Alumno[idAlumno]!recibir(nota);
        }
    }
}

Process Administrador{
    text examen;
    int idAlumno;
    int idProfesor;
    cola examenes(text,int);
    int cant = 0;
    do Alumno[*]?llegue() -> {
        cant++;
        if (cant == N){
            for (int i=0;i < N;i++){
                Alumno[i]!comenzar();
            }
        }
    }
    * Alumno[*]?corregir(examen,idAux) -> examenes.push(examen,idAlumno);
    * !empty(fila); Profesor[*]?libre(idProfesor) -> examenes.pop();
      Profesor[idProfesor]!corregir(examen,idAlumno);
    od
}
```

### 4) En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con exclusión mutua) y un empleado encargado de administrar el uso del mismo. Hay P personas que esperan a que el empleado lo deje acceder al simulador, lo usa por un rato y se retira. El empleado deja usar el simulador a las personas respetando el orden de llegada. Nota: cada persona usa sólo una vez el simulador.

```java
Process Persona[id: 0..P-1]{
    Empleado!llegue(id);
    Empleado?usar();
    UsarSimulador();
    Empleado!terminar();
}

Process Empleado{
    cola c;
    boolean libre = true;
    int idAux;

    do (!libre); Persona[*]?llegue(idAux) -> c.push(idAux);
    * (libre && c.isEmpty()); Persona[*]?llegue(idAux) -> libre = false;
                                                        c[idAux]!usar();
    * (!c.isEmpty()); Persona[*]?terminar() -> idAux = c.pop();
                                                Persona[idAux]!usar();
    * (c.isEmpty()); Persona[*]?terminar() -> libre = true;
    od
}
```

### 5) En un estadio de fútbol hay una máquina expendedora de gaseosas que debe ser usada por E Espectadores de acuerdo al orden de llegada. Cuando el espectador accede a la máquina en su turno usa la máquina y luego se retira para dejar al siguiente. Nota: cada Espectador una sólo una vez la máquina.

```java
Process Espectador[id: 0..E-1]{
    Maquina!llegar(id);
    Maquina?turno();
    UsarMaquina();
    Maquina!salir();
}

Process Maquina{
    cola c;
    boolean libre = true;
    int idAux;

    do (!libre); Espectador[*]?llegar(idAux) -> c.push(idAux);
    * (libre && c.empty()); Espectador[*]?llegar(idAux) -> libre = false;
                                                        c[idAux]!turno();
    * (!c.empty()); Espectador[*]?salir() -> idAux = c.pop();
                                                    Espectador[idAux]!turno();
    * (c.empty()); Espectador[*]?salir() -> libre = true;
    od
}
```
