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

### Checkpoint 3: Ensamblado con A y desensamblado con U

![CP3](capturas/CP3_ensamblado_desensamblado.png)

Primero se ejecutó `U 100 L10` sobre la memoria sin inicializar. DEBUG interpretó como instrucciones bytes que no eran código (por ejemplo `INSB` y `DB 66`), lo que confirma que en modo real no hay distinción entre código y datos: el procesador ejecuta lo que encuentre en CS:IP.

Después se ensambló el programa con `A 100` y se verificó con `U 100 109`:

| Dirección | Bytes | Instrucción | Análisis |
|---|---|---|---|
| 0100 | B8 05 00 | MOV AX,0005 | Opcode B8 + inmediato 0x0005 en little-endian |
| 0103 | BB 03 00 | MOV BX,0003 | Opcode BB + inmediato 0x0003 |
| 0106 | 01 D8 | ADD AX,BX | Opcode 01 /r con ModR/M D8 |
| 0108 | CD 20 | INT 20 | Opcode CD + número de interrupción |

El programa ocupa 10 bytes, igual que en la guía. La diferencia está en `ADD AX,BX`: el DEBUG de esta instalación lo codificó como `01 D8` y no como `03 C3`. Las dos codificaciones son válidas y equivalentes. En x86 la suma registro-registro tiene dos opcodes, 01 (ADD r/m16, r16) y 03 (ADD r16, r/m16), que difieren solo en el bit de dirección. Con `01 D8`, el campo reg de ModR/M (11 011 000) es BX, la fuente, y el campo r/m es AX, el destino. Con `03 C3` los papeles se invierten. El resultado de la ejecución es idéntico, y U lo confirma al mostrar en ambos casos el mismo mnemónico `ADD AX,BX`.

### Checkpoint 4: Modificación de memoria con E y direccionamiento directo

![CP4](capturas/CP4_memoria_direccionamiento.png)

Se limpió la región con `F 300 L10 00`, se verificó con `D 300 L10` (16 bytes en 00) y se escribieron dos bytes con `E 300 78 56`. El segundo `D 300 L10` mostró `78 56 00 00 ...`, con la columna ASCII `xV` porque 0x78 es "x" y 0x56 es "V". Leídos como palabra little-endian, esos dos bytes representan 0x5678.

Luego se ensambló `MOV AX,[0300]` en 0320 con `A 320`, y `U 320 324` confirmó la codificación `A1 00 03`. Con `R IP` se fijó IP en 0320 y se ejecutó `T`. El resultado fue **AX=5678** e IP avanzó a 0323, sobre el `INT 20`. Esto verifica que la instrucción leyó de memoria el valor escrito antes con E.

### Decisión técnica (Paso 12): verificación no destructiva de una escritura en memoria

De los cuatro comandos disponibles, el adecuado para confirmar la escritura del Paso 11 es D. Invocar `E 300` sin lista de bytes no es seguro, porque DEBUG entra en modo interactivo: muestra el valor actual de cada byte y espera una tecla. Cualquier dígito hexadecimal tecleado por error reemplaza el contenido, y la verificación misma se convierte en una nueva escritura. En el Paso 11 E fue apropiado porque recibió la lista completa `78 56` y su intención explícita era modificar. F tampoco sirve para verificar, aunque opere sobre un rango igual que D: su único efecto es escribir un patrón, así que usarlo destruiría precisamente los bytes que se quieren comprobar. R solo muestra o cambia registros y no permite observar DS:0300. D, en cambio, solo lee memoria y la presenta en hexadecimal y ASCII sin alterar ningún byte ni registro. Esa propiedad de solo lectura es la que exige una verificación: el acto de observar no puede cambiar lo observado.

### Decisión técnica (Paso 14): direccionamiento inmediato frente a directo a memoria

Aunque `MOV AX,0005` (B8 05 00) y `MOV AX,[0300]` (A1 00 03) ocupan los mismos 3 bytes, el direccionamiento directo a memoria requiere un ciclo adicional de acceso al bus. En el modo inmediato, el valor 0x0005 viaja dentro del flujo de instrucción y llega al procesador en la misma búsqueda (fetch) que trae el opcode. En el modo directo, los dos bytes que siguen al opcode no son el dato sino una dirección: el procesador forma la dirección física DS×16+0300, en este caso 0x07540, y realiza una lectura adicional para traer la palabra almacenada allí. Aun así, el direccionamiento directo es preferible cuando el objetivo es leer un valor que puede cambiar en tiempo de ejecución, como los bytes 78 56 escritos con E en el Paso 11, que ninguna constante ensamblada podría anticipar. Para saber qué codificación produjo cada comando A no hace falta ejecutar nada: basta con `U 100 102` y `U 320 322`, que muestran los opcodes B8 y A1 junto a su operando y revelan si este es un valor o una dirección.
