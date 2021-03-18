# Basics of VE Intrinsics

This section describes some basic consepts of VE Intrinsics.

## Design of VE Intrinsics

VE intrinsics are intended to be as consistent as possible with the VE assembly
described in [the assembly
manual](https://www.hpc.nec/documents/sdk/pdfs/VectorEngine-as-manual-v1.3.pdf).
Conventions such as function name, order of arguments are borrowed from the
assembly. This is because goal of VE intrinsics is providing an easy way to
write assembly code  with help of the compiler techniques such as register
allocation, transformations like unrolling and C++ template. 

If you are familiar with VE ISA, you can start to write intrinsics without
reading this document, [function
list](https://sx-aurora-dev.github.io/velintrin.html) is all you need. But this
section helps to understand VE intrinsics quickly. Knowledge on other vector
ISA or SIMD ISA could help to understand VE intrinsics.

Unfortunately the assembly manual does not describe functionality of
instructions. You may need to refer the [ISA
guide](https://www.hpc.nec/documents/guide/pdfs/Aurora_ISA_guide.pdf) to know
what intrinsics do. [Our intrinsic
list](https://sx-aurora-dev.github.io/velintrin.html) includes links to both
manuals.

## Vector Type and Functions

VE Intrinsics introduce **vector type** `__vr` and **functions** that operate
on `__vr` type variables. Typical code with intrinsics starts with loading data
into `__vr` type variables, then works with them, and finally stores to memory.

### Vector Type

 `__vr` is defined as below. It can hold 256 x 64bit values (2048KB in total).

```c++
typedef double __vr __attribute__((__vector_size__(2048)));
```

Variables of `__vr` are assigned to vector registers. As a vector register can
hold any types of value including fp32, fp64, int32, int64, etc, you can place
any types of value into `__vr` variable.

It is important to know that VE has 64 vector registers. Using too many `__vr`
variables results in vector register spills you all hate.

### Intrinsic Functions

Function format is  `_vel_<asm>_<suffix>`.

`<asm>` is an instruction mnemonic in [the assembly
manual](https://www.hpc.nec/documents/sdk/pdfs/VectorEngine-as-manual-v1.3.pd).
Some assembly instructions have mnemonic suffix to give additional means to an
instruction. For example, `vfadd.d` instruction has `.d` suffix to show that
operands are handled as fp64. See the assembly manual for the complete list of
mnemonic suffix. In intrinsics, dot(`.`) in mnemonic is simply removed. For
example, `vfadd.d` becomes `vfaddd`.

`<suffix>` is a list of types of return value and arguments in a single
character.

- `v`: vector
- `s`: scalar
- `m` and `M`: vector mask for 256 elements and 512 elements
- `l`: vector length

For example, suffix `vssl` in `_vel_vld_vssl` means vector(return value),
scalar(1st argument), scalar(2nd) and vector length(3rd). Some functions has
variants with different suffix to accept different type of arguments. For
example, `_vel_vfaddd_vvvl` is addition of two vectors and `_vel_vfaddd_vsvl`
is addition among a vector and a scalar where a
scalar is added to all elements in a vector.

### Vector Length and Pass Through Argument

Vector length argument shows number of elements updated by the function. For
example,

`va = _vel_vfaddd_vvvl(vb, vc, 100)`

updates first 100 elements of `va` with the result of `vb + vc`.  Elements
after vector length, i.e. last 156 elements in this case, becomes
**undefined**. Note that this is different from hardware instructions where
such elements are not changed.

You can use another variant of a function to give a  *pass through argument*
that is filled into elements after vector length. For example,
`_vel_vfaddd_vvvvl`(four `v`s in a suffix) is the variant of
`_vel_vfaddd_vvvl`(three `v`s) and its third argument is a pass through
argument (In [the intrinsics
list](https://sx-aurora-dev.github.io/velintrin.html), a pass through argument
is shown as `pt`). In the case of

`va = _vel_vfaddd_vvvvl(vb, vc, vd, 100)`,


last 156 elements of `va` are updated with last 156 elements of `vd`.
Typically, a pass through argument is same as an output variable like 

`va = _vel_vfadd_vvvvl(vb, vc, va, 100)`

to keep elements after vector length unchanged.

Note: Hardware vector instructions do not actually have a vector length
operand.  Instead vector length is given by the vector length register, and its
value is loaded by `LVL` (Load Vector Length) instruction. VE intrinsics does
not provide a function for LVL instruction, but it is automatically generated
by the compiler from a vector length argument in intrinsics.

## Vector Mask

VE has eight 256 bit vector mask registers. A vector mask register is given to
a vector instruction to *mask* elements of its output vector. When an element
is masked, i.e. a bit is zero, such element is not updated by the instruction.

VE intrinsics introduce `__vm256` type to hold a vector mask. Intrinsic
functions has variants that accept `__vm256` variable as an argument.  Such
variants also accept a pass through argument to fill masked elements.  For
example, the variant of `_vel_vfaddd_vvvl` is `_vel_vfaddd_vvvmvl` where
`m` in suffix shows a vector mask and `v` after `m` means a pass through
argument. `va = _vel_vfaddd_vvvmvl(vb, vc, vm, vd, vl)` works as

```c++
for (i = 0; i < vl; ++i)
  va[i] = vm[i] ? vb[i] + vc[i] : vd[i]
```

Typically, a pass through argument is same as an output variable like 

`va = _vel_vfadd_vvvmvl(vb, vc, vm, va, vl)`

to keep masked elements unchanged.

A vector mask is used for a loop with a conditional branch.

```c++
void vaddif(double* a, double const* b, double const* c, double const* cond, int n) {
    for (int i = 0; i < n; ++i) {
        if (cond[i] > 0.0)
            a[i] = b[i] + c[i];
    }
}
```

An example written in intrinsics is shown below. 

```c++
#define VL 256
for (int i = 0; i < n; i += VL) {
    int vl = std::min(VL, n - i);
    __vr va = _vel_vld_vssl(8, a, vl);
    __vr vb = _vel_vld_vssl(8, b, vl);
    __vr vc = _vel_vld_vssl(8, c, vl);
    __vr vcond = _vel_vld_vssl(8, cond, vl);
    __vm256 mask = _vel_vfmkdgt_mvl(vcond, vl); // mask = vcond > 0
    va = _vel_vfaddd_vvvmvl(vb, vc, mask, va, vl); // va = mask ? vb + vc : va
    _vel_vst_vssl(va, 8, a, vl);

    a += vl;
    b += vl;
    c += vl;
    cond += vl;
}
```


