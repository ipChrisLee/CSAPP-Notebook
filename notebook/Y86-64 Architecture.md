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





