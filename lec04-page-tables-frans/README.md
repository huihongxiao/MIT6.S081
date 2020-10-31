# Lec04 Page tables \(Frans\)

准备工作，阅读【1】中第3章；阅读memlayout.h【2】；阅读vm.c【3】；阅读kalloc【4】；阅读riscv.h【5】；阅读exec.c【6】



【1】[https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf](https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf)

【2】[https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/memlayout.h](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/memlayout.h)

【3】[https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/vm.c](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/vm.c)

【4】[https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/kalloc.c](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/kalloc.c)

【5】[https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/riscv.h](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/riscv.h)

【6】[https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/exec.c](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/exec.c)

