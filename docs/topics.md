# Other Topics

In this section, more topics will be described.

## Vector Load and Store

The instrinsics you have to know first should be vector load and store to
load/store regular data such as consecutive or constant stride data from/to
memory. The intrinsic you need varies depending on your data type. To load
64bit data such as fp64(double) and int64, `_vel_vld_vssl` intrinsic can be
used. To load 32bit data, there are two intrinsics because we have conventions
to load fp32(float) into upper half of 64b, and to load int32 into lower half.
To this end, load upper`_vel_vldu_vssl` and load lower `_vel_vldl_vssl` can be
used. 

```c++
__vr v = _vel_vld_vssl(8, a, 256); // double* a or int64_t* a
__vr v = _vel_vldu_vssl(4, a, 256); // float* a;
__vr v = _vel_vldl_vssl(4, a, 256); // int32_t* a;
```

Vector load and vector store have two scalar arguments. First one is a stride
in byte and second one is a base address. When you load consecutive elements of
fp64, a stride should be 8B. When fp32, it is 4B. In above example, all loads
are consecutive loads. You can use longer stride to load non consecutive
elements, such as loading a column in 2d matrix.

```c++
double m[256*256]; // 256x256 row-major matrix
__vr v0 = _vel_vld_vssl(8, m, 256); // load row: m[i] (i=0-255)
__vr v1 = _vel_vld_vssl(8*256, m, 256); // load column: m[i*256] (i=0-255)
__vr v2 = _vel_vsl_vssl(8*2, m+1, 256); // load odd elements: m[2*i+1] (i=0-255)
```

## Packed Instructions

VE has packed instructions that operate 512 elements of fp32 or int32 in a
vector register. In intrinsics, packed instruction has prefix `p` before
instruction name. For example, `_vel_pvfadd_vl` is packed version of
`_vel_vfadds_vvvl` (`s` in `vfadds` that means fp32 is removed). 

You can load 512 32-bit elemens into a vector register by using a single VLD
instruction (as 256 32-bit elements), but data should be 8e aligned.

When a packed instruction takes a scalar value as an input operand, it also has
to be packed, i.e. a 64-bit scalar consists of 32b upper and 32b lower part.
VE intrinsics includes [the
functions](https://sx-aurora-dev.github.io/velintrin.html#sec12) to build a
packed scalar.

Hardware packed instructions can take two 256-bit vector mask register.  VE
intrinsics provide `__vm512` type that is mapped to two hardware vector mask
registers, and intrinsics for packed instructions accpet it (`M` in a suffix).
VE intrinsics also provide pseudo instructions for `__vm512` variables such as
`__vm512 _vel_andm_MMM(__vm512 vmy, __vm512 vmz)`. These
pseudo instructions are compiled to two hardware instructions.

You can extract `__vm256` from `__vm512` by using `_vel_extract_vm512u` and
`_vel_extract_vm512l`. Also functions for inserting are provided.
