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
