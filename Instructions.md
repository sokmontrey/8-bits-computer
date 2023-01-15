# 8-bit computer instructions

## LOAD_INS
Load instruction code (OP Code) from the INS_RAM to INS_REG. This assume that the pointer is at the instruction location.

Requirement: N/A
LOAD_INS: 0000 0000

0. Load pointer from PROG_CNT to MEM_REG
1. Load instruction code from RAM to INS_REG
2. Increment PROG_CNT, reset STEP

### Pin Control
0. PROG_CNT_OE, MEM_REG_WE
1. RAM_OE, INS_REG_WE
2. PROG_CNT_INCR, STEP_RESET

## LOAD_D/I_A/B
Load an 8bit data from Data RAM / Instruction RAM at a certain memory address (ADDR) to A_REG / B_REG (depend on the instruction code).

Requirement: ADDR

LOAD_D_A: 0011 0001
LOAD_D_B: 0010 0001
LOAD_I_A: 0001 0001
LOAD_I_B: 0000 0001

1. Load pointer from PROG_CNT to MEM_REG
2. Load ADDR from RAM to MEM_REG
3. Load 8bit from RAM to X_REG
	1. if D: enable RAM_D_SE
	2. if A: enable X_REG_A_SE
4. Increment PROG_CNT, reset INS_REG, reset STEP

### Pin Control
1. PROG_CNT_OE, MEM_REG_WE
2. RAM_OE, MEM_REG_WE
3. RAM_OE, REG_WE
	1. if D: RAM_D_SE
	2. if A: REG_A_SE
4. PROG_CNT_INCR, INS_REG_RESET, STEP_RESET

## LOAD_IM_A/B
Load an 8bit data provided (VALUE) to A_REG / B_REG (spend on the instruction code).

Requirement: VALUE

LOAD_IM_A: 0001 0010
LOAD_IM_B: 0000 0010

1. Load pointer from PROG_CNT to MEM_REG
2. Load VALUE from RAM to X_REG
	1. if A: enable X_REG_A_SE
3. Increment PROG_CNT, reset INS_REG, reset STEP

### Pin Control
1. PROG_CNT_OE, MEM_REG_WE
2. RAM_OE, REG_WE
	1. if A: REG_A_SE
3. PROG_CNT_INCR, INS_REG_RESET, STEP_RESET

## LOAD_C_A/B
Load 8bit data from C_REG to A_REG / B_REG

Requirement: N/A

LOAD_C_A: 0001 0011
LOAD_C_B: 0000 0011

1. Load 8bit from C_REG to X_REG
	1. if A: enable X_REG_A_SE
2. Increment PROG_CNT, Reset INS_REG, reset STEP
	
### Pin Control
1. C_REG_OE, REG_WE
	1. if A: REG_A_SE
2. PROG_CNT_INCR, INS_REG_RESET, STEP_RESET

## ADD/SUBSTRACT
Add A_REG with B_REG / Substract B_REG **from** A_REG, then store the result in C_REG.

Requirement: N/A

ADD: 0000 0100
SUBSTRACT: 0100 0100

1. Load 8bit from ALU to C_REG
	1. if SUBSTRACT: enable ALU_SUB_EN
2. Reset INS_REG, reset STEP

### Pin Control
1. ALU_OE, C_REG_WE
	1. if SUBSTRACT: SUB_EN
2. INS_REG_RESET, STEP_RESET

## SAVE_C_D/I
Save 8bit data in C_REG to Data RAM / Instruction RAM with the address provided (ADDR).

Requirement: ADDR

SAVE_C_D: 0010 0101
SAVE_C_I: 0000 0101

1. Load pointer form PROG_CNT to MEM_REG
2. Load ADDR from RAM to MEM_REG
3. Load 8bit from C_REG to RAM
	1. if D: enable RAM_D_SE
4. Increment PROG_CNT, reset INS_REG, reset STEP

### Pin Control
1. PROG_CNT_OE, MEM_REG_WE
2. RAM_OE, MEM_REG_WE
3. C_REG_OE, RAM_WE
	1. if D: RAM_D_SE
4. PROG_CNT_INCR, INS_REG_RESET, STEP_RESET

## JUMP
Change pointer to the given address (ADDR).

Requirement: ADDR

JUMP: 0000 0110

1. Load pointer from PROG_CNT to MEM_REG
2. Load ADDR from RAM to PROG_CNT
3. Reset INS_REG, reset STEP

### Pin Control
1. PROG_CNT_OE, MEM_REG_WE
2. RAM_OE, PROG_CNT_LOAD
3. INS_REG_RESET, STEP_RESET

## JUMP_EQ
Change pointer to the given address (ADDR) if A_REG == B_REG or (A_REG - B_REG == 0)

Requirement: ADDR

JUMP_EQ: 0000 0111

1. Enable ALU_SUB_EN, enable FLAG_EN 
2. If IS_ZERO_FLAG is 0 (result is not 0):
	1. Increment PROG_CNT, reset FLAG, reset INS_REG, reset STEP
3. If IS_ZERO_FLAG is 1 (result is all 0):
	1. Load pointer from PROG_CNT to MEM_REG
	2. Load ADDR from RAM to PROG_CNT
	3. Reset FLAG, reset INS_REG, reset STEP
	
### Pin Control
1. SUB_EN, FLAG_EN
2. if IS_ZERO_FLAG is 0:
	1. PROG_CNT_INCR, FLAG_RESET, INS_REG_RESET, STEP_RESET
3. if IS_ZERO_FLAG is 1:
	1. PROG_CNT_OE, MEM_REG_WE
	2. RAM_OE, PROG_CNT_LOAD
	3. FLAG_RESET, INS_REG_RESET, STEP_RESET

## JUMP_MORE
Change pointer to the given first address (ADDR1) if A_REG > B_REG, and to the given second address (ADDR2) if A_REG < B_REG.

Requirement: ADDR1 (if true), ADDR2 (if false)

JUMP_MORE: 1000 0111

1. Enable ALU_SUB_EN, enable FLAG_EN
2. If IS_POSSITIVE_FLAG is 0 (A_REG < B_REG):
	1. Increment PROG_CNT
	2. Load pointer from PROG_CNT to MEM_REG
	3. Load ADDR2 from RAM to PROG_CNT
	4. Reset INS_REG, reset STEP, reset FLAG
3. if IS_POSSITIVE_FLAG is 1 (A_REG > B_REG):
	1. Load pointer from PROG_CNT to MEM_REG
	2. Load ADDR1 from RAM to PROG_CNT
	3. Reset INS_REG, reset STEP, reset FLAG
	
### Pin Control
1. SUB_EN, FLAG_EN
2. if IS_POSSITIVE_FLAG is 0:
	1. PROG_CNT_INCR
	2. PROG_CNT_OE, MEM_REG_WE
	3. RAM_OE, PROG_CNT_LOAD
	4. INS_REG_RESET, STEP_RESET, FLAG_RESET
3. if IS_POSSITIVE_FLAG is 1:
	1. PROG_CNT_OE, MEM_REG_WE
	2. RAM_OE, PROG_CNT_LOAD
	3. INS_REG_RESET, STEP_RESET, FLAG_RESET

## HALT
Stop the clock

Requirement: N/A

HALT: 1111 1111

1. Enable CLOCK_STOP (maybe)

### Pin Control

1. CLK_STOP
