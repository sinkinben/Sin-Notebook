## GDB Usage

This article is a notebook for myself to remember the usage of GDB.

## breakpoint

1. Set breakpoint at a function

   ```bash
   b <function name>
   ```

2. Show all the breakpoints you have set

   ```bash
   info b
   ```

3. Delete a breakpoint

   ```
   d [breakpoint order]
   ```

   The `breakpoint order` can get by `info b`  .



## info command

1. Show the registers

   ```bash
   info r
   ```

2. 



## x command

1. Show instruction

   ```bash
   x /[n]i $eip
   ```

   This command will show n instructions starting `$eip`. `$eip` can be other addresses. 

2. Show a string

   ```bash
   x /s [address]
   ```

3. Show a hex number

   ```bash
   x /x [address]
   ```



## How to exec

1. single step: `si`
2. jump out of a function: `finish`

