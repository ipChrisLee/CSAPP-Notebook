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

HCL description and notes: (the content in `<>` is pseudocode)

```verilog
bool instr_valid = <
	icode is invalid : 1;
    icode is not invalid : 0;
>
bool need_regids = icode in {IRRMOVQ, }
```

Align unit

