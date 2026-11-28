#### Useful instructions

`objdump -t bomb`
`objdump -d bomb`
`strings bomb`
`gdb bomb`

First thing I should do when running the `gdb` is to set break points for `explode_bomb` and different phases, and when reaching the start of a phase, use `disas` (disassemble) to checkout the assembler code of the corresponding function.

`info registers` to check value in registers.

`stepi` step one instruction.

# How does different instructions work?

## `PUSH`

Used to place a value onto the stack.
