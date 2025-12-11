# 💻 Processador 4-bit — Visão Geral e Especificação

Este documento detalha o formato das instruções do processador de 4-bit, o mapeamento de bits, os códigos da ULA (Unidade Lógico-Aritmética) e as instruções de acesso à memória (LOAD, STORE, MOV).

## 1. 🚀 Visão Geral Rápida

O processador utiliza **instruções de 16 bits** e opera em um conjunto de **8 registradores (R0 a R7)**.

* **Tamanho da Instrução:** 16 bits.
* **Formato de Entrada:** Binário (16 bits), frequentemente representado em **Hexadecimal** para facilitar a leitura.
* **Formatos Principais:** Definidos pelos bits de Opcode (`OP`, bits 15–14).

| OP (b15–14) | Formato de Instrução | Descrição |
| :---: | :---: | :--- |
| `00` | **ULA (ALU)** | Operações aritméticas, lógicas, shift, multiplicação. |
| `11` | **MOV / RAM** | Instruções de mover valor imediato (`MOV immediate`), carregar (`LOAD`), e armazenar (`STORE`). |

## 2. 🧩 Mapeamento de Bits (Formato Geral — 16 bits)

A indexação é feita do bit 15 (Mais Significativo — MSB) ao bit 0 (Menos Significativo — LSB).

| Campo | Bits | Descrição |
| :---: | :---: | :--- |
| **OP** | 15–14 | Opcode principal (define o formato da instrução). |
| **sub-op / ULA** | 13–11 | Sub-operação (para MOV/RAM) ou Código da Operação da ULA (para ULA). |
| **Reservado** | 10–9 | Não usado / Reservado (`00`). |
| **RA / imm / addr** | 8–6 | Registro Fonte A (ULA), ou parte de immediate/endereço (MOV/RAM). |
| **RB / imm / addr** | 5–3 | Registro Fonte B (ULA), ou parte de immediate/endereço (MOV/RAM). |
| **RD / reg\_addr** | 2–0 | Registro Destino (ULA), ou Registro/Endereço (MOV/RAM). |



### 2.1. Formato ULA (ALU) — quando `OP = 00`

Usado para todas as operações da Unidade Lógico-Aritmética.

| Bits | Campo | Largura | Descrição |
| :---: | :---: | :---: | :--- |
| 15–14 | **OP** | 2 | `00` (ULA) |
| 13–11 | **ULA_op** | 3 | Código da operação ULA (Ex.: ADD, SUB, MULT, SHIFT). |
| 10–9 | **Reservado** | 2 | `00` |
| 8–6 | **RA** | 3 | Registro Fonte A (`R0`–`R7`). |
| 5–3 | **RB** | 3 | Registro Fonte B (`R0`–`R7`). |
| 2–0 | **RD** | 3 | Registro Destino (`R0`–`R7`). |

**Resumo:** `00 | ULA_op(3) | 00 | RA(3) | RB(3) | RD(3)`

### 2.2. Formato MOV / RAM — quando `OP = 11`

Usado para transferência de dados (imediato) e acesso à memória RAM.

| Bits | Campo | Largura | Descrição |
| :---: | :---: | :---: | :--- |
| 15–14 | **OP** | 2 | `11` (MOV/RAM) |
| 13–11 | **sub-op** | 3 | Define MOV immediate, LOAD, STORE, etc. |
| 10–7 | **Reservado** | 4 | `0000` (Inferido a partir dos exemplos) |
| 6–3 | **Immediate / Endereço** | 4 | Valor imediato (MOV) ou Endereço/Dado (LOAD/STORE). |
| 2–0 | **Reg Destino / Endereço** | 3 | Registro Destino (MOV) ou Endereço de RAM/Registrador (LOAD/STORE). |

**Resumo:** `11 | sub-op(3) | 00 | imm/addr(4) | reg_addr(3)`

---

## 3. 🔢 Códigos de Operação da ULA (`OP = 00`)

Os seguintes códigos de 3 bits definem as operações da ULA (bits 13–11), inferidos a partir dos exemplos fornecidos:

| ULA Opcode (bin) | Hex (Exemplo) | Operação | Descrição |
| :---: | :---: | :---: | :--- |
| `000` | `0044` | **ADD** | $R_{dest} \leftarrow R_A + R_B$ |
| `001` | `0906` | **SUB** | $R_{dest} \leftarrow R_A - R_B$ |
| `101` | `284D` | **SHL** | **SHIFT LEFT:** $R_{dest} \leftarrow R_A \ll R_B$ (quantidade) |
| `110` | `304B` | **SHR** | **SHIFT RIGHT:** $R_{dest} \leftarrow R_A \gg R_B$ (quantidade) |
| `111` | `3847` | **MULT** | $R_{dest} \leftarrow R_A \times R_B$ |
| `outros` | — | N/A | (Necessário documentação completa se existirem) |

---

## 4. 💾 Sub-opcodes MOV / RAM (`OP = 11`)

Os códigos de 3 bits (bits 13–11) definem o tipo de instrução de acesso a dados ou memória.

| Sub-op (bin) | Hex (Exemplo) | Significado (Inferido) | Comportamento |
| :---: | :---: | :---: | :--- |
| `000` | `C019` | **MOV Immediate** | Move o valor de 4 bits (bits 6–3) para o Registrador Destino (bits 2–0). Ex: `R1 ← 3`. |
| `001` | `C804` | **LOAD** | Carrega o conteúdo de RAM na posição (endereço) para o Registrador Destino. Ex: `R_dest ← RAM[4]`. |
| `110` | `F019` | **STORE** | Armazena um valor na RAM. (Conforme sua nota: armazena **3** (do `imm` 4-bit) na RAM na **posição do contador/endereçador**). |

> **Observação sobre STORE:** O exemplo `F019` armazena o valor `3` (bits 6–3) na RAM na posição indicada pelo contador (ou registrador endereçador, conforme sua implementação).

---

## 5. 💡 Tabela de Conversão Binário ↔ Hexadecimal e Decodificação

Esta tabela mostra a decodificação de instruções de 16 bits, dos exemplos fornecidos:

| Binário (16 bits) | Hex | Interpretação | Ação (Mnemônico) |
| :---: | :---: | :---: | :--- |
| `1100 0000 0001 1001` | `C019` | OP=11, sub=000 (MOV imm), imm=3, reg=R1 | `R1 ← 3` |
| `1100 1000 0000 0100` | `C804` | OP=11, sub=001 (LOAD), addr=4 | `R_dest ← RAM[4]` |
| `0000 0000 0100 0100` | `0044` | OP=00, ULA=000 (ADD), RA=R1, RB=R0, RD=R4 | `R4 ← R1 + R0` |
| `0000 1001 0000 0110` | `0906` | OP=00, ULA=001 (SUB), RA=R4, RB=R0, RD=R6 | `R6 ← R4 - R0` |
| `0011 1000 0100 0111` | `3847` | OP=00, ULA=111 (MULT), RA=R1, RB=R0, RD=R7 | `R7 ← R1 × R0` |
| `0011 0000 0100 1011` | `304B` | OP=00, ULA=110 (SHR), RA=R1, RB=1, RD=R3 | `R3 ← R1 >> 1` |
| `1111 0000 0001 1001` | `F019` | OP=11, sub=110 (STORE), imm=3 | `RAM[Contador] ← 3` (Conforme sua nota) |

---

