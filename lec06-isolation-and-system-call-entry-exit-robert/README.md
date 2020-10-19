# Lec06 Isolation & system call entry/exit \(Robert\)

准备工作：阅读【1】中第4章，除了4.6；阅读RISCV.h【2】；阅读trampoline.S【3】；阅读trap.c【4】



【1】[https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf](https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf)

【2】[https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/riscv.h](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/riscv.h)

【3】[https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/trampoline.S](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/trampoline.S)

【4】[https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/trap.c](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/trap.c)

