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
