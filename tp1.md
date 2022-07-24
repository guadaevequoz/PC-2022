## Práctica 1 - Variables Compartidas

### 1) Para el siguiente programa concurrente suponga que todas las variables están inicializadas en 0 antes de empezar. Indique cual/es de las siguientes opciones son verdaderas:

#### a) En algún caso el valor de x al terminar el programa es 56.

#### b) En algún caso el valor de x al terminar el programa es 22.

#### c) En algún caso el valor de x al terminar el programa es 23.

```
* P1 -> P2 -> P3 = 10 -> 11 -> 56
La opción verdadera es la A.
```

### 2) Realice una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema. Dado un numero N verifique cuantas veces aparece ese número en un arreglo de longitud M.

```java
int array[N];
int cont = 0;
process A[i:0..N-1]{
    if(array[i] == N)
        <cont++>;
}
```

### 3) Indicar si el siguiente código funciona para resolver el problema de Productor/Consumidor con un buffer de tamaño N. En caso de no funcionar, debe hacer las modificaciones necesarias.

#### a) Indicar si el siguiente código funciona para resolver el problema de Productor/Consumidor con un buffer de tamaño N. En caso de no funcionar, debe hacer las modificaciones necesarias.

```java
int cant = 0; int pri_ocupada = 0; int pri_vacia = 0; int buffer[N];
Process Productor::
{ while (true)
 { produce elemento
 <await (cant < N); cant++>
 buffer[pri_vacia] = elemento;
 pri_vacia = (pri_vacia + 1) mod N;
 }
}

Process Consumidor::
{ while (true)
 { <await (cant > 0); cant-- >
 elemento = buffer[pri_ocupada];
 pri_ocupada = (pri_ocupada + 1) mod N;
 consume elemento
 }
}
```

```
No funciona porque cuando ingresa no protege la variable compartida buffer. El codigo corregido es:
```

```java
int cant = 0; int pri_ocupada = 0; int pri_vacia = 0; int buffer[N];
Process Productor::
{ while (true)
 { //produce elemento
 <await (cant < N); cant++>
 <buffer[pri_vacia] = elemento;>
 pri_vacia = (pri_vacia + 1) mod N;
 }
}

Process Consumidor::
{ while (true)
 { <await (cant > 0); cant-- >
 <elemento = buffer[pri_ocupada];>
 pri_ocupada = (pri_ocupada + 1) mod N;
 //consume elemento
 }
}
```

#### b) Modificar el código para que funcione para C consumidores y P productores.

```java
int cant = 0; int pri_ocupada = 0; int pri_vacia = 0; int buffer[N];
Process Productor[i:0..P-1]
{ while (true)
 { //produce elemento
 <await (cant < N); cant++>
 <buffer[pri_vacia] = elemento;>
 pri_vacia = (pri_vacia + 1) mod N;
 }
}

Process Consumidor[i:0..C-1]
{ while (true)
 { <await (cant > 0); cant-- >
 <elemento = buffer[pri_ocupada];>
 pri_ocupada = (pri_ocupada + 1) mod N;
 //consume elemento
 }
}

```

### 4) Realice una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema. Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola, cuando un proceso necesita usar una instancia del recurso la saca de la cola, la usa y cuando termina de usarla la vuelve a depositar.

```java
cola c[5];
int cant = 5;

Process Proceso[i:0..N-1]{
    <await(cant > 0); cant--; recurso =  c.pop();>
    //usar recurso
    <c.push(recurso)>;
    <cant++>;
}
```

### 5) En cada ítem debe realizar una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema, teniendo en cuenta las condiciones indicadas en el item. Existen N personas que deben imprimir un trabajo cada una.

#### a) Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas.

```java
boolean impresoraLibre = true;
Process Persona[i:0..N-1]{
    <await(impresoraLibre); impresoraLibre = false;>
    Imprimir(documento);
    impresoraLibre=true;
}
```

#### b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.

```java
boolean impresoraLibre = true;
Process Persona[i:0..N-1]{
    <await(impresoraLibre); impresoraLibre = false;>
    Imprimir(documento);
    impresoraLibre=true;
}
```

#### c) Modifique la solución de (a) para el caso en que se deba respetar el orden dado por el identificador del proceso (cuando está libre la impresora, de los procesos que han solicitado su uso la debe usar el que tenga menor identificador).

#### d) Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la impresora hasta que no haya terminado de usarla la persona X-1).

#### e) Modifique la solución de (c) para el caso en que además hay un proceso Coordinador que le indica a cada persona cuando puede usar la impresora.

### 6) Resolver con SENTENCIAS AWAIT (<> y/o <await B; S>) el siguiente problema. En un examen final hay P alumnos y 3 profesores. Cuando todos los alumnos han llegado comienza el examen. Cada alumno resuelve su examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes respectando el orden en que los alumnos van entregando.

```java
cola qEntregas;
int notas[P]=([P] = -1), cantAlumnos = 0, cantEntregas = 0;

Process Alumnos[id:0..P-1]{
    examen e;
    <cantAlumnos++>;
    <await(cantAlumnos == p)>;
    e.hacerExamen();
    <qEntregas.push(e); cantEntregas++>;
    <await(notas[id] <> -1)>;
}

Process Profesores[id:0..2]{
    examen e;
    while(true){
        <await(!qEntregas.isEmpty()); e = qEntregas.pop()>;
        notas[e.id] = e.corregir();
    }
```

### 7) Dada la siguiente solución para el Problema de la Sección Crítica entre dos procesos (suponiendo que tanto SC como SNC son segmentos de código finitos, es decir que terminan en algún momento), indicar si cumple con las 4 condiciones requeridas:

```java
int turno = 1;
Process SC1::
{
    while (true)
    {
        while (turno == 2) skip;
        SC;
        turno = 2;
        SNC;
        }
}

Process SC2::
{
    while (true)
    {
        while (turno == 1) skip;
        SC;
        turno = 1;
        SNC;
        }
}

```

```
Si, cumple con las 4 condiciones.
```
