# Introduction: Implementing VectorAdd

Let's start with a simple example. This is C/C++ implementation of addition of
two vectors.

```C++
void vadd(double* a, double const* b, double const* c, int n) {
    for (int i = 0; i < n; ++i)
        a[i] = b[i] + c[i];    
}
```

Vector instructions can operate 256 elements in a single instruction.
Strip-mining to make an innermost loop with 256 iterations is first step to use
intrinsics.

```c++
#define VL 256
void vadd(double* a, double const* b, double const* c, int n) {
    for (int i = 0; i < n; i += VL) {
         int vl = min(VL, n - i);
         for (int j = 0; j < vl; ++j)
             a[j] = b[j] + c[j];
         a += vl;
         b += vl;
         c += vl;
    }
}
```

Then replace the innermost loop with intrinsic functions and `__vr` (vector register) type
variables after including `velintrin.h`.


```c++
#include <velintrin.h>
#define VL 256
void vadd(double* a, double const* b, double const* c, int n) {
    for (int i = 0; i < n; i += VL) {
        int vl = min(VL, n - i);
        __vr vb = _vel_vld_vssl(8, b, vl);      // load b to vb
        __vr vc = _vel_vld_vssl(8, c, vl);      // load c to vc
        __vr va = _vel_vfaddd_vvvl(vb, bc, vl); // va = vb + vc
        _vel_vst_vssl(va, 8, a, vl);            // store va to a
        a += vl;
        b += vl;
        c += vl;
    }
}
```

A vector register can hold 256 64bit elements and an intrinsic function
operates `vl` elements in it. `vl` is passed as a last argument of an intrinsic
function.

The four intrinsics used in this example are compiled to four vector
instructions `vld, vld, vfadd.d, vst`.  This is an important benefit of
intrinsics. You can use the instructions you want to use.

Other examples can be found:

- <https://github.com/sx-aurora-dev/vetfkernel/tree/master/vml/src/intrinsic>
- <https://github.com/sx-aurora-dev/vednn>

