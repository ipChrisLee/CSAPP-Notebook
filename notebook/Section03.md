# `Useful Note`

Here are some useful notes, such as 'size of C data types in x86-84'. 

## `About instruction`

### `Sizes of C data types in x86-84`

<img src="ref/截屏2021-12-28 11.04.59.png" style="zoom:50%;" />



### `Integer registers`

<img src="ref/截屏2021-12-28 11.06.07.png" style="zoom:50%;" />



### `Operand Forms`

<img src="ref/截屏2021-12-28 10.59.36.png" style="zoom:50%;" />





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

Except 'leaq' instruction, which will not change ANY condition codes, the instructions above will set condition codes (CF,ZF,SF,OF).

For the logical operations, the CF and OF are set to zero, the ZF is set iff the result is 0, the SF is set iff the highest bit of result is 1.



## mul and div instructions

<img src="ref/截屏2022-01-18 16.26.29.png" style="zoom:50%;" />
