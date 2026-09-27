# Simulador de CPU de 8 Bits - Arquitectura von Neumann

Este proyecto consiste en la implementación de un simulador didáctico de un procesador de 8 bits basado en la arquitectura von Neumann. El sistema incluye el mapeo completo de memoria RAM de 256 bytes, la estructura de registros internos, el conjunto de instrucciones (ISA), el motor del ciclo de instrucción dividido en 4 fases y un panel de auditoría de micro-operaciones en tiempo real.

---

## 1. Arquitectura del Sistema

El procesador opera bajo un esquema von Neumann donde las instrucciones y los datos comparten el mismo espacio de memoria principal (RAM de 256 bytes).

```mermaid
graph TD
    subgraph CPU [Núcleo CPU de 8 Bits]
        PC[Program Counter - PC] --> MAR[Memory Address Register - MAR]
        IR[Instruction Register - IR] --> UC[Unidad de Control - UC]
        UC --> ALU[Unidad Aritmético Lógica - ALU]
        AX[Acumulador - AX] <--> ALU
        BX[Registro Auxiliar - BX] <--> ALU
        ALU --> Flags[Banderas ZF, CF, SF]
    end

    subgraph Memory [Memoria Principal]
        RAM[Memoria RAM 256 Bytes: 00h - FFh]
    end

    MAR --> RAM
    RAM <--> MDR[Memory Data Register - MDR]
    MDR <--> IR
    MDR <--> AX
2. Mapa de Memoria RAM
La memoria RAM de 256 bytes está mapeada dentro del rango hexadecimal 00h a FFh:

00h - 7Fh: Segmento de Código (almacenamiento de opcodes y operandos de programas).

80h - FFh: Segmento de Datos (almacenamiento de variables, acumuladores y resultados parciales).

3. Registros del Procesador y Banderas
Registros Internos
PC (Program Counter - 8 bits): Contiene la dirección de memoria de la siguiente instrucción a ejecutar.

IR (Instruction Register - 8 bits): Almacena la instrucción leída desde la RAM durante la fase de Fetch.

MAR (Memory Address Register - 8 bits): Sostiene la dirección física de la RAM a la que se desea acceder.

MDR (Memory Data Register - 8 bits): Registro intermedio de datos leídos o por escribir en la RAM.

AX (Acumulador General - 8 bits): Registro principal para operaciones aritméticas y lógicas de la ALU.

BX (Registro Auxiliar - 8 bits): Registro secundario de uso general.

Banderas de Estado (ALU)
ZF (Zero Flag): Se establece en 1 si el resultado de la última operación fue cero (0x00).

CF (Carry Flag): Se establece en 1 si existió desbordamiento superior (> 255) o inferior (< 0).

SF (Sign Flag): Se establece en 1 si el bit más significativo (Bit 7) del resultado está activo.
4. Conjunto de Instrucciones (ISA)MnemónicoOpcode (Hex)LongitudDescripciónMOV AX, imm0x102 BytesCarga un valor inmediato de 8 bits en el registro AXMOV BX, imm0x112 BytesCarga un valor inmediato de 8 bits en el registro BXLOAD AX, [dir]0x202 BytesLee el contenido de la memoria en dir y lo guarda en AXLOAD BX, [dir]0x212 BytesLee el contenido de la memoria en dir y lo guarda en BXSTORE [dir], AX0x302 BytesEscribe el valor actual de AX en la dirección dir de la RAMSTORE [dir], BX0x312 BytesEscribe el valor actual de BX en la dirección dir de la RAMADD AX, imm0x402 BytesSuma un valor inmediato a AX y actualiza banderasSUB AX, imm0x502 BytesResta un valor inmediato a AX y actualiza banderasINC AX0x601 ByteIncrementa AX en 1DEC AX0x701 ByteDecrementa AX en 1 y actualiza banderasCMP AX, imm0x802 BytesCompara AX con un valor inmediato actualizando banderasJMP dir0x902 BytesSalto incondicional a la dirección especificadaJZ dir0xA02 BytesSalto condicional si la bandera Zero Flag (ZF) es 1JNZ dir0xB02 BytesSalto condicional si la bandera Zero Flag (ZF) es 0HLT0xFF1 ByteDetiene la ejecución del ciclo de reloj5. Ciclo de Instrucción (4 Fases)El motor del simulador procesa cada instrucción dividiéndola estrictamente en 4 fases de reloj:Fetch (Búsqueda): Transfiere el contenido de PC a MAR, lee el opcode de la RAM hacia MDR y lo carga en IR. Incrementa PC.Decode (Decodificación): Evalúa el opcode en IR y determina si requiere leer un segundo byte de operando desde RAM.Execute (Ejecución): Realiza la operación lógica/aritmética correspondiente en la ALU o efectúa la bifurcación de control.Store (Almacenamiento): Escribe el resultado final en el registro destino (AX/BX) o en la RAM mediante MAR y MDR.
6. Programa de Prueba: Multiplicación por Sumas Sucesivas
El repositorio incluye un programa cargado en RAM a partir de la dirección 00h que calcula el producto de 3 x 4:
Dirección | Opcode / Datos | Código Ensamblador  | Comentario
--------------------------------------------------------------------------------------
0x00      | 10 00          | MOV AX, 00h         | Inicializa resultado acumulado en 0
0x02      | 11 03          | MOV BX, 03h         | Carga el multiplicando (3)
0x04      | 30 80          | STORE [80h], AX     | Guarda parcial en RAM[80h]
0x06      | 10 04          | MOV AX, 04h         | Carga contador (4)
0x08      | 20 80          | LOAD AX, [80h]      | Inicio del bucle: Cargar parcial
0x0A      | 40 03          | ADD AX, 03h         | Suma el multiplicando (+3)
0x0C      | 30 80          | STORE [80h], AX     | Guarda resultado parcial
0x0E      | 10 04          | MOV AX, 04h         | Recupera valor de iteraciones
0x10      | 70             | DEC AX              | Decrementa el contador en 1
0x11      | 30 81          | STORE [81h], AX     | Guarda contador actualizado
0x13      | B0 08          | JNZ 08h             | Salta a 0x08 si el contador != 0
0x15      | FF             | HLT                 | Fin. Resultado 12d (0Ch) en RAM[80h]
7. Estructura de Control y Metodología del Proyecto
Este proyecto se desarrolló aplicando la metodología Kanban vinculando los commits del repositorio directamente a los Issues de GitHub Projects:

Gestión de Tareas: Tablero Kanban con flujo To Do, In Progress y Done.

Commits Semánticos: Enlazados atómicamente a cada tarea mediante referencias closes #N.
8. Formato de Auditoría y Log de Micro-operaciones
El simulador genera trazas de auditoría por consola durante el procesamiento paso a paso:
[CLOCK 01] FETCH   -> MAR: 0x00 | MDR: 0x10 | IR: MOV AX, imm | PC: 0x01
[CLOCK 02] DECODE  -> Leyendo operando inmediato: 0x00 desde RAM[0x01] | PC: 0x02
[CLOCK 03] EXECUTE -> AX: 0x00 | Flags -> ZF: 1, CF: 0, SF: 0
[CLOCK 04] FETCH   -> MAR: 0x0A | MDR: 0x40 | IR: ADD AX, 0x03 | PC: 0x0B
[CLOCK 05] EXECUTE -> ALU ADD -> Operandos: (0x00 + 0x03) | Resultado: 0x03
[CLOCK 06] STORE   -> AX: 0x03 | Flags -> ZF: 0, CF: 0, SF: 0
[CLOCK 07] FETCH   -> MAR: 0x13 | MDR: 0xB0 | IR: JNZ 0x08 | PC: 0x14
[CLOCK 08] EXECUTE -> Evaluación Banderas: ZF=0. Condición cumplida -> Modificando PC = 0x08
9. Metodología de Desarrollo e Integración Continua
El proyecto fue gestionado aplicando la metodología agil Kanban vinculada con el control de versiones Git/GitHub:

Gestión de Tareas: Tablero Kanban con trazabilidad en tres estados (To Do, In Progress, Done).

Commits Semánticos: Enlazados atómicamente a cada Issue del repositorio utilizando referencias closes #N para garantizar el registro de progreso requerido en el proceso de evaluación.
