ESPECIFICACIÓN TÉCNICA: SIMULADOR DE CPU DE 8 BITS
Asignatura: Arquitectura de Computadoras

Docente: Ing. Loayza

Arquitectura de Referencia: Modelo von Neumann (8 bits)

Proyecto: Sistema de Simulación, Ejecución Paso a Paso y Monitoreo de Hardware

1. Visión General del Sistema
El presente proyecto implementa un simulador funcional de una Unidad Central de Procesamiento (CPU) de 8 bits basado en la arquitectura von Neumann. El sistema integra el mapeo físico de memoria RAM (256 bytes), un conjunto de registros internos, una Unidad Aritmético Lógica (ALU) con gestión de banderas, un decodificador de instrucciones (ISA) y la máquina de estados finitos que ejecuta el ciclo de instrucción de 4 fases (Fetch, Decode, Execute, Store).

Adicionalmente, el sistema expone un panel de auditoría de micro-operaciones que permite inspeccionar la transferencia de datos entre buses, registros y celdas de memoria en tiempo real durante cada ciclo de reloj.

2. Arquitectura de Hardware y Diagrama de Bloques
Bajo el paradigma von Neumann, tanto las instrucciones del programa como los datos procesados comparten el mismo bus y el mismo espacio de direccionamiento en la memoria principal.
graph TD
    subgraph CPU [Núcleo de Procesamiento CPU]
        PC[Program Counter - PC] --> MAR[Memory Address Register - MAR]
        IR[Instruction Register - IR] --> UC[Unidad de Control - UC]
        UC --> ALU[Unidad Aritmético Lógica - ALU]
        AX[Acumulador General - AX] <--> ALU
        BX[Registro Auxiliar - BX] <--> ALU
        ALU --> Flags[Banderas: ZF, CF, SF]
    end

    subgraph Memory [Memoria Principal RAM]
        RAM[Memoria RAM de 256 Bytes: 00h - FFh]
    end

    MAR --> RAM
    RAM <--> MDR[Memory Data Register - MDR]
    MDR <--> IR
    MDR <--> AX
    MDR <--> BX
Implementación Primitiva de Lectura y Escritura
const RAM = new Uint8Array(256);

function Read(address) {
  if (address < 0x00 || address > 0xFF) {
    throw new Error("Acceso a memoria fuera del rango permitido (00h-FFh)");
  }
  return RAM[address];
}

function Write(address, value) {
  if (address < 0x00 || address > 0xFF) {
    throw new Error("Acceso a memoria fuera del rango permitido (00h-FFh)");
  }
  RAM[address] = value & 0xFF; // Máscara para garantizar 8 bits
}
3. Mapeo y Segmentación de Memoria RAM
El espacio de direccionamiento del sistema abarca un total de 256 bytes direccionables mediante un bus de direcciones de 8 bits (rango hexadecimal 00h a FFh).
Rango de Memoria,Tamaño,Segmento,Descripción y Uso Principal
00h - 7Fh,128 Bytes,Segmento de Código,Reservado para la carga secuencial de opcodes y operandos inmediatos/direcciones del programa.
80h - FFh,128 Bytes,Segmento de Datos,"Reservado para el almacenamiento dinámico de variables, acumulación de resultados y contadores."
4. Registros del Procesador y Banderas de la ALU
4.1 Registros Internos de Procesamiento
Registro,Tamaño,Tipo,Función Principal
PC,8 Bits,Control,Program Counter: Dirección de la siguiente instrucción a ejecutar. Incrementa automáticamente.
IR,8 Bits,Control,Instruction Register: Almacena el opcode de la instrucción actual traído desde la RAM.
MAR,8 Bits,Memoria,Memory Address Register: Dirección física que se desea colocar en el bus de memoria.
MDR,8 Bits,Memoria,Memory Data Register: Buffer bidireccional para datos leídos o por escribir en la RAM.
AX,8 Bits,Datos,Acumulador General: Registro principal para operaciones aritméticas y lógica.
BX,8 Bits,Datos,Registro Auxiliar: Registro secundario para operandos y almacenamiento de soporte.
4.2 Banderas de Estado de la ALU (Flags)
Bandera,Nombre,Condición de Activación (1)
ZF,Zero Flag,Se activa si el resultado de la última operación en la ALU es exactamente igual a 0x00.
CF,Carry Flag,Se activa si ocurrió un desbordamiento superior (> 255) o subdesbordamiento (< 0).
SF,Sign Flag,"Refleja el bit más significativo (Bit 7), indicando un valor negativo en complemento a dos."
5. Conjunto de Instrucciones (ISA - Instruction Set Architecture)
El procesador implementa un conjunto de instrucciones reducidas (RISC de 8 bits) parametrizado bajo la siguiente especificación:
Mnemónico,Opcode (Hex),Tamaño,Descripción Funcional
"MOV AX, imm",0x10,2 Bytes,Carga un byte inmediato directamente en el registro AX.
"MOV BX, imm",0x11,2 Bytes,Carga un byte inmediato directamente en el registro BX.
"LOAD AX, [dir]",0x20,2 Bytes,Lee el byte en la dirección dir de la RAM y lo almacena en AX.
"LOAD BX, [dir]",0x21,2 Bytes,Lee el byte en la dirección dir de la RAM y lo almacena en BX.
"STORE [dir], AX",0x30,2 Bytes,Escribe el valor actual de AX en la dirección dir de la RAM.
"STORE [dir], BX",0x31,2 Bytes,Escribe el valor actual de BX en la dirección dir de la RAM.
"ADD AX, imm",0x40,2 Bytes,"Suma un valor inmediato a AX y actualiza banderas (ZF, CF, SF)."
"SUB AX, imm",0x50,2 Bytes,"Resta un valor inmediato a AX y actualiza banderas (ZF, CF, SF)."
INC AX,0x60,1 Byte,Incrementa el registro AX en 1 unidad.
DEC AX,0x70,1 Byte,Decrementa el registro AX en 1 unidad y actualiza banderas.
"CMP AX, imm",0x80,2 Bytes,Compara AX con un inmediato (resta interna) y actualiza banderas.
JMP dir,0x90,2 Bytes,Modifica el PC para forzar un salto incondicional a dir.
JZ dir,0xA0,2 Bytes,Bifurcación condicional a dir si la bandera ZF está activa (1).
JNZ dir,0xB0,2 Bytes,Bifurcación condicional a dir si la bandera ZF está inactiva (0).
HLT,0xFF,1 Byte,Detiene el ciclo de ejecución del procesador.
6. Motor del Ciclo de Instrucción (4 Fases)
La ejecución de cada instrucción se divide strictly en 4 fases secuenciales sincronizadas:
Fase,Nombre,Descripción Técnica de Micro-operaciones
1,Fetch,"Transfiere la dirección desde PC hacia MAR, ejecuta la lectura en RAM hacia MDR y carga el opcode en IR. Incrementa PC."
2,Decode,Evalúa el opcode almacenado en IR para determinar si requiere lectura de operandos adicionales sobre el bus de memoria.
3,Execute,La Unidad de Control habilita los caminos de datos en la ALU o efectúa la actualización del registro PC en caso de saltos (JMP/JZ/JNZ).
4,Store,"Consolida la actualización de registros de destino (AX, BX) o la escritura en celdas de RAM (STORE)."
7. Programa de Prueba: Multiplicación por Sumas Sucesivas
Para validar el comportamiento del procesador frente a estructuras de control iterativas (bucles) y saltos condicionales, se carga en el segmento de código el programa para resolver la multiplicación de 3 x 4:
Dirección,Opcode / Datos,Ensamblador,Descripción Técnica
0x00,10 00,"MOV AX, 00h",Inicializa el acumulador AX en 0
0x02,11 03,"MOV BX, 03h",Carga el multiplicando (3d) en BX
0x04,30 80,"STORE [80h], AX",Guarda resultado parcial en RAM[80h]
0x06,10 04,"MOV AX, 04h",Carga contador de iteraciones (4d) en AX
0x08,20 80,"LOAD AX, [80h]",[Inicio Bucle]: Carga parcial desde RAM[80h]
0x0A,40 03,"ADD AX, 03h",Suma el multiplicando (AX + 3)
0x0C,30 80,"STORE [80h], AX",Guarda nuevo parcial en RAM[80h]
0x0E,10 04,"MOV AX, 04h",Carga el contador actual
0x10,70,DEC AX,Decrementa el contador en 1
0x11,30 81,"STORE [81h], AX",Actualiza contador en RAM[81h]
0x13,B0 08,JNZ 08h,"Evalúa ZF: Si ZF == 0, salta a 0x08"
0x15,FF,HLT,Parada: Resultado 12d (0Ch) en RAM[80h]
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
El proyecto se gestionó bajo la metodología Kanban mediante GitHub Projects y control de versiones semántico con Git:
Estado Kanban,ID Tarea,Nombre de la Tarea / Issue,Vinculación Git / Commit
Done,#1,feat: IMPLEMENTAR LA ARQUITECTURA Y MAPEO DE LA MEMORIA RAM DE 256 BYTES (00H-FFH),closes #1
Done,#2,feat: Estructura de registros del CPU y banderas de estado ALU,closes #2
Done,#3,feat: Especificación del conjunto de instrucciones ISA y opcodes,closes #3
Done,#4,feat: Motor del ciclo de instrucción completo en 4 fases,closes #4
In Progress,#5,test: Programa demostrativo de multiplicación por sumas sucesivas,closes #5
In Progress,#6,ui: Consola de log y registro de micro-operaciones en tiempo real,closes #6
In Progress,#7,test: Pruebas de integración QA y validación de banderas de la ALU,closes #7
In Progress,#8,docs: Documentación técnica README.md y arquitectura en Mermaid,closes #8
To Do,#9,docs: Estructura y guion estratégico para la defensa oral,Pendiente
To Do,#10,docs: Cierre del proyecto y auditoría de historial de commits,Pendiente
