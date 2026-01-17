# MN-Core Tutorial

After confirming that commands like `assemble3 --help` work in your environment, write the following code to an appropriate file (e.g. sample.vsm).

```
d get $lm0n0c0b0m0p0 1
```

Once the file is ready, execute the following command.

```sh
$ assemble3 sample.vsm > sample.asm
```

If there are no problems up to this point, try running sample.asm with the emulator.

```sh
$ gpfn3_package_main -i sample.asm -d dump.txt
$ cat dump.txt
```

If you get output like the following, your execution environment is working correctly.

```
DEBUG-LM0(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x0}}, v:0x0) #d get $lm0n0c0b0m0p0 1
```

In the following text, unless otherwise noted, code blocks containing `d get` are executable vsm (MN-Core assembly), and the output from dump.txt obtained by executing them using the same procedure as above is shown in the immediately following code block.

Text after `#` in vsm is a comment.

## Execution on 1PE (1/4096 MN-Core2)

- Items added in this section
  - Debug set/get
  - Arithmetic and Logic Unit (ALU)
  - LM0/LM1
  - GRF0/GRF1
  - T register
  - Mask register
  - Forwarding-path

### d set/get

The `<raw-payload>` handled by `d set/get` is specified in long word units (64bit). When using `d set` for word (32bit) or half-word (16bit) values, you need to add the remaining 32bit or 48bit elements.

Write a single double-precision 1.0 (0x3FF0000000000000) to address 0 of LM0 located at n0c0b0m0p0:

```
d set $lm0n0c0b0m0p0 1 3FF0000000000000
# $lm0: LM0 ($ln => LM1, $lr => GRF0, $ls => GRF1)
#    ^: address
# n0c0b0m0p0
#  ^        : group-id [0,4)
#    ^      : l2b-id [0,2)
#      ^    : l1b-id [0,8)
#        ^  : mab-id [0,16)
#          ^: pe-id [0,4)
d getd $lm0n0c0b0m0p0 1
#    ^ d: double
```

```
DEBUG-LM0(n0c0b0m0p0,0):(1) (0x3ff0000000000000) #d getd $lm0n0c0b0m0p0 1
```

Write two float-precision 1.0 (0x3F800000) to address 0 of LM0 located at n0c0b0m0p0:

```
d set $lm0n0c0b0m0p0 1 3F8000003F800000
d getf $lm0n0c0b0m0p0 1
#    ^ f: float
```

```
DEBUG-LM0(n0c0b0m0p0,0):(1, 1) (0x3f800000, 0x3f800000) #d getf $lm0n0c0b0m0p0 1
```

Write four half-precision 1.0 (0x3e00) to address 0 of LM0 located at n0c0b0m0p0:

```
d set $lm0n0c0b0m0p0 1 3e003e003e003e00
d geth $lm0n0c0b0m0p0 1
#    ^ h: half
```

```
DEBUG-LM0(n0c0b0m0p0,0):(1, 1, 1, 1) (0x3e00, 0x3e00, 0x3e00, 0x3e00) #d geth $lm0n0c0b0m0p0 1
```

When specifying multiple payloads with `d set/get`, note that the minimum unit of address increment is a word. The `l` immediately after `$` specifies access in long word units, and removing it results in word specification.
note: For long word specification, addresses must be aligned to multiples of 2. For example, $lm1 is invalid, so the emulator exhibits undefined behavior.

```
Data layout on LM0:  3FF00000_00000000_3FF00000_00000000
    Long word spec: |      $lm0       |      $lm2       |
         Word spec: |  $m0   |  $m1   |  $m2   |  $m3   |
```

Write two double-precision 1.0:

```
d set $lm0n0c0b0m0p0 2 3FF00000000000003FF0000000000000
d getd $lm0n0c0b0m0p0 2
```

```
DEBUG-LM0(n0c0b0m0p0,0):(1) (0x3ff0000000000000) #d getd $lm0n0c0b0m0p0 2
DEBUG-LM0(n0c0b0m0p0,2):(1) (0x3ff0000000000000) #d getd $lm0n0c0b0m0p0 2
                     ^: The next address is 2
```

When handling integers, precision specification is not required for `d get`:

```
d set $lm0n0c0b0m0p0 1 0042004200000042
d get $lm0n0c0b0m0p0 1
```

```
DEBUG-LM0(n0c0b0m0p0,0):(f:2.00268e-307, i:{{0x42,0x42},{0x0,0x42}}, v:0x42004200000042) #d get $lm0n0c0b0m0p0 1
```

Locations other than LM0 can also be targets of `d set/get` (omitting `d set`).

```
d get $ln0n0c0b0m0p0 1
d get $lr0n0c0b0m0p0 1
d get $ls0n0c0b0m0p0 1
d get $ltn0c0b0m0p0 1
d get $omr1n0c0b0m0p0 1
```

```
      vvv: This part displays the location
DEBUG-LM1(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x0}}, v:0x0) #d get $ln0n0c0b0m0p0 1
DEBUG-GREG0(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x0}}, v:0x0) #d get $lr0n0c0b0m0p0 1
DEBUG-GREG1(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x0}}, v:0x0) #d get $ls0n0c0b0m0p0 1
DEBUG-TREG(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x0}}, v:0x0) #d get $ltn0c0b0m0p0 1
      vvv: Mask registers output 4 cycles (=1 step) together
DEBUG-OMR(n0c0b0m0p0,1):Mask{0} #d get $omr1n0c0b0m0p0 1
DEBUG-OMR(n0c0b0m0p0,1):Mask{0} #d get $omr1n0c0b0m0p0 1
DEBUG-OMR(n0c0b0m0p0,1):Mask{0} #d get $omr1n0c0b0m0p0 1
DEBUG-OMR(n0c0b0m0p0,1):Mask{0} #d get $omr1n0c0b0m0p0 1
```

### Characteristics of each operand

LM0/LM1 (2048 long words): Represented as $lm/$ln in vsm respectively. Designed as storage locations for long-lived variables, and in exchange for large capacity, with some exceptions, only one of Read or Write can be performed in the same step.

```
d set $lm0n0c0b0m0p0 1 0000000000000042

linc $lm0 $ln0

d get $ln0n0c0b0m0p0 1

# It takes 2 steps before what was written can be read. (optional: what happens if you reduce the number of nops...?)
# This is a constraint on actual hardware, and the internal state of the emulator is updated immediately, so the updated value can be read exceptionally via `d get`.
# Also, only for LM, reads are not possible for 2 steps after writing, regardless of address.
nop
nop
linc $ln0 $lm2

d get $lm2n0c0b0m0p0 1

nop/2
# ladd $ln0 $lm2 $ln2 # <= error
ladd $ln0 $lm2 $ln0   # <= valid (exception where instruction bit collision does not occur)

d get $ln0n0c0b0m0p0 1
```

```
DEBUG-LM1(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x43}}, v:0x43) #d get $ln0n0c0b0m0p0 1
DEBUG-LM0(n0c0b0m0p0,2):(f:0, i:{{0x0,0x0},{0x0,0x44}}, v:0x44) #d get $lm2n0c0b0m0p0 1
DEBUG-LM1(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x87}}, v:0x87) #d get $ln0n0c0b0m0p0 1
```

GRF0/GRF1 (256 long words): Represented as $lr/$ls in vsm respectively. These are designed as storage locations for short-lived variables, and Read and Write can be performed simultaneously in the same step.

```
d set $lr0n0c0b0m0p0 1 0000000000000002
d set $ls0n0c0b0m0p0 1 0000000000000005

linc $lr0 $lr0
linc $ls0 $ls2

d get $lr0n0c0b0m0p0 1
d get $ls2n0c0b0m0p0 1

nop
# Only 1 step has passed since writing to $ls2, but reading from a different address is possible. (Not possible for LM)
ladd $lr0 $ls0 $lr2

d get $lr2n0c0b0m0p0 1

nop/2
ladd $lr2 $ls2 $lr4 $ls4

d get $lr4n0c0b0m0p0 1
d get $lr4n0c0b0m0p0 1
```

```
DEBUG-GREG0(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x3}}, v:0x3) #d get $lr0n0c0b0m0p0 1
DEBUG-GREG1(n0c0b0m0p0,2):(f:0, i:{{0x0,0x0},{0x0,0x6}}, v:0x6) #d get $ls2n0c0b0m0p0 1
DEBUG-GREG0(n0c0b0m0p0,2):(f:0, i:{{0x0,0x0},{0x0,0x8}}, v:0x8) #d get $lr2n0c0b0m0p0 1
DEBUG-GREG0(n0c0b0m0p0,4):(f:0, i:{{0x0,0x0},{0x0,0xE}}, v:0xE) #d get $lr4n0c0b0m0p0 1
DEBUG-GREG0(n0c0b0m0p0,4):(f:0, i:{{0x0,0x0},{0x0,0xE}}, v:0xE) #d get $lr4n0c0b0m0p0 1
```

T (normally 4 long words, maximum 4\*double long words): Represented as $t($lt,$llt) in vsm. Can be treated the same as GRF except for the inability to specify addresses and capacity, and also has special uses such as T register indirect reference. Word length cannot be specified, but it is automatically determined from the precision conversion option value of the opcode.

```
d set $lm0n0c0b0m0p0 4 l0000000000000001l0000000000000002l0000000000000003l0000000000000004
d set $lm8n0c0b0m0p0 4 l0000000000000005l0000000000000006l0000000000000007l0000000000000008

lpassa $lm0v $lt

d get $tn0c0b0m0p0 4

lpassa $lm8v $t

d get $ltn0c0b0m0p0 4
```

```
DEBUG-TREG(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x1}}, v:0x1) #d get $ltn0c0b0m0p0 4
DEBUG-TREG(n0c0b0m0p0,1):(f:0, i:{{0x0,0x0},{0x0,0x2}}, v:0x2) #d get $ltn0c0b0m0p0 4
DEBUG-TREG(n0c0b0m0p0,2):(f:0, i:{{0x0,0x0},{0x0,0x3}}, v:0x3) #d get $ltn0c0b0m0p0 4
DEBUG-TREG(n0c0b0m0p0,3):(f:0, i:{{0x0,0x0},{0x0,0x4}}, v:0x4) #d get $ltn0c0b0m0p0 4
DEBUG-TREG(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x5}}, v:0x5) #d get $ltn0c0b0m0p0 4
DEBUG-TREG(n0c0b0m0p0,1):(f:0, i:{{0x0,0x0},{0x0,0x6}}, v:0x6) #d get $ltn0c0b0m0p0 4
DEBUG-TREG(n0c0b0m0p0,2):(f:0, i:{{0x0,0x0},{0x0,0x7}}, v:0x7) #d get $ltn0c0b0m0p0 4
DEBUG-TREG(n0c0b0m0p0,3):(f:0, i:{{0x0,0x0},{0x0,0x8}}, v:0x8) #d get $ltn0c0b0m0p0 4
```

M (4bit): Represented as $omr in vsm. Holds information about whether to write (1) or not write (0) in each cycle.

```
d set $lm0n0c0b0m0p0 4 l0000000000000000l0000000000000001l0000000000000042l0000000000000000
d set $lm8n0c0b0m0p0 4 l3FF0000000000000l3FF0000000000000l3FF0000000000000l3FF0000000000000

lpassa $lm0v $omr1
maskn 1
lpassa $lm8v $ln0v
mask 0

d get $lm0n0c0b0m0p0 4
d get $omr1n0c0b0m0p0 1
d getd $ln0n0c0b0m0p0 4
```

```
DEBUG-LM0(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x0}}, v:0x0) #d get $lm0n0c0b0m0p0 4
DEBUG-LM0(n0c0b0m0p0,2):(f:0, i:{{0x0,0x0},{0x0,0x1}}, v:0x1) #d get $lm0n0c0b0m0p0 4
DEBUG-LM0(n0c0b0m0p0,4):(f:0, i:{{0x0,0x0},{0x0,0x42}}, v:0x42) #d get $lm0n0c0b0m0p0 4
DEBUG-LM0(n0c0b0m0p0,6):(f:0, i:{{0x0,0x0},{0x0,0x0}}, v:0x0) #d get $lm0n0c0b0m0p0 4
                             vv: Due to implementation reasons, 15(=0b1111) is displayed
DEBUG-OMR(n0c0b0m0p0,1):Mask{15} #d get $omr1n0c0b0m0p0 1  cycle 1: write (1)
DEBUG-OMR(n0c0b0m0p0,1):Mask{0} #d get $omr1n0c0b0m0p0 1   cycle 2: do not write (0)
DEBUG-OMR(n0c0b0m0p0,1):Mask{0} #d get $omr1n0c0b0m0p0 1   cycle 3: do not write (0)
DEBUG-OMR(n0c0b0m0p0,1):Mask{15} #d get $omr1n0c0b0m0p0 1  cycle 4: write (1)
DEBUG-LM1(n0c0b0m0p0,0):(1) (0x3ff0000000000000) #d getd $ln0n0c0b0m0p0 4
DEBUG-LM1(n0c0b0m0p0,2):(0) (0x0000000000000000) #d getd $ln0n0c0b0m0p0 4
DEBUG-LM1(n0c0b0m0p0,4):(0) (0x0000000000000000) #d getd $ln0n0c0b0m0p0 4
DEBUG-LM1(n0c0b0m0p0,6):(1) (0x3ff0000000000000) #d getd $ln0n0c0b0m0p0 4
```

MR (2 planes of 4x4(double),8x4(float),8x8(pseudo-float),16x16(half)): Corresponds to $x,$y. Details are in the MAB section.

forwarding-path: Represented as $mauf,$aluf,$lbf,$mreadf in vsm depending on the output source unit. The output of the immediately preceding arithmetic unit etc. can be read from here. With exceptions, it is updated every step, so the contents cannot be reused. If you want to reuse computation results, simply write them to memory, and even in that case, the forwarding-path contents are updated.

```
d set $lr0n0c0b0m0p0 1 0000000000000042

linc $lr0 $lm0
linc $aluf $ln0

d get $lm0n0c0b0m0p0 1
d get $ln0n0c0b0m0p0 1
```

```
DEBUG-LM0(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x43}}, v:0x43) #d get $lm0n0c0b0m0p0 1
DEBUG-LM1(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x44}}, v:0x44) #d get $ln0n0c0b0m0p0 1
```

Fixed value input: Represented as $l2bid,$l1bid,$mabid,$peid,$subpeid,$msb1 in vsm. Since all PEs on MN-Core2 receive the same assembly (SIMD), when you want to perform different processing for each PE, use these operands for mask processing, etc.

#### T register indirect reference

Like indexing on numpy.ndarray, there are cases where you want to perform indirect reference based on addresses determined at runtime. In such cases, use the T register indirect reference function.

```
d set $lm0n0c0b0m0p0 4 l0000000000000000l3FF0000000000000l4000000000000000l4008000000000000
# When you want to access 4 elements on LM0 in the order [2,1,3,0], write to the T register as follows.
#                                     v: 2*2           v: 2*1           v: 2*3           v: 2*0
d set $ltn0c0b0m0p0 4 l0000000000000004l0000000000000002l0000000000000006l0000000000000000

lpassa $lmt $ln0v

d getd $lm0n0c0b0m0p0 4
d getd $ln0n0c0b0m0p0 4
```

```
DEBUG-LM0(n0c0b0m0p0,0):(0) (0x0000000000000000) #d getd $lm0n0c0b0m0p0 4
DEBUG-LM0(n0c0b0m0p0,2):(1) (0x3ff0000000000000) #d getd $lm0n0c0b0m0p0 4
DEBUG-LM0(n0c0b0m0p0,4):(2) (0x4000000000000000) #d getd $lm0n0c0b0m0p0 4
DEBUG-LM0(n0c0b0m0p0,6):(3) (0x4008000000000000) #d getd $lm0n0c0b0m0p0 4
DEBUG-LM1(n0c0b0m0p0,0):(2) (0x4000000000000000) #d getd $ln0n0c0b0m0p0 4
DEBUG-LM1(n0c0b0m0p0,2):(1) (0x3ff0000000000000) #d getd $ln0n0c0b0m0p0 4
DEBUG-LM1(n0c0b0m0p0,4):(3) (0x4008000000000000) #d getd $ln0n0c0b0m0p0 4
DEBUG-LM1(n0c0b0m0p0,6):(0) (0x0000000000000000) #d getd $ln0n0c0b0m0p0 4
```

## 1MAB=4PE (4/4096 MN-Core2)

- Items added in this section
  - Matrix Arithmetic Unit (MAU)
  - Matrix register

### Vector operations

Despite the name Matrix Arithmetic Unit (MAU), vector operations on MN-Core2 are performed by the MAU. In most cases, calculations are closed per PE, but double-precision multiplication involves operations that are aware of the 4 PEs.

#### dv(mul|fma)

```
d set $lm0n0c0b0m0p0 1 3FF0000000000000
d set $lm0n0c0b0m0p1 1 4000000000000000
d set $lm0n0c0b0m0p2 1 4008000000000000
d set $lm0n0c0b0m0p3 1 4010000000000000
d set $ln0n0c0b0m0p0 1 3FF0000000000000
d set $ln0n0c0b0m0p1 1 4008000000000000
d set $ln0n0c0b0m0p2 1 4014000000000000
d set $ln0n0c0b0m0p3 1 401C000000000000

# p[0:3] can be omitted. In that case, it is automatically expanded.
d getd $lm0n0c0b0m0 1
d getd $ln0n0c0b0m0 1

dvmulu $lm0 $ln0 $nowrite
dvfmad $lm0 $ln0 $mauf $ls2

# Writing the above 2 lines verbosely looks like this. (optional: try running the following and looking at the dump results)

# dvmulu $lm0 $ln0 $lr0
# dvmuld $lm0 $ln0 $ls0
# d getd $lr0n0c0b0m0 1
# d getd $ls0n0c0b0m0 1
# nop/2
# dvadd $lr0 $ls0 $ls2

d getd $ls2n0c0b0m0 1
```

```
DEBUG-LM0(n0c0b0m0p0,0):(1) (0x3ff0000000000000) #d getd $lm0n0c0b0m0 1
DEBUG-LM0(n0c0b0m0p1,0):(2) (0x4000000000000000) #d getd $lm0n0c0b0m0 1
DEBUG-LM0(n0c0b0m0p2,0):(3) (0x4008000000000000) #d getd $lm0n0c0b0m0 1
DEBUG-LM0(n0c0b0m0p3,0):(4) (0x4010000000000000) #d getd $lm0n0c0b0m0 1
DEBUG-LM1(n0c0b0m0p0,0):(1) (0x3ff0000000000000) #d getd $ln0n0c0b0m0 1
DEBUG-LM1(n0c0b0m0p1,0):(3) (0x4008000000000000) #d getd $ln0n0c0b0m0 1
DEBUG-LM1(n0c0b0m0p2,0):(5) (0x4014000000000000) #d getd $ln0n0c0b0m0 1
DEBUG-LM1(n0c0b0m0p3,0):(7) (0x401c000000000000) #d getd $ln0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p0,2):(1) (0x3ff0000000000000) #d getd $ls2n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p1,2):(6) (0x4018000000000000) #d getd $ls2n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p2,2):(15) (0x402e000000000000) #d getd $ls2n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p3,2):(28) (0x403c000000000000) #d getd $ls2n0c0b0m0 1
```

#### fv(mul|fma)

```
d set $lm0n0c0b0m0p0 1 3F80000040A00000
d set $lm0n0c0b0m0p1 1 4000000040C00000
d set $lm0n0c0b0m0p2 1 4040000040E00000
d set $lm0n0c0b0m0p3 1 4080000041000000
d set $ln0n0c0b0m0p0 1 3F80000041100000
d set $ln0n0c0b0m0p1 1 4040000041300000
d set $ln0n0c0b0m0p2 1 40A0000041500000
d set $ln0n0c0b0m0p3 1 40E0000041700000
d set $lr0n0c0b0m0p0 1 3F8000003F800000
d set $lr0n0c0b0m0p1 1 3F8000003F800000
d set $lr0n0c0b0m0p2 1 3F8000003F800000
d set $lr0n0c0b0m0p3 1 3F8000003F800000

d getf $lm0n0c0b0m0 1
d getf $ln0n0c0b0m0 1

fvmul $lm0 $ln0 $ls0

d getf $ls0n0c0b0m0 1

fvfma $lm0 $ln0 $lr0 $ls0

d getf $ls0n0c0b0m0 1
```

```
DEBUG-LM0(n0c0b0m0p0,0):(1, 5) (0x3f800000, 0x40a00000) #d getf $lm0n0c0b0m0 1
DEBUG-LM0(n0c0b0m0p1,0):(2, 6) (0x40000000, 0x40c00000) #d getf $lm0n0c0b0m0 1
DEBUG-LM0(n0c0b0m0p2,0):(3, 7) (0x40400000, 0x40e00000) #d getf $lm0n0c0b0m0 1
DEBUG-LM0(n0c0b0m0p3,0):(4, 8) (0x40800000, 0x41000000) #d getf $lm0n0c0b0m0 1
DEBUG-LM1(n0c0b0m0p0,0):(1, 9) (0x3f800000, 0x41100000) #d getf $ln0n0c0b0m0 1
DEBUG-LM1(n0c0b0m0p1,0):(3, 11) (0x40400000, 0x41300000) #d getf $ln0n0c0b0m0 1
DEBUG-LM1(n0c0b0m0p2,0):(5, 13) (0x40a00000, 0x41500000) #d getf $ln0n0c0b0m0 1
DEBUG-LM1(n0c0b0m0p3,0):(7, 15) (0x40e00000, 0x41700000) #d getf $ln0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p0,0):(1, 45) (0x3f800000, 0x42340000) #d getf $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p1,0):(6, 66) (0x40c00000, 0x42840000) #d getf $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p2,0):(15, 91) (0x41700000, 0x42b60000) #d getf $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p3,0):(28, 120) (0x41e00000, 0x42f00000) #d getf $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p0,0):(2, 46) (0x40000000, 0x42380000) #d getf $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p1,0):(7, 67) (0x40e00000, 0x42860000) #d getf $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p2,0):(16, 92) (0x41800000, 0x42b80000) #d getf $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p3,0):(29, 121) (0x41e80000, 0x42f20000) #d getf $ls0n0c0b0m0 1
```

#### hv(mul|fma)

```
d set $lm0n0c0b0m0p0 1 3E00428044404540
d set $lm0n0c0b0m0p1 1 4000430044804580
d set $lm0n0c0b0m0p2 1 4100438044C045C0
d set $lm0n0c0b0m0p3 1 4200440045004600
d set $ln0n0c0b0m0p0 1 3E00444046204720
d set $ln0n0c0b0m0p1 1 410044C046604760
d set $ln0n0c0b0m0p2 1 4280454046A047A0
d set $ln0n0c0b0m0p3 1 438045C046E047E0
d set $lr0n0c0b0m0p0 1 3E003E003E003E00
d set $lr0n0c0b0m0p1 1 3E003E003E003E00
d set $lr0n0c0b0m0p2 1 3E003E003E003E00
d set $lr0n0c0b0m0p3 1 3E003E003E003E00

# (optional: try adding vsm here that matches the expected output using the above as reference)
```

Expected output:

```
DEBUG-GREG1(n0c0b0m0p0,0):(1, 45, 153, 325) (0x3e00, 0x48d0, 0x4c64, 0x4e8a) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p1,0):(6, 66, 190, 378) (0x4300, 0x4a10, 0x4cf8, 0x4ef4) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p2,0):(15, 91, 231, 435) (0x45c0, 0x4ad8, 0x4d9c, 0x4f66) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p3,0):(28, 120, 276, 496) (0x4780, 0x4bc0, 0x4e28, 0x4fe0) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p0,0):(2, 46, 154, 326) (0x4000, 0x48e0, 0x4c68, 0x4e8c) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p1,0):(7, 67, 191, 379) (0x4380, 0x4a18, 0x4cfc, 0x4ef6) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p2,0):(16, 92, 232, 436) (0x4600, 0x4ae0, 0x4da0, 0x4f68) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p3,0):(29, 121, 277, 497) (0x47a0, 0x4bc8, 0x4e2a, 0x4fe2) #d geth $ls0n0c0b0m0 1
```

Unexpected output (hint: half-precision addition is basically performed after precision extension):

```
DEBUG-GREG1(n0c0b0m0p0,0):(1.75, 0, 4.40625, 0) (0x3f80, 0x0000, 0x4234, 0x0000) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p1,0):(2.75, 0, 5.03125, 0) (0x40c0, 0x0000, 0x4284, 0x0000) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p2,0):(3.4375, 0, 5.42188, 0) (0x4170, 0x0000, 0x42b6, 0x0000) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p3,0):(3.875, 0, 5.875, 0) (0x41e0, 0x0000, 0x42f0, 0x0000) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p0,0):(1.78125, 6.98492e-09, 4.40625, -0) (0x3f90, 0x07c0, 0x4234, 0x803e) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p1,0):(2.76562, 0, 5.03125, 2.12109) (0x40c4, 0x01f0, 0x4284, 0x401f) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p2,0):(3.44531, 0, 5.42188, 2.12109) (0x4172, 0x00f8, 0x42b6, 0x401f) #d geth $ls0n0c0b0m0 1
DEBUG-GREG1(n0c0b0m0p3,0):(3.87891, 0, 5.875, 2.12109) (0x41e1, 0x007c, 0x42f0, 0x401f) #d geth $ls0n0c0b0m0 1
```

While (d|f|h)vpassa may seem to behave the same as IALU's passa, it involves normalization in the process, so you should not use vpassa on integers or special negative numbers:

```
d set $lm0n0c0b0m0p0 1 0000004280000000

fvpassa $lm0 $ln0

d getf $lm0n0c0b0m0p0 1
d get $lm0n0c0b0m0p0 1
d getf $ln0n0c0b0m0p0 1
d get $ln0n0c0b0m0p0 1
```

```
DEBUG-LM0(n0c0b0m0p0,0):(0, -0) (0x00000042, 0x80000000) #d getf $lm0n0c0b0m0p0 1
DEBUG-LM0(n0c0b0m0p0,0):(f:0, i:{{0x0,0x42},{0x8000,0x0}}, v:0x4280000000) #d get $lm0n0c0b0m0p0 1
# The output below shows that 42 and -0 have been corrupted
DEBUG-LM1(n0c0b0m0p0,0):(0, 0) (0x00000000, 0x00000000) #d getf $ln0n0c0b0m0p0 1
DEBUG-LM1(n0c0b0m0p0,0):(f:0, i:{{0x0,0x0},{0x0,0x0}}, v:0x0) #d get $ln0n0c0b0m0p0 1
```

A valid use of vpassa is precision conversion:

```
d set $lm0n0c0b0m0p0 4 l3FF0000000000000l4000000000000000l4008000000000000l4010000000000000

dvpassar $lm0v $n0v

d getd $lm0n0c0b0m0p0 4
d getf $ln0n0c0b0m0p0 2

d set $lm0n0c0b0m0p0 1 3E00428044404540

hvpassa $lm0 $lln0

d geth $lm0n0c0b0m0p0 1
d getf $ln0n0c0b0m0p0 2
```

```
DEBUG-LM0(n0c0b0m0p0,0):(1) (0x3ff0000000000000) #d getd $lm0n0c0b0m0p0 4
DEBUG-LM0(n0c0b0m0p0,2):(2) (0x4000000000000000) #d getd $lm0n0c0b0m0p0 4
DEBUG-LM0(n0c0b0m0p0,4):(3) (0x4008000000000000) #d getd $lm0n0c0b0m0p0 4
DEBUG-LM0(n0c0b0m0p0,6):(4) (0x4010000000000000) #d getd $lm0n0c0b0m0p0 4
DEBUG-LM1(n0c0b0m0p0,0):(1, 2) (0x3f800000, 0x40000000) #d getf $ln0n0c0b0m0p0 2
DEBUG-LM1(n0c0b0m0p0,2):(3, 4) (0x40400000, 0x40800000) #d getf $ln0n0c0b0m0p0 2
DEBUG-LM0(n0c0b0m0p0,0):(1, 5, 9, 13) (0x3e00, 0x4280, 0x4440, 0x4540) #d geth $lm0n0c0b0m0p0 1
DEBUG-LM1(n0c0b0m0p0,0):(1, 5) (0x3f800000, 0x40a00000) #d getf $ln0n0c0b0m0p0 2
DEBUG-LM1(n0c0b0m0p0,2):(9, 13) (0x41100000, 0x41500000) #d getf $ln0n0c0b0m0p0 2
```

### Matrix operations

Matrix operations roughly require the steps of block-floating each matrix and writing to the matrix register, and calling m(mul|fma) instructions. Here we explain matrix multiplication in double precision.

#### Consider 4x4 matmul in double (AxB)

```
# A
d set $lm0n0c0b0m0p0 4 80000000000000000000000000000000BFF0000000000000BFF0000000000000
d set $lm0n0c0b0m0p1 4 000000000000000040100000000000003FF0000000000000BFF0000000000000
d set $lm0n0c0b0m0p2 4 4000000000000000BFF0000000000000BFF00000000000003FF0000000000000
d set $lm0n0c0b0m0p3 4 BFF0000000000000C010000000000000BFF0000000000000BFF0000000000000
# B
d set $ln0n0c0b0m0p0 4 80000000000000003FF00000000000003FF0000000000000BFF0000000000000
d set $ln0n0c0b0m0p1 4 3FF0000000000000BFF00000000000003FF00000000000003FF0000000000000
d set $ln0n0c0b0m0p2 4 BFF000000000000040080000000000003FF00000000000003FF0000000000000
d set $ln0n0c0b0m0p3 4 C0100000000000003FF0000000000000C008000000000000BFF0000000000000

d getd $lm0n0c0b0m0 4
d getd $ln0n0c0b0m0 4

# Block-floating means aligning the exponent values for each of the two inputs to the inner product circuit. For double precision, this is done using the dbfn instruction.
dbfn $lm0v $lr0v # BF-ed A
dbfn $ln0v $ls0v # BF-ed B

# Next, write BF-ed A to the matrix register
dmwrite $lr0v $lx0

# Issue dmmulu+dmfmad
dmmulu $lx $ls0v $nowrite
dmfmad $lx $ls0v $mauf $lm8v

d getd $lm8n0c0b0m0 4
```

```
DEBUG-LM0(n0c0b0m0p0,0):(-0) (0x8000000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p0,2):(0) (0x0000000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p0,4):(-1) (0xbff0000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p0,6):(-1) (0xbff0000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p1,0):(0) (0x0000000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p1,2):(4) (0x4010000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p1,4):(1) (0x3ff0000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p1,6):(-1) (0xbff0000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p2,0):(2) (0x4000000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p2,2):(-1) (0xbff0000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p2,4):(-1) (0xbff0000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p2,6):(1) (0x3ff0000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p3,0):(-1) (0xbff0000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p3,2):(-4) (0xc010000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p3,4):(-1) (0xbff0000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p3,6):(-1) (0xbff0000000000000) #d getd $lm0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p0,0):(-0) (0x8000000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p0,2):(1) (0x3ff0000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p0,4):(1) (0x3ff0000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p0,6):(-1) (0xbff0000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p1,0):(1) (0x3ff0000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p1,2):(-1) (0xbff0000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p1,4):(1) (0x3ff0000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p1,6):(1) (0x3ff0000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p2,0):(-1) (0xbff0000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p2,2):(3) (0x4008000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p2,4):(1) (0x3ff0000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p2,6):(1) (0x3ff0000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p3,0):(-4) (0xc010000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p3,2):(1) (0x3ff0000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p3,4):(-3) (0xc008000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM1(n0c0b0m0p3,6):(-1) (0xbff0000000000000) #d getd $ln0n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p0,8):(2) (0x4000000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p0,10):(5) (0x4014000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p0,12):(5) (0x4014000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p0,14):(3) (0x4008000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p1,8):(21) (0x4035000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p1,10):(-11) (0xc026000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p1,12):(15) (0x402e000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p1,14):(7) (0x401c000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p2,8):(6) (0x4018000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p2,10):(-6) (0xc018000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p2,12):(2) (0x4000000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p2,14):(2) (0x4000000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p3,8):(2) (0x4000000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p3,10):(2) (0x4000000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p3,12):(2) (0x4000000000000000) #d getd $lm8n0c0b0m0 4
DEBUG-LM0(n0c0b0m0p3,14):(2) (0x4000000000000000) #d getd $lm8n0c0b0m0 4
```

The above dump results are simply illustrated as follows.

```
                   A (on $lx)                     B                                  Result
         PE0   PE1   PE2   PE3          PE0   PE1   PE2   PE3                PE0   PE1   PE2   PE3
       +-----+-----+-----+-----+      +-----+-----+-----+-----+            +-----+-----+-----+-----+
addr 0 | -0  |  0  |  2  | -1  | mmul | -0  |  1  | -1  | -4  |    addr  8 |  2  |  21 |  6  |  2  |
addr 2 |  0  |  4  | -1  | -4  |  +   |  1  | -1  |  3  |  1  | =  addr 10 |  5  | -11 | -6  |  2  |
addr 4 | -1  |  1  | -1  | -1  | mfma |  1  |  1  |  1  | -3  |    addr 12 |  5  |  15 |  2  |  2  |
addr 6 | -1  | -1  |  1  | -1  |      | -1  |  1  |  1  | -1  |    addr 14 |  3  |  7  |  2  |  2  |
       +-----+-----+-----+-----+      +-----+-----+-----+-----+            +-----+-----+-----+-----+
```

If you trace the calculations up to this point while checking the diagram, you should feel like you calculated something different from the usual matrix multiplication. The reason is that when viewing the address extension direction as rows and the PE direction as columns, what was calculated here is (A@B^T)^T. The [Software Developer Manual](https://projects.preferred.jp/mn-core/assets/mncore2_dev_manual_ja.pdf) has a more detailed explanation, but intuitively, it takes the product of a vector extracted row by row in the address direction and a matrix, and repeats that for each cycle. For example,

```
Result[addr8,PE0]=A[addr0,PE0]*B[addr0,PE0]+A[addr0,PE1]*B[addr0,PE1]+A[addr0,PE2]*B[addr0,PE2]+A[addr0,PE3]*B[addr0,PE3]
Result[addr8,PE1]=A[addr2,PE0]*B[addr0,PE0]+A[addr2,PE1]*B[addr0,PE1]+A[addr2,PE2]*B[addr0,PE2]+A[addr2,PE3]*B[addr0,PE3]
Result[addr8,PE2]=A[addr4,PE0]*B[addr0,PE0]+A[addr4,PE1]*B[addr0,PE1]+A[addr4,PE2]*B[addr0,PE2]+A[addr4,PE3]*B[addr0,PE3]
Result[addr8,PE3]=A[addr6,PE0]*B[addr0,PE0]+A[addr6,PE1]*B[addr0,PE1]+A[addr6,PE2]*B[addr0,PE2]+A[addr6,PE3]*B[addr0,PE3]
               ^        ^                         ^                         ^                         ^
...
```

The calculation proceeds in this manner. Recreating this calculation content using NumPy looks like this.

```py
import numpy as np
a = np.array(
    [[0, 0, 2, -1], [0, 4, -1, -4], [-1, 1, -1, -1], [-1, -1, 1, -1]], dtype=np.float32
)
b = np.array(
    [[0, 1, -1, -4], [1, -1, 3, 1], [1, 1, 1, -3], [-1, 1, 1, -1]], dtype=np.float32
)
print((a @ b.T).T)
```

```
[[  2.  21.   6.   2.]
 [  5. -11.  -6.   2.]
 [  5.  15.   2.   2.]
 [  3.   7.   2.   2.]]
```

When one side of the matrix product is transposed, you might think it's a completely different calculation. However, due to MN-Core's special memory structure, data on the host needs to be rearranged before being sent to the device, and by doing the transposition along with this, you can ultimately obtain what you want. Also, of course, you can perform matrix transposition on MN-Core.

challenge: Try writing a 16x16 matmul (A@B) in half. To realize an intuitive A@B, let's calculate B^TxA on MN-Core2. The explanation about matrix transposition is given around mread in the [Software Developer Manual](https://projects.preferred.jp/mn-core/assets/mncore2_dev_manual_ja.pdf).

```
# A
d set $lm0n0c0b0m0p0 16 8000BE00BE00BE003E003E003E003E003E003E00BE003E003E00BE003E00BE00BE003E00BE003E00BE00BE003E003E003E00BE003E003E003E00BE003E00BE003E00BE00BE003E003E00BE00BE00BE00BE003E00BE00BE00BE00BE00BE00BE00BE003E00BE003E003E00BE003E00BE00BE00BE003E003E003E00BE00BE003E00
d set $lm0n0c0b0m0p1 16 3E003E003E00BE003E00BE003E003E003E00BE00BE003E003E003E00BE0041003E003E003E003E003E00BE003E003E003E003E003E00BE003E004280BE003E003E00BE00BE00BE00BE003E00BE003E00BE00BE003E003E00BE00BE00BE003E00BE003E003E00BE00BE00BE00BE00BE003E00BE00C1003E00BE003E00BE00BE00
d set $lm0n0c0b0m0p2 16 BE00BE00BE003E003E003E003E00BE003E00BE003E003E00BE00BE00BE00BE003E003E00BE00BE00BE003E003E003E003E003E003E003E00BE003E003E003E003E00BE00BE003E00BE00BE004000BE003E003E003E003E00BE003E003E00BE00BE00BE00BE004000BE00BE003E003E00BE003E003E00BE003E00BE003E00BE00
d set $lm0n0c0b0m0p3 16 BE00BE00BE00BE00BE003E003E00BE00BE00BE003E00BE003E003E00BE00BE00BE003E003E003E003E00BE003E003E00BE003E003E00BE003E00BE00BE00BE00BE00BE00BE003E00BE00BE003E003E003E00BE00BE003E00BE00BE00BE00BE003E00BE00BE003E00BE00BE00BE003E003E003E003E003E00BE00BE00BE00BE00
# B
d set $ln0n0c0b0m0p0 16 80003E003E00BE003E003E003E003E00BE00BE003E003E00BE00BE003E003E004200BE003E003E00BE00BE00BE003E00BE003E00BE003E003E003E003E003E003E003E00BE003E003E003E003E00BE0040003E00BE003E00BE003E003E00BE003E003E00BE00BE00BE00BE00BE003E003E003E00BE00BE003E00BE003E00BE00
d set $ln0n0c0b0m0p1 16 3E00BE003E003E00BE00BE00BE003E0042003E003E003E003E00BE00BE00BE00BE00BE003E00BE00BE003E003E00BE00BE003E003E00BE003E00BE00BE00BE003E003E003E00BE00BE003E003E003E00BE003E00BE003E00BE00BE00BE003E003E003E003E003E00BE00BE003E003E00BE003E003E003E00BE003E003E003E00
d set $ln0n0c0b0m0p2 16 3E003E003E00BE00BE003E00BE00BE003E003E003E00BE00BE003E003E003E003E003E00BE00BE003E003E00BE00BE003E003E00BE003E003E003E00BE003E00BE003E004100BE00BE003E00BE003E003E003E003E00BE003E00BE00BE003E00BE003E003E00BE003E00BE003E003E003E003E003E00BE003E004200BE00BE00
d set $ln0n0c0b0m0p3 16 BE00BE003E00BE003E00BE00BE003E00BE003E00BE00BE003E00BE00BE00BE00BE003E00BE00BE003E00BE00BE003E003E003E00BE00BE00BE00BE003E00BE003E003E00BE00BE003E00BE003E003E003E00BE00BE000000BE00BE003E00BE003E00BE00BE003E00BE00BE00BE003E003E00BE00BE003E003E003E003E00BE00

# (write vsm here!)
d geth $ln32n0c0b0m0 16
```

Expected output:

```
DEBUG-LM1(n0c0b0m0p0,32):(-5, -3, -1, -1) (0xc280, 0xc100, 0xbe00, 0xbe00) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,34):(7, 4, 2, 8) (0x4380, 0x4200, 0x4000, 0x4400) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,36):(9, 6, 4, 2) (0x4440, 0x4300, 0x4200, 0x4000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,38):(1, -4, 2, 4) (0x3e00, 0xc200, 0x4000, 0x4200) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,40):(5, -2, 0, 6) (0x4280, 0xc000, 0x0000, 0x4300) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,42):(7, 2, 4, -2) (0x4380, 0x4000, 0x4200, 0xc000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,44):(-1, 0, -2, 4) (0xbe00, 0x0000, 0xc000, 0x4200) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,46):(1, -2, 0, 2) (0x3e00, 0xc000, 0x0000, 0x4000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,48):(1, -4, 6, -4) (0x3e00, 0xc200, 0x4300, 0xc200) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,50):(3, 1, -3, -3) (0x4100, 0x3e00, 0xc100, 0xc100) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,52):(5, 10, 0, -2) (0x4280, 0x4480, 0x0000, 0xc000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,54):(1, 2, 0, -2) (0x3e00, 0x4000, 0x0000, 0xc000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,56):(-10, -1, 1, -3) (0xc480, 0xbe00, 0x3e00, 0xc100) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,58):(-5, -2, 4, -6) (0xc280, 0xc000, 0x4200, 0xc300) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,60):(11, -6, 4, -2) (0x44c0, 0xc300, 0x4200, 0xc000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p0,62):(-5, -2, -4, 2) (0xc280, 0xc000, 0xc200, 0x4000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,32):(-6, -3, -1, -7) (0xc300, 0xc100, 0xbe00, 0xc380) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,34):(3, -2, 2, 0) (0x4100, 0xc000, 0x4000, 0x0000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,36):(-1, -8, -8, -2) (0xbe00, 0xc400, 0xc400, 0xc000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,38):(11, -6, 2, -4) (0x44c0, 0xc300, 0x4000, 0xc200) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,40):(-9, 0, 4, -6) (0xc440, 0x0000, 0x4200, 0xc300) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,42):(1, 4, 0, 2) (0x3e00, 0x4200, 0x0000, 0x4000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,44):(-1, 2, 6, 0) (0xbe00, 0x4000, 0x4300, 0x0000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,46):(1, 4, 4, -2) (0x3e00, 0x4200, 0x4200, 0xc000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,48):(1, -6, -2, -4) (0x3e00, 0xc300, 0xc000, 0xc200) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,50):(-4, 3, -3, 1) (0xc200, 0x4100, 0xc100, 0x3e00) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,52):(-5, 4, -4, 2) (0xc280, 0x4200, 0xc200, 0x4000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,54):(-1, 0, -8, -2) (0xbe00, 0x0000, 0xc400, 0xc000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,56):(-6, -1, -5, -1) (0xc300, 0xbe00, 0xc280, 0xbe00) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,58):(5, 0, -4, 6) (0x4280, 0x0000, 0xc200, 0x4300) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,60):(5, 0, 0, 6) (0x4280, 0x0000, 0x0000, 0x4300) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p1,62):(3, 0, -4, -6) (0x4100, 0x0000, 0xc200, 0xc300) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,32):(3, -10, -9, 3) (0x4100, 0xc480, 0xc440, 0x4100) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,34):(2, 5, 6, 0) (0x4000, 0x4280, 0x4300, 0x0000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,36):(0, -1, 4, -2) (0x0000, 0xbe00, 0x4200, 0xc000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,38):(6, -3, -2, 0) (0x4300, 0xc100, 0xc000, 0x0000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,40):(0, 9, -4, 2) (0x0000, 0x4440, 0xc200, 0x4000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,42):(4, 9, -4, 2) (0x4200, 0x4440, 0xc200, 0x4000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,44):(6, 1, 6, 0) (0x4300, 0x3e00, 0x4300, 0x0000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,46):(8, 3, -8, -6) (0x4400, 0x4100, 0xc400, 0xc300) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,48):(-2, -1, 2, 0) (0xc000, 0xbe00, 0x4000, 0x0000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,50):(7, 4, -1, -5) (0x4380, 0x4200, 0xbe00, 0xc280) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,52):(-4, 5, -4, 2) (0xc200, 0x4280, 0xc200, 0x4000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,54):(-4, -9, -4, 6) (0xc200, 0xc440, 0xc200, 0x4300) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,56):(-3, 0, -9, 3) (0xc100, 0x0000, 0xc440, 0x4100) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,58):(4, -3, 0, -2) (0x4200, 0xc100, 0x0000, 0xc000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,60):(0, 5, 4, -2) (0x0000, 0x4280, 0x4200, 0xc000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p2,62):(-4, -5, 8, -2) (0xc200, 0xc280, 0x4400, 0xc000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,32):(-5, 5, 3, -2) (0xc280, 0x4280, 0x4100, 0xc000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,34):(0, -2, -6, -3) (0x0000, 0xc000, 0xc300, 0xc100) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,36):(-2, -4, 0, -5) (0xc000, 0xc200, 0x0000, 0xc280) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,38):(-12, -2, 2, -1) (0xc500, 0xc000, 0x4000, 0xbe00) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,40):(6, 0, -4, 1) (0x4300, 0x0000, 0xc200, 0x3e00) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,42):(2, 0, 0, -5) (0x4000, 0x0000, 0x0000, 0xc280) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,44):(0, -2, -6, -3) (0x0000, 0xc000, 0xc300, 0xc100) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,46):(-2, -8, 0, 3) (0xc000, 0xc400, 0x0000, 0x4100) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,48):(-4, 6, 6, -9) (0xc200, 0x4300, 0x4300, 0xc440) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,50):(1, -3, 5, 1) (0x3e00, 0xc100, 0x4280, 0x3e00) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,52):(6, 0, 4, -1) (0x4300, 0x0000, 0x4200, 0xbe00) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,54):(-2, 0, 8, 3) (0xc000, 0x0000, 0x4400, 0x4100) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,56):(5, -1, 1, 0) (0x4280, 0xbe00, 0x3e00, 0x0000) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,58):(-6, 4, 8, -5) (0xc300, 0x4200, 0x4400, 0xc280) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,60):(-2, -4, 0, 3) (0xc000, 0xc200, 0x0000, 0x4100) #d geth $ln32n0c0b0m0 16
DEBUG-LM1(n0c0b0m0p3,62):(2, 0, 0, -1) (0x4000, 0x0000, 0x0000, 0xbe00) #d geth $ln32n0c0b0m0 16
```

## 1L1B=16MAB (64/4096 MN-Core2)

- Items added in this section
  - Distribute/Gather + Broadcast/Reduce between L1B and 16MABs (result reduce network, RRN)
  - L1BM (8192 long words): Represented as $lb in vsm.

### Implementation of Sum using RRN

When reducing data spanning multiple MABs, use the RRN. The implementation of Sum (torch.sum) uses double-precision addition to reduce, and then sends the calculation results back to each MAB via broadcast using the same communication path.

```
d set $lm0n0c0b0m0p0 1 8000000000000000
d set $lm0n0c0b0m0p1 1 C02F000000000000
d set $lm0n0c0b0m0p2 1 C02E000000000000
d set $lm0n0c0b0m0p3 1 C02D000000000000
d set $lm0n0c0b0m1p0 1 C02C000000000000
d set $lm0n0c0b0m1p1 1 C02B000000000000
d set $lm0n0c0b0m1p2 1 C02A000000000000
d set $lm0n0c0b0m1p3 1 C029000000000000
d set $lm0n0c0b0m2p0 1 C028000000000000
d set $lm0n0c0b0m2p1 1 C027000000000000
d set $lm0n0c0b0m2p2 1 C026000000000000
d set $lm0n0c0b0m2p3 1 C025000000000000
d set $lm0n0c0b0m3p0 1 C024000000000000
d set $lm0n0c0b0m3p1 1 C023000000000000
d set $lm0n0c0b0m3p2 1 C022000000000000
d set $lm0n0c0b0m3p3 1 C021000000000000
d set $lm0n0c0b0m4p0 1 C020000000000000
d set $lm0n0c0b0m4p1 1 C01E000000000000
d set $lm0n0c0b0m4p2 1 C01C000000000000
d set $lm0n0c0b0m4p3 1 C01A000000000000
d set $lm0n0c0b0m5p0 1 C018000000000000
d set $lm0n0c0b0m5p1 1 C016000000000000
d set $lm0n0c0b0m5p2 1 C014000000000000
d set $lm0n0c0b0m5p3 1 C012000000000000
d set $lm0n0c0b0m6p0 1 C010000000000000
d set $lm0n0c0b0m6p1 1 C00C000000000000
d set $lm0n0c0b0m6p2 1 C008000000000000
d set $lm0n0c0b0m6p3 1 C004000000000000
d set $lm0n0c0b0m7p0 1 C000000000000000
d set $lm0n0c0b0m7p1 1 BFF8000000000000
d set $lm0n0c0b0m7p2 1 BFF0000000000000
d set $lm0n0c0b0m7p3 1 BFE0000000000000
d set $lm0n0c0b0m8p0 1 0000000000000000
d set $lm0n0c0b0m8p1 1 3FE0000000000000
d set $lm0n0c0b0m8p2 1 3FF0000000000000
d set $lm0n0c0b0m8p3 1 3FF8000000000000
d set $lm0n0c0b0m9p0 1 4000000000000000
d set $lm0n0c0b0m9p1 1 4004000000000000
d set $lm0n0c0b0m9p2 1 4008000000000000
d set $lm0n0c0b0m9p3 1 400C000000000000
d set $lm0n0c0b0m10p0 1 4010000000000000
d set $lm0n0c0b0m10p1 1 4012000000000000
d set $lm0n0c0b0m10p2 1 4014000000000000
d set $lm0n0c0b0m10p3 1 4016000000000000
d set $lm0n0c0b0m11p0 1 4018000000000000
d set $lm0n0c0b0m11p1 1 401A000000000000
d set $lm0n0c0b0m11p2 1 401C000000000000
d set $lm0n0c0b0m11p3 1 401E000000000000
d set $lm0n0c0b0m12p0 1 4020000000000000
d set $lm0n0c0b0m12p1 1 4021000000000000
d set $lm0n0c0b0m12p2 1 4022000000000000
d set $lm0n0c0b0m12p3 1 4023000000000000
d set $lm0n0c0b0m13p0 1 4024000000000000
d set $lm0n0c0b0m13p1 1 4025000000000000
d set $lm0n0c0b0m13p2 1 4026000000000000
d set $lm0n0c0b0m13p3 1 4027000000000000
d set $lm0n0c0b0m14p0 1 4028000000000000
d set $lm0n0c0b0m14p1 1 4029000000000000
d set $lm0n0c0b0m14p2 1 402A000000000000
d set $lm0n0c0b0m14p3 1 402B000000000000
d set $lm0n0c0b0m15p0 1 402C000000000000
d set $lm0n0c0b0m15p1 1 402D000000000000
d set $lm0n0c0b0m15p2 1 402E000000000000
d set $lm0n0c0b0m15p3 1 402F000000000000

d getd $lm0n0c0b0 1

l1bmrdfadd $lm0 $lb0
nop
nop
l1bmm $lb0 $lm4v/1000

d getd $lm4n0c0b0m0p0 1
d getd $lm4n0c0b0m0p1 1
d getd $lm4n0c0b0m0p2 1
d getd $lm4n0c0b0m0p3 1
```

```
DEBUG-LM0(n0c0b0m0p0,0):(-0) (0x8000000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m0p1,0):(-15.5) (0xc02f000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m0p2,0):(-15) (0xc02e000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m0p3,0):(-14.5) (0xc02d000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m1p0,0):(-14) (0xc02c000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m1p1,0):(-13.5) (0xc02b000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m1p2,0):(-13) (0xc02a000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m1p3,0):(-12.5) (0xc029000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m2p0,0):(-12) (0xc028000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m2p1,0):(-11.5) (0xc027000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m2p2,0):(-11) (0xc026000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m2p3,0):(-10.5) (0xc025000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m3p0,0):(-10) (0xc024000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m3p1,0):(-9.5) (0xc023000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m3p2,0):(-9) (0xc022000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m3p3,0):(-8.5) (0xc021000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m4p0,0):(-8) (0xc020000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m4p1,0):(-7.5) (0xc01e000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m4p2,0):(-7) (0xc01c000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m4p3,0):(-6.5) (0xc01a000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m5p0,0):(-6) (0xc018000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m5p1,0):(-5.5) (0xc016000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m5p2,0):(-5) (0xc014000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m5p3,0):(-4.5) (0xc012000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m6p0,0):(-4) (0xc010000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m6p1,0):(-3.5) (0xc00c000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m6p2,0):(-3) (0xc008000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m6p3,0):(-2.5) (0xc004000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m7p0,0):(-2) (0xc000000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m7p1,0):(-1.5) (0xbff8000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m7p2,0):(-1) (0xbff0000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m7p3,0):(-0.5) (0xbfe0000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m8p0,0):(0) (0x0000000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m8p1,0):(0.5) (0x3fe0000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m8p2,0):(1) (0x3ff0000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m8p3,0):(1.5) (0x3ff8000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m9p0,0):(2) (0x4000000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m9p1,0):(2.5) (0x4004000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m9p2,0):(3) (0x4008000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m9p3,0):(3.5) (0x400c000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m10p0,0):(4) (0x4010000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m10p1,0):(4.5) (0x4012000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m10p2,0):(5) (0x4014000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m10p3,0):(5.5) (0x4016000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m11p0,0):(6) (0x4018000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m11p1,0):(6.5) (0x401a000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m11p2,0):(7) (0x401c000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m11p3,0):(7.5) (0x401e000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m12p0,0):(8) (0x4020000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m12p1,0):(8.5) (0x4021000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m12p2,0):(9) (0x4022000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m12p3,0):(9.5) (0x4023000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m13p0,0):(10) (0x4024000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m13p1,0):(10.5) (0x4025000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m13p2,0):(11) (0x4026000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m13p3,0):(11.5) (0x4027000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m14p0,0):(12) (0x4028000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m14p1,0):(12.5) (0x4029000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m14p2,0):(13) (0x402a000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m14p3,0):(13.5) (0x402b000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m15p0,0):(14) (0x402c000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m15p1,0):(14.5) (0x402d000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m15p2,0):(15) (0x402e000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m15p3,0):(15.5) (0x402f000000000000) #d getd $lm0n0c0b0 1
DEBUG-LM0(n0c0b0m0p0,4):(0) (0x0000000000000000) #d getd $lm4n0c0b0m0p0 1
DEBUG-LM0(n0c0b0m0p1,4):(-8) (0xc020000000000000) #d getd $lm4n0c0b0m0p1 1
DEBUG-LM0(n0c0b0m0p2,4):(0) (0x0000000000000000) #d getd $lm4n0c0b0m0p2 1
DEBUG-LM0(n0c0b0m0p3,4):(8) (0x4020000000000000) #d getd $lm4n0c0b0m0p3 1
```
