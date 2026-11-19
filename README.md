# cobos-post1-u3: Manejo del DEBUG en DOSBox

**Curso:** Arquitectura de Computadores (1154507-B), Universidad Francisco de Paula Santander
**Estudiante:** Isabella Cobos Lizarazo (1152507)
**Unidad 3:** Post-contenido. Exploración, ensamblado y ejecución paso a paso en DOSBox

Este repositorio documenta un laboratorio de dos partes realizado en DOSBox 0.74-3 con el depurador DEBUG. La **Parte 1** inspecciona el estado inicial del procesador y la memoria con los comandos R, F, D, E y U. Termina leyendo con direccionamiento directo a memoria (`MOV AX,[0300]`) un valor escrito antes con E. La **Parte 2** ensambla programas con A, los ejecuta instrucción a instrucción con T y compara un bucle con LOOP frente a su equivalente con DEC/JNZ, además del comando G como verificación rápida.

Todas las capturas se tomaron con la función de captura propia de DOSBox (Ctrl+F5), que guarda la pantalla emulada de 640×400 tal como se ve. Cuando una traza no cabe en las 25 líneas de la pantalla de texto, la captura del checkpoint reúne las pantallas consecutivas de la misma sesión, separadas por una franja gris.

### Estructura del repositorio

```
cobos-post1-u3/
├── README.md
└── capturas/
    ├── CP1_registros.png                 (Parte 1)
    ├── CP2_volcado_memoria.png           (Parte 1)
    ├── CP3_ensamblado_desensamblado.png  (Parte 1)
    ├── CP4_memoria_direccionamiento.png  (Parte 1)
    ├── CP1_traza_suma.png                (Parte 2)
    ├── CP2_traza_loop.png                (Parte 2)
    ├── CP2_codigo_maquina_loop.png       (Parte 2, Paso 8)
    ├── CP3_ensamblado_dec_jnz.png        (Parte 2, Paso 9)
    └── CP3_traza_dec_jnz.png             (Parte 2)
```

### Comandos DEBUG utilizados

| Comando | Uso en el laboratorio |
|---|---|
| `R`, `R AX`, `R IP` | Mostrar todos los registros, o mostrar y modificar uno |
| `F 200 L40 AB CD EF` | Rellenar un rango con un patrón cíclico |
| `D 200 L40`, `D 0 L20`, `D CS:100 L0C` | Volcar memoria en hexadecimal y ASCII |
| `E 300 78 56` | Escribir bytes puntuales |
| `A 100`, `A 200`, `A 320` | Ensamblar instrucciones directamente en memoria |
| `U 100 109`, `U 320 324`, ... | Desensamblar para verificar la codificación |
| `T` | Ejecutar una instrucción y mostrar el estado resultante |
| `G 20C` | Ejecutar a velocidad normal hasta un punto de interrupción |
| `Q` | Salir de DEBUG |

## Parte 1: Exploración con DEBUG en DOSBox

### Entorno utilizado

El laboratorio se ejecutó en DOSBox 0.74-3 sobre Windows, con la carpeta `C:\DOSWork` del equipo anfitrión montada como unidad `C:` virtual y el `DEBUG.EXE` ubicado en la raíz de esa carpeta. Antes de trabajar se cargó la distribución de teclado latinoamericana con `KEYB LA`, porque con la distribución española que DOSBox carga por defecto los caracteres `:`, `[` y `]` no se producían con las teclas esperadas.

```
Z:\> KEYB LA
Z:\> MOUNT C C:\DOSWork
Z:\> C:
C:\> MD LAB3POST1
C:\> CD LAB3POST1
C:\LAB3POST> DEBUG
```

El prompt muestra `C:\LAB3POST>` y no `C:\LAB3POST1>`. DOS solo admite nombres de directorio de hasta 8 caracteres más extensión (formato 8.3), así que DOSBox recorta `LAB3POST1` a `LAB3POST`. Por la misma razón, la Parte 2 se trabajó en `LAB3P2`: `LAB3POST2` también se habría recortado a `LAB3POST` y habría coincidido con el directorio de la Parte 1.

### Checkpoint 1: Estado de registros (comando R)

![CP1](capturas/CP1_registros.png)

| Registro | Valor inicial | Observación |
|---|---|---|
| AX, BX, CX, DX | 0000 | Registros de propósito general en cero |
| SP | FFFE | Tope inicial de la pila, al final del segmento de 64 KB |
| BP, SI, DI | 0000 | Sin uso al iniciar |
| DS, ES, SS, CS | 0724 | Los cuatro segmentos apuntan al mismo párrafo, el del PSP que asignó DOS |
| IP | 0100 | Primera dirección ejecutable después de los 256 bytes del PSP |
| Banderas | NV UP EI PL NZ NA PO NC | Todas en su estado de reposo; EI indica interrupciones habilitadas |

El segmento real asignado en esta sesión fue 0x0724, no 0x1357 como en la guía, porque el valor depende de cuánta memoria convencional ya ocupa el DOS emulado. La línea de desensamblado inicial mostró `0724:0100 C3 RET` en lugar de `CD20 INT 20`: al iniciar DEBUG sin archivo, la posición CS:0100 contiene lo que haya quedado en memoria. Después se ejecutó `R AX`, que mostró el valor actual `AX 0000` y aceptó `1234`. La segunda invocación de `R` confirmó que solo AX cambió a 1234 (4660 en decimal) y que el resto del estado del procesador permaneció idéntico.

### Checkpoint 2: Volcado hexadecimal con F y D

![CP2](capturas/CP2_volcado_memoria.png)

Con `F 200 L40 AB CD EF` se rellenaron 64 bytes desde DS:0200, y `D 200 L40` mostró el patrón repetido cíclicamente en las 4 filas. Como 64 no es múltiplo de 3, cada fila empieza en un byte distinto del patrón (AB, CD, EF, AB).

**Columnas de la salida de D.** La primera columna es la dirección lógica del primer byte de la fila en formato `segmento:desplazamiento` (por ejemplo `0724:0200`). La zona central contiene 16 bytes en hexadecimal, separados por un guion entre el octavo y el noveno para facilitar el conteo. La columna derecha muestra los mismos 16 bytes interpretados como ASCII, con un punto en lugar de cada byte no imprimible, que es la razón por la que AB, CD y EF aparecen como puntos. Cada fila avanza 0x10 en el desplazamiento, así que la dirección de cualquier byte se obtiene sumando su posición dentro de la fila a la dirección inicial.

Con `D 0 L20` se exploraron los primeros 32 bytes del PSP. Los dos primeros bytes son `CD 20`, la instrucción INT 20 que DOS coloca al inicio del PSP para que un programa COM que ejecute un RET con la pila vacía termine de forma controlada. Los demás bytes (`FF 9F` como tope de memoria disponible, y la tabla de manejadores de archivo `01 01 01 00 02 FF...` en 0x18) difieren de los de la guía porque reflejan la configuración concreta del DOS emulado.
