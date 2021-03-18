# Getting started: Setup LLVM-VE

LLVM-VE is a community supported open source compiler and not included in
the official SDK for SX-Aurora TSUBASA. First of all you have to install
LLVM-VE. The LLVM-VE rpm packages are released at our
[branch](https://github.com/sx-aurora-dev/llvm-project/releases) of llvm. Note
that the official [llvm repository](https://github.com/llvm/llvm-project) also
has VE backend, but it does not yet support VE intrinsics at the time of
writing.  If you prefer a yum repository, see
<https://www.sx-aurora.com/repos/llvm/x86_64/>.

Install LLVM-VE.

```bash
% yum install llmv-ve-<version>.rpm llvm-ve-link-<version>.rpm
```

LLVM-VE is installed into `/opt/nec/nosupport/llvm-ve-<version>` and latest
version is linked from `/opt/nec/nosupport/llvm-ve`. The compiler can be
invoked as below.

```shell
% /opt/nec/nosupport/llvm-ve/clang -target ve-linux -O3 -c func.c
```

Because current LLVM-VE does not support automatic vectorization, we recommend
to use llvm-ve with ncc/nc++/nfort, default compilers for VE. You typically
create the dedicated source files for llvm-ve that includes minimum code with
intrinsics. Other source files should be compiled by ncc, then linked by ncc.
For example,

```bash
% /opt/nec/nosupport/llvm-ve/clang -target ve-linux -O3 -c intrinsics.c
% /opt/nec/ve/bin/ncc -c main.c
% /opt/nec/ve/bin/ncc -o a.out intrinsics.o main.o
```

## OpenMP

LLVE-VE supports OpenMP but its runtime is different from ncc's runtime. We
strongly recommend to use ncc's OpenMP. In other word, source code with
intrinsics should not include OpenMP.
