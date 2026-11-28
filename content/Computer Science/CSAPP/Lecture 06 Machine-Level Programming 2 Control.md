#### Processor State (x86-64, Partial)

Information about currently executing program

- Temporary data `(%rax, ...)`
- Location of runtime stack `(%rsp)`
- Location of current code control point `(%rip, ...)`
- Status of recent tests `(CF, ZF, SF, OF)`

#### Conditional Codes (Implicit Setting)

##### Single Bit Registers

- CF Carry Flag (for unsigned)
- SF Sign Flag (for signed)
- ZF Zero Flag
- OF Overflow Flag (for signed)

##### Implicitly set by arithmetic operations

Example: `addq Src, Dest` which works like `t = a + b`
**CF set** if carry out from most significant bit (unsigned overflow)
**ZF set** if `t == 0`
**SF set** if `t < 0` (as signed)
**OF set** if two's-complement (signed) overflow

##### Explicit Setting by Compare Instruction

`cmpq Src2, Src1`
`cmpq b, a` like computing `a - b` without setting a destination

##### Explicit Setting by Test Instruction

`testq Src2, Src1`
`testq b, a` is like computing `a & b`

#### Loop Translation

##### While translation

`While` version

```
while (Test)
	Body
```

`Goto` version

```
	goto test;
loop:
	Body
test:
	if (Test)
		goto loop;
done:
```

##### For loop translation

First translate it into while form and then use the while translation.
