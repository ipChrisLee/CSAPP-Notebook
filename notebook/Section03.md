# `Useful Note`

Here are some useful notes, such as 'size of C data types in x86-84'. 

## `About instruction`

### `Sizes of C data types in x86-84`

<img src="ref/截屏2021-12-28 11.04.59.png" style="zoom:50%;" />



### `Integer registers`

<img src="ref/截屏2021-12-28 11.06.07.png" style="zoom:50%;" />



### `Operand Forms`

<img src="ref/截屏2021-12-28 10.59.36.png" style="zoom:50%;" />



### 



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



