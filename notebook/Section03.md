# `Useful Note`

Here are some useful notes, such as 'size of C data types in x86-84'. 

## `About instruction`

### `Sizes of C data types in x86-84`

<img src="ref/截屏2021-12-28 11.04.59.png" style="zoom:50%;" />



## Registers

### Integer Registers

<img src="ref/截屏2021-12-28 11.06.07.png" style="zoom:50%;" />

`%rbx`, `%rbp` and `%r12-%r15` (six registers in sum)are classified as **callee-saved** registers. The callee MUST preserve the values of these registers, which means, they have the same value when callee ends and returns to caller as when callee is called.Ohter registers, except `%rsp`, are classified as **caller-saved** registers, since is caller's duty to save the values of these registers.





### Some special registers

`%rsp`：stack pointer register, points to the top element of the stack. We can treat it as integer register, `addq $16,%rsp`, for example.

`%rip`：(not listed above) instuction pointer register, or PC.



### arguments register

<img src="ref/截屏2022-02-12 10.11.49.png" style="zoom:50%;" />

With x86-64, up to six integral arguments can be passed via registers. The registers are used in a specified order.



### `Operand Forms`

<img src="ref/截屏2021-12-28 10.59.36.png" style="zoom:50%;" />

`$Imm` use decimal number.



# `Program Encodings`

## `How a .o file generated?`

We can use GCC to generate a excutive '.o' file without any other file by `gcc test.c -o test`.

But from .c file to .o file, there are many procedures that we should focus on. So it's important to know how to generate intermedia files by GCC.

1. Preprocess:(from .c to .i by preprocessor)

   ```shell
   gcc -E test.c -o test.i
   ```

2. compile:(from .i to .s by compiler)

   ```shell
   gcc -S test.i -o test.s
   ```

3. assemble:(from .s to .o by assembler)

   ```shell
   gcc -c test.i -o test.o  # use -c not -C if code is not completed.
   ```

4. link:(from .s to executive .o by linker)

   ```shell
   gcc -C test.i -o test.o
   ```



# `instructions`

Here are lists of instructions seperated by instruction classes memtioned in CSAPP.

## `mov` class

<img src="ref/截屏2022-01-18 16.25.09.png" style="zoom:50%;" />

If the destination of movl is a register, the high 32 bits of the register would be set to 0. So in fact, 'movl \$0, %edx' has same effect as 'movq \$0, %rdx'.



## `movz` class

zero extension

<img src="ref/截屏2022-01-18 16.25.23.png" style="zoom:50%;" />



## `movs` class

sign extension

<img src="ref/截屏2022-01-18 16.25.35.png" style="zoom:50%;" />



## stack related instructions

<img src="ref/截屏2022-01-18 16.25.53.png" style="zoom:50%;" />



## arithmetic instructions

<img src="ref/截屏2022-01-18 16.26.05.png" style="zoom:50%;" />

The instructions listed above are 'instrucion class' in fact, except 'leaq' instruction. For example, to left shift %cl, which is 8 bit length,  with 4 bits, you should use 'salb %cl,4'. 

The SUB here means the two-complement subtraction. 

For Flag-Bits:

Except 'leaq' instruction, which will not change ANY condition codes, the instructions above will set condition codes (CF,ZF,SF,OF).

For the logical operations, the CF and OF are set to zero, the ZF is set iff the result is 0, the SF is set iff the highest bit of result is 1.

For the shift operations, the CF is set to the last bit shifted out, while the OF is set to zero, the ZF and SF perform same to logical case.

The INC and DEC set the OF, ZF and SF, but leave the CF unchanged.



## mul and div instructions

<img src="ref/截屏2022-01-18 16.26.29.png" style="zoom:50%;" />



## comparision and test instructions

<img src="ref/截屏2022-02-05 17.45.38.png" style="zoom:50%;" />

The CMP instructions behave in the same way as the SUB instructions, except that they set the condition codes without updating their destinations.

The TEST instructions behave in the same way as the ADD instructions, except that they set the condition codes without updating their destinations.

To TEST a single variable, use `testq %rsi,%rsi`.

## set instructions

<img src="ref/截屏2022-02-05 17.55.19.png" style="zoom:50%;" />

A SET instruction has either one of **the low-order single-byte register elements** or **a single-byte memory location** as its destination.

So to generate a 32-bit or 64-bit result, we must also clear the high-order bits.



## jump instructions

<img src="ref/截屏2022-02-05 18.49.43.png" style="zoom:50%;" />

When the instruction `cmp $1,$2` followed by `jg J`, if `$2<$1`, it will jump to `J`. So, the `cmp $1,$2` means compare `$2` to `$1`.



# conditional move instructions

Conditional transfer of data.

<img src="ref/截屏2022-02-11 10.20.25.png" style="zoom:50%;" />

Let `R=S` iff condition is true.

The bit wide is implicit. But single-byte conditional moves are not supported.



## procedure control transfer

<img src="ref/截屏2022-02-12 09.19.30.png" style="zoom:50%;" />

When P calls Q, `call` will pushes an address A onto the stack and sets the PC to the beginning of Q. A is referred to as the `return address` and is computerd as the address of the instruction immediately following the `call` instruction.

The `ret` instruction will pop an address A off the stack and sets the PC to A.

