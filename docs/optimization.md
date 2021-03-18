# Optimization Hints

Here is some topics about perforamnce optimization.

## `nc` variants of memory access intrinsics

Memor access intrinsics such as vector load, store, gather and scatter, have
variants that has `nc` in mnemonic such as `_vel_vldnc_vssl`. It can be used to
minimize polution of LLC, Last Level Cache. See [ISA
Guide](https://www.hpc.nec/documents/guide/pdfs/Aurora_ISA_guide.pdf#page=90)
for details.

other topics...

- scheduling: load/store and arithmetic
- svob, ot
- unrolling
- approximate
- low throghput, cross-pipe instructions
