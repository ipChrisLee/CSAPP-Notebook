# Architecture

## info

**Little-endian encoding** for Y86-64 (same to X86-64).

1 word = 8 bytes

Nouns:

* YIS (instruction set simulator)
* YAS (Y86-64 assembler) 



## Registers

<img src="ref_Y86-64/截屏2022-02-17 08.49.49.png" style="zoom:50%;" />

All registers have no fixed meanings or values, except for `%rsp`, which is used as a stack pointer.

NOTICE: The only `callee-saved` register is `%rsp`!

Register Encoding:

<img src="ref_Y86-64/截屏2022-02-17 09.11.57.png" style="zoom:50%;" />



## Instructions

<img src="ref_Y86-64/截屏2022-02-17 08.52.20.png" style="zoom:50%;" />

Notice for some instructions:

* `OPq`: Four integer operation instructions: 

  `addq`, `subq`, `andq`, `xorq`

  This instruction will set three condition codes `ZF`, `SF` and `OF`.

  NOTE:

  * The operand of `OPq` should **NOT** be immediate! So `addq 1,%rax` is invalid. To add an immediate to a argument, we need to load the constant in a register, and use `addq` between this register and the dest register.
  * Y86-64 doesn't has `test` instruction, so inorder to test a value, you should use `andq` instead.
  * Use `xorq` to set a register to 0 instead of using `mov $0`.

* `jXX`: Seven jump instructions: 

  * `jmp`: unconditional jump
  * `jle`, `jl`: conditional jump based on signed integer comparision
  * `je`, `jne`: conditional jump based on bit-level equality
  * `jge`, `jg`: conditional jump based on unsigned integer comparision

* `cmovXX`: Six conditional move instructions:

  `cmovle`, `cmovl`, `cmove`, `cmovne`, `cmovge`, `cmovg`

* **`comvXX` has SAME `icode` as `movq`!** The diff between them is `ifun`.

* So Y86-64 only supports **signed comparision**!

* `halt` causes the processor top stop, with status code set to `HLT`

* Encoding:

  <img src="ref_Y86-64/截屏2022-02-17 09.13.25.png" style="zoom:50%;" />

  `rrmovq` is viewed as 'unconditional move'.

* The `dest` used in `jXX` and `call` is absolute address.



## Exceptions

Y86-64 status codes:

<img src="ref_Y86-64/截屏2022-02-17 09.47.39.png" style="zoom:50%;" />

`HLT` indicates that the processor has executed a `halt` instructions.

Y86-64 has limited address range, and any access to an address beyond the limit will trigger an `ADR` exception.



## Directive

`.pos` tells the assembler to adjust the address at which it is generating code or to insert some words of data.

`.align` tells the assembler to align the data.



# Hardware Control Language HCL and Hardware Structure of Y86-64

## info

Three major components are required to implement a digital system: combinational logic, memory elements, clock signals.



## Syntax

### Operand and Operator

HCL has two types of operand: bool for `bit` level signal and `int` for word level signal.

Although HCL has two types of operand, we only use one type of operator, which means there is no 'bit and' and 'logical and' at all. For example, `bool eq=(a&&b)||(!a&&!b)` will set bit level signal `eq` to 1 if bit level signal `a` is equal to `b`, and set bit level signal `eq` to 0 if bit level signal `a` is not equal to `b`. `bool eq=(A==B)` will set `eq` to 1 if bit representation of word `A` is same to `B`, and to 0 if different.

Unlike C, operator `&&` and `||` don't have a property of short-circurt, to any level signal.

There are two special function called `MUX` (multiplexing) and `Set Membership`, which will be introduced later.



### `MUX`

The declarition form of  `mux`:

```verilog
word out=[
    select_1 : expr_1;
    select_2 : expr_2;
    ...
    select_k : expr_k;
]
```

The mux function decides the choice to selece **from top to ground**. So  `1` is always the last select choice.



### `Set Membership`

For the expression:

```verilog
bool s1 = code==2 || code==3;
bool s0 = code==1 || code==3;
```

We can write a more concise expression:

```verilog
bool s1 = code in {2,3};
bool s0 = code in {1,3};
```

General form:

```verilog
iexpr in {iexpr_1,iexpr_2, ..., iexpr_k}
```



## Six States of instruction execution and illustration

### Six States

There are some variables in register when executing instrutions. Here we will list them meanwhile.

1. Fetch

   * Variables:
     * `valC`: Constant word.
     * `valP`: Address of the instruction following the executing instruction.
   * Notice:
     * `valP` increases automatically in this state.

2. Decode

   * Variables:
     * `valA`: Value read from register `srcA`.
     * `valB`: Value read from register `srcB`.
   * Notice:
     * `valA` and `valB` are read simultaneously.

3. Execute

   * Variables:
     * `valE`: The value, which may be either the result of `OP` instruction or the effective address of a memory reference or the change of stack pointer, computed by ALU.
     * `Cnd`: Conditional judgement result.

4. Memory

   * Variables:
     * `valM`: The data read from memory.

5. Write Back

6. PC Update

   Update PC according to `Cond`



### Six states of all instructions

#### `OPq`, `rrmovq` and `irmovq`

<img src="ref_Y86-64/截屏2022-02-18 09.53.08.png" style="zoom:50%;" />

To notice that, because of hardware design, we can't directly move data from `valA` to `R[rB]`. In fact, the data is transferred from `valA` to `valE`, then to `R[rB]` from `valE`.



#### `rmmovq` and `mrmovq`

<img src="ref_Y86-64/截屏2022-02-18 09.54.12.png" style="zoom:50%;" />



#### `pushq` and `popq`

<img src="ref_Y86-64/截屏2022-02-18 09.54.59.png" style="zoom:50%;" />



#### `jXX`, `call` and `ret`

<img src="ref_Y86-64/截屏2022-02-18 09.55.41.png" style="zoom:50%;" />



### State Elements

There are four state elements: program counter, condition code regiseter, register file, and data memory.



# SEQ

## Hardware structure of SEQ

### Detailed view

<img src="ref_Y86-64/截屏2022-02-18 11.18.53.png" style="zoom:40%;" />

* Clocked registers are shown as white rectangles.

* Hardware units are shown as light blue boxes.

  We will see them as 'black boxes' now.

* Control logic blocks are drawn as gray rounded rectangles.

* Wire names are indicated in white circles.

* Word-wide data connections are shown as medium lines.

* Byte and narrower data connections are shown as thin lines.

* Single-bit connections are shown as dotted lines.

The combinational logic is the logic circuit. In our SEQ model, all the parts except for clocked registers (the program counter and condition code register) and random access memories (the register ﬁle, the instruction memory, and the data memory) are combinational logic.



### Computation steps in the sequential implementation

<img src="ref_Y86-64/截屏2022-02-18 11.24.42.png" style="zoom:50%;" />

It seems that the hardware executes the instruction same as the table above, but in fact it doesn't do like that.



### SEQ Timing

There are some facts that I think is important to understand timing of SEQ.

* The first and the most important fact you should know: You can read (from memory/locked-registers) and compute at ANYTIME, which just takes some time. But you can  write (to memory/locked-registers) ONLY when the rising edge of clock occurs, which takes very little time. The table below summarize this fact.

  | Event                                                        | When                         | Consuming of Time |
  | ------------------------------------------------------------ | ---------------------------- | ----------------- |
  | Read from memory/locked-registers<br />Compute in ALU<br />... | ANYTIME                      | some time         |
  | Write to memory/locked-registers                             | ONLY when rising edge occurs | very little time  |

* The second fact is: Digital signal is propagating in logic circuit ANYTIME. So any time you change the input of a logic circuit, the states connected to input will be changing in little time if there is no constraint such as clocked registers and memory.

  The ALU is a unit of logic circuit, and it is the best unit to show the fact: Anytime you change the input of ALU, the new result computed by ALU will be in output soon (not same time since it costs time for digital signal propagates in circuit).



So the time cycle of SEQ is:

<img src="ref_Y86-64/截屏2022-02-18 11.54.31.png" style="zoom:50%;" />

Before the rising edge at (1), the value cycle3 needed hasn't been written.
After the rising edge at (1), the value cycle3 needed has been written, and then cycle3 uses it to compute and generate new data.
Before the rising edge at (2), even if the value which cycle3 generated is ready, it can't be written.
After the rising edge at (2), the value which cycle3 generated is written.
The diagram below shows what I said in last four lines.

<img src="ref_Y86-64/截屏2022-02-18 12.23.17.png" style="zoom:30%;" />



So, in conclusion, we may think the six states of instruction is coming one by one. But in fact, the first four states (i.e. fetch, decode, execute and memory states) come continguously, and when rising edge occurs, the last two states (i.e. write back and PC update states) come.



## Detailed hardware structure and HCL description of SEQ

### Fetch Stage

Detailed hardware structure:

<img src="ref_Y86-64/截屏2022-02-18 12.58.43.png" style="zoom:50%;" />

#### Split Unit and Signals

The byte addressed by PC is called 'instruction byte', which will be splitted into `icode` and `ifun`.

If `imem_error` occurs, when instruction address is not valid, `icode` and `ifun` equals the byte value of `nop` instruction.

`icode` will set `instr_valid` if instruction is invalid (Q: Why needn't consider `ifun`), set `need_regids` if instruction needs register, set `need_valC` if instruction includes a constant value.

HCL description and notes: (the content in `<>` is pseudocode)

```verilog
bool instr_valid = <
	icode is invalid : 1;
    icode is not invalid : 0;
>
bool need_regids = icode in {IRRMOVQ, IOPQ, IPUSHQ, IPOPQ, IIRMOVQ, IRMMOVQ, IMRMOVQ}
bool need_valC = icode in {IIRMOVQ, IRMMOVQ, IMRMOVQ, IJXX, ICALL}
```



#### Align Unit and Signals

The `align` unit will output `valC`, according to `need_regids` signal. If `need_regids` is set, `align` will set `valC` to bytes 2-9. If `need_regids` is not set, `align` will set `valC` to bytes 1-8.

`rA` and `rB` are always set to the second byte of instruction, even if the instruction doesn't need register. Remind that if either `rA` or `rB` is not used by an instruction, the corrsponding half byte will set to 0xF, which means there is  no register in register file. For the instruction which needs no registers, the `rA` and `rB` will be set to the second byte, but will not affect the result because of the signal of `icode`, which we will talk about later.



#### PC increment

If PC equals `p` now, the generated PC value `valP` equals `p + 1 + need_regids + 8*need_valC `



### Decode and Write-Back Stages

Detailed hardware structure:

<img src="ref_Y86-64/截屏2022-03-01 08.46.33.png" style="zoom:50%;" />

#### Ports and Register Address

`register file` has two simultaneous reads and two simultaneous writes. (Remind: the read ports read from register file without considering the edge signal, and write ports write into registers only when rising edge occurs)

`srcA` and `srcB` are addresses for `valA` and `valB`. `dstE` and `dstM` are addresses for `valM` and `valE`. 

If we conclude decode and write back stages, `valE` is always related to `rB`, and `valM` is always related to `rA`. So we connect `dstE` with `rB` and `dstM` with `rA`. 

The `rA` and `rB` are not always needed for instruction. If such instruction doesn't need registers at all, `rA` and `rB` are meaningless value, and we shouldn't read from such register addresses. At this time, `icode` will make its sense.

```verilog
word srcA = [
    icode in {IRRMOVQ, IRMMOVQ, IOPQ, IPUSHQ, ICMOVXX} : rA;  // At these time, we need value stored in rA
    icode in {IPOPQ, IRET} : RRSP;  // At these time, we need value stored in %rsp.
    1 : RNONE;  // i.e. 0xF
] // NOTE: ICMOVXX has SAME icode as IRRMOVQ, and so it is considered into this HCL code.
```

Notice that, `rA` of `popq rA` is the address of the word read from memory, so `srcA` is not `rA` in this case.

```verilog
word srcB = [
    icode in {IRMMOVQ, IMRMOVQ, IOPQ} : rB;
    icode in {IRET, IPUSHQ, IPOPQ, ICALL} : RRSP;
    1 : RNONE;
]
```

`dstE` is the address of `valE`, and `valE` is always written into `rB`, so `dstE` is always:

```verilog
word dstE = [
    icode in {IRRMOVQ} && cnd : rB; 
    // cnd is generated in execute stage, and only if cnd is true, dstE equals rB.
    // Instruction cmovxx has same icode as movq, and the difference between them is the ifun.
    // They have identical instruction code IRRMOVQ!
    // Signal cnd will be generated according to ifun and cc. When ifun equals 0, cnd equals 1, and thus movq will execute correctly.
    icode in {IIRMOVQ, IOPQ} : rB;
    icode in {IPUSHQ, IPOPQ, ICALL, IRET} : RRSP;
    1 : RNONE;
]
```

`dstM` is the address of `valM`, and `valM` is always written into `rA`, so `dstM` is always:

```verilog
word dstM = [
    icode in {IMRMOVQ, IPOPQ} : rA;
    1 : RNONE;
]
```



Simultaneous writting will cause conflict when the dest registers are same. `popq %rsp` is the only instruction faces this conflict. And to solve this conflict, we give higher priority to port `valM` than that to port `valE`, which meets the fact that `popq %rsp` will increase `%rsp` first and then set `%rsp` to value read from memory.



### Execute Stage

<img src="ref_Y86-64/截屏2022-03-01 10.17.43.png" style="zoom:50%;" />

ALU (i.e. arithmetic/logic unit) performs caculation on inputs `aluA` and `aluB` based on `alufun` signal, resulting the output signal `valE`.

Concluding from the sequences of instructions, we can find that `aluA` can be `valA`, `valC`, `+8` or `-8`:

```verilog
word aluA = [
	icode in {IRRMOVQ, IOPQ} : valA;
    icode in {IIRMOVQ, IRMMOVQ, IMRMOVQ} : valC;
    icode in {ICALL, IPUSHQ} : -8;
    icode in {IRET, IPOPQ} : 8;  // Other instructions don’t need ALU.
]
```

`aluB` can be `valB` or `0`.

```verilog
word aluB = [
    icode in {IOPQ, IRMMOVQ, IMRMOVQ, IPUSHQ, IPOPQ, ICAL IRET} : valB;
    icode in {IRRMOVQ, IIRMOVQ} : 0;
]
```

`alufun` is `ifun` when `icode` is `IOPQ`, and `ALUADD` for other cases.

```verilog
word alufun = [
    icode == IOPQ : ifun;
    1 : ALUADD;
]
```

We only set conditional flags when executing `IOPQ` instructions, so:

```verilog
bool set_cc = icode in {IOPQ};
```



### Memory Stage

<img src="ref_Y86-64/截屏2022-03-01 10.43.03.png" style="zoom:50%;" />

```verilog
word mem_addr = [
    icode in {IRMMOVQ, IPUSHQ, ICALL, IMRMOVQ} : valE;
    icode in {IPOPQ, IRET} : valA;
    // Other instructions don’t need address
]
word mem_data = [
    icode in {IRMMOVQ, IPUSHQ} : valA;
    icode in {ICALL} : valP;
    // Other instructions don’t write anything.
]
bool mem_read = icode in {IMRMOVQ, IPOPQ, IRET};
bool mem_write = icode in {IRMMOVQ, IPUSHQ, ICALL};
word stat = [
    imem_error || dmem_error : SADR;  // address exception
    !instr_valid : SINS; // illegal instruction exception
    icode == == IHALT : SHLT; // halt
    1 : SAOK; // normal
] // Q: How to decide priority here?
```



### PC Update Stage

<img src="ref_Y86-64/截屏2022-03-01 10.53.43.png" style="zoom:50%;" />

```verilog
word new_pc = [
	// Call. Use instruction constant
    icode == ICALL : valC; 
    // Taken branch. Use instruction constant
    icode == IJXX && Cnd : valC; 
    // Completion of RET instruction. Use value from stack
    icode == IRET : valM; 
    // Default: Use incremented PC
    1 : valP;
];
```



# SEQ+

## intro

### circuit retiming

Retiming changes the state representation for a system without changing its logical behavior.

In our changing of SEQ to SEQ+, we move PC-update stage to the most beginning of the clock cycle. And we remove PC register in SEQ+, which is neeadless because now PC is generated by a logical circuit.



### hardware structure

<img src="ref_Y86-64/截屏2022-03-02 09.14.27.png" style="zoom:50%;" />



# PIPE-

## hardware structure

<img src="ref_Y86-64/截屏2022-03-02 09.16.46.png" style="zoom:50%;" />

(This structure doesn't need PC register neither.)

### intro

Here we have five stages: F, D, E, M and W. Every blue box indicates pipeline registers for every stage.

The signals named as `X_reg` are signals stored in a pipeline register `reg` which is before `X` stage. The signals named as `y_reg` are signals computed within stage `y`.

### diff from SEQ+

#### `Select A`

We merge signal `valA` with `valP`. Only `call` and `jxx` need `valP` after decode stage, but neither of them need `valA` to store the value read from register file. 

This unit eliminates the need for `Data` block in SEQ and SEQ+ since they have same similar purpose.

#### branch prediction `Predict PC`

For pipeline system, because of **control dependencies**, we need to determine which instruction to execute next before the previous instruction finishes executing. 

The stragegy we use here is **always taken(AT)**, and learn about other branch prediction strategies in CS:APP page 428. AT means we always take the branch one to execute. For example, `valC` is considered as next `PC` for `call` and `jxx`, and `valP` is considered as next `PC` for other instructions (For `ret` instruction, although there is practical technique for return address prediction, we will not attempt to predict any value.).



## hazards

### Data Hazards caused by Data Dependency

Our pipeline system design above can't handle problems of data dependency at all. And here we will try to solve this problem. 

Notice that, according to the aside in CS:APP P435, we only need to deal with register data harzards.

Two solutions:

#### by Stalling

See figure:

<img src="ref_Y86-64/截屏2022-03-02 10.44.05.png" style="zoom:50%;" />

Three `bubble` 'instructions' is injected before `addq` instruction. Every `bubble` is injected by suspended `addq` instruction which is stalled in D stage. 

Note: the instructions after stalled instruction will be stalled too.

The `bubble` takes corresponding effect as `nop` instruction.



#### by Forwarding: PIPE

The hardware structure of PIPE:

<img src="ref_Y86-64/截屏2022-03-02 11.43.36.png" style="zoom:50%;" />

The key units are `Sel+Fwd A` and `Fwd B`. See figure:

<img src="ref_Y86-64/截屏2022-03-02 11.47.01.png" style="zoom:50%;" />

`addq` instruction can get value directly from `M_valE` and `e_valE`, which means `addq` doesn't need to be stalled for write-back of `irmovq`!



#### by load interlock: Fix load/use data hazard

<img src="ref_Y86-64/截屏2022-03-02 11.52.19.png" style="zoom:50%;" />

Because memory stage is behind execute stage and address `valE` of load is calculated by ALU in execute stage, the instruction followed load instruction should be stalled until the execute stage of load instruction finishes and get register value then.



### Control Hazards caused by Control Dependency

Our pipeline system design will face problems when prediction of PC is incorrect, which may happen when executing `ret` and `jxx` instructions.

So we can just solve this issue calssified by instruction:

Case of `ret`:

<img src="ref_Y86-64/截屏2022-03-02 12.08.38.png" style="zoom:50%;" />

Case of `jxx`:

<img src="ref_Y86-64/截屏2022-03-02 12.08.52.png" style="zoom:50%;" />

NOTICE: `jxx` decide if the branch is taken in execute stage by updating `cnd` signal. So we can judge if the predection is correct just after the execute stage of `jxx`.





