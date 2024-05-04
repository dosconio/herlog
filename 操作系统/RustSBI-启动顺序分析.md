

### 20240504

commit: `20240503 RustSBI Bootstrap Used` 

```asm
	.section .text.entry
	.globl _entry
_entry:
	la sp, boot_stack_top
	call main

	.section .bss.stack
	.globl boot_stack
boot_stack:
	.space 4096 * 16
	.globl boot_stack_top
boot_stack_top:
```

```C
void main()
{
	while (1);
}
```

```bash
ayano@u64mac:~/mecocoa$ make dbg-r
make -f mecocoa/makefil/riscv64.make debug
make[1]: Entering directory '/home/ayano/mecocoa'
Mecocoa Risc-V64
riscv64-unknown-elf-gcc -Wall -Werror -O -fno-omit-frame-pointer -ggdb -MD -mcmodel=medany -ffreestanding -fno-common -nostdlib -mno-relax -Iriscv64 -fno-stack-protector -D _LOG_LEVEL_ERROR -fno-pie -no-pie -c riscv64/kernel-riscv64.c -o /home/ayano/_obj/mcca/riscv64/kernel-riscv64.o
riscv64-unknown-elf-ld -z max-page-size=4096 -T riscv64/kernel.ld -o /mnt/hgfs/_bin/mecocoa/mcca-riscv64 /home/ayano/_obj/mcca/riscv64/kernel-riscv64.o /home/ayano/_obj/mcca/riscv64/successor.o
riscv64-unknown-elf-objdump -S /mnt/hgfs/_bin/mecocoa/mcca-riscv64 > /mnt/hgfs/_bin/mecocoa/mcca-riscv64.asm
riscv64-unknown-elf-objdump -t /mnt/hgfs/_bin/mecocoa/mcca-riscv64 | sed '1,/SYMBOL TABLE/d; s/ .* / /; /^$/d' > /mnt/hgfs/_bin/mecocoa/mcca-riscv64.sym
Build  : Finish
qemu-system-riscv64 -nographic -machine virt -bios ./depends/RustSBI/rustsbi-qemu.bin -kernel /mnt/hgfs/_bin/mecocoa/mcca-riscv64 -S -gdb tcp::15234 &
sleep 1
qemu-system-riscv64: -gdb tcp::15234: Failed to find an available port: Address already in use
riscv64-unknown-elf-gdb --init-command=cocheck/riscv64.gdbinit
GNU gdb (SiFive GDB 8.3.0-2020.04.1) 8.3
Copyright (C) 2019 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "--host=x86_64-linux-gnu --target=riscv64-unknown-elf".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://github.com/sifive/freedom-tools/issues>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
The target architecture is assumed to be riscv:rv64

warning: No executable has been specified and target does not support
determining executable automatically.  Try using the "file" command.
0x0000000080200012 in ?? ()
Breakpoint 1 at 0x1000
Breakpoint 2 at 0x80200000: file riscv64/successor.S, line 6.
```

```bash
(gdb) x/12i 0x1000   # 从 0x1000 处反汇编出 12 条指令
   0x1000:	auipc	t0,0x0 # 将一个20位的立即数符号扩展为32位，然后将其加到PC的高20位中，生成一个32位的地址。
   0x1004:	addi	a2,t0,40 # 把一个寄存器的值和一个12位的立即数相加，并把结果存入另一个寄存器
   0x1008:	csrr	a0,mhartid # 将当前线程的ID加载到`a0`寄存器中 マ
   0x100c:	ld	a1,32(t0) # a1 = *(t0+0x20)
   0x1010:	ld	t0,24(t0) # t0 = *(t0+0x18); t0 is 0x80000000
   0x1014:	jr	t0 
   0x1018:	unimp # NUL 的反汇编， 此处为 dword: 0x80000000
   0x101a:	0x8000 
   0x101c:	unimp # NUL ...
   0x101e:	unimp
   0x1020:	unimp # dword 0x87000000
   0x1022:	0x8700

(gdb) c
Continuing.
[rustsbi] RustSBI version 0.3.0-alpha.2, adapting to RISC-V SBI v1.0.0
		……此处为 RustSBI 的输出 ……

Breakpoint 2, skernel () at riscv64/successor.S:6
6		la sp, boot_stack_top
1: x/12i $pc-8
   0x801ffff8:	unimp
   0x801ffffa:	unimp
   0x801ffffc:	unimp
   0x801ffffe:	unimp
=> 0x80200000 <skernel>:	auipc	sp,0x11
   0x80200004 <skernel+4>:	mv	sp,sp
   0x80200008 <skernel+8>:	jal	ra,0x8020000c <main>
   0x8020000c <main>:	addi	sp,sp,-16
   0x8020000e <main+2>:	sd	s0,8(sp)
   0x80200010 <main+4>:	addi	s0,sp,16
   0x80200012 <main+6>:	j	0x80200012 <main+6> # while (1);
   0x80200014:	unimp

```


