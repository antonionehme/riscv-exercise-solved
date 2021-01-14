# CHERI exercise

This is a small exercise to get started with CHERI (on RISC-V, QEMU emulation). 

## Recommended setup

While QEMU with CHERI-RISC-V should run on most Linux/Unix/Mac platforms, we recommend using Ubuntu 18.04 - if needed you can do this using a VM (for example from https://www.osboxes.org/ubuntu/#ubuntu-1804-vbox). Note that the tools take a while to build (several hours depending on CPU etc), so plan in some time to wait for the compilation to finish.

## Resources

The following resources by the CHERI team from Cambridge are useful:

 * The getting started guide, including installation instructions for the emulator etc: https://ctsrd-cheri.github.io/cheri-exercises/cover/index.html 
 * The `cheribuild` tool: https://github.com/CTSRD-CHERI/cheribuild.git
 * How to copy files in/out of QEMU to the host is documented here: https://github.com/CTSRD-CHERI/cheri-exercises/pull/26
   Essentially, you can simply use `mount_smbfs -I 10.0.2.4 -N //10.0.2.4/source_root /mnt` in the QEMU guest, which will mount the CHERI base directory of the host on `/mnt`.

Also, if you ever need to exit QEMU: press `Ctrl-a` then release and press `x`   

## Task

We will use a (slightly modified) exercise from https://github.com/CTSRD-CHERI/cheri-exercises/

 * Fork this repository here (*not* the CHERI exercise one) - we expect you to add your solutions in this README.md where it says *INSERT SOLUTION HERE*. Please make sure you do reasonable commits and commit messages. You can also use other features of Github e.g. issues.
 
 * Assuming that you have installed CHERI-RISC-V in `~/cheri`, make sure your forked repo is cloned to `~/cheri/riscv-exercise`
 
 * Compile `buffer-overflow.c` to a RISC-V binary `buffer-overflow-hybrid` in hybrid capability mode (`riscv64-hybrid`). You can use the `ccc` script from `task/tools` (see the exercise docs for details) for that. What is the full commandline for compilation? 
 
 ```
 sh tools/ccc riscv64-hybrid -G0 buffer-overflow.c -o buffer-overflow-hybrid
 ```
 
 * There is a security flaw in `buffer-overflow.c`. Briefly explain what the flaw is: 
 
 ```
 The variable c was declared and assigned a value in memory straight after memset for the buffer. As a result, c will in stored in a register close to the buffer. Sending an input that is larger than the memory allocated to the buffer (16 in this case) leads to overriding the content of the adjacent memory addresses.
 ```
 
 * Start CHERI-RISC-V in QEMU, copy `buffer-overflow-hybrid` to the QEMU guest, and run it with a commandline argument that triggers the mentioned security flaw to overwrite the variable `c` with an attacker-controlled value. Give all the commands you have to run (assuming CHERI is in `~/cheri` and cheribuild in `~/cheribuild`):
 
  ```
  The commands are:
  `python3 cheribuild.py run-riscv64-hybrid` /*to start the Qemu image*/
  `mount_smbfs -I 10.0.2.4 -N //10.0.2.4/source_root /mnt` to mount the CHERI base directory of the host on `/mnt
  cd ../mnt/riscv-exervice/task
  `./buffer-overflow-hybrid '-----------------------A'` to replace c with 'A'
  ```
  
 * Now, compile the same program in pure capability mode (`riscv64-purecap`) to `buffer-overflow-purecap`. What happens when you run this program in QEMU with the same input that triggered the flaw in `buffer-overflow-hybrid`? Explain why this happens!

 ```
 The execution throws an error with `./buffer-overflow-purecap '-----------------------A'`:
c = c
In-address space security exception
root@cheribsd-riscv64-hybrid:/mnt/riscv-exercise/task # Jan 13 14:22:55 cheribsd-riscv64-hybrid kernel: pid 733 (buffer-overflow-pur), uid (0): Failed to open coredump file 'buffer-overflow-pur.core', error=13

This is due to the spacial protection properties offered through Cheri Pure-Capability:
The narrow bounds imposed by the compiler when using the pure capability mode prevents the use of pointers to manipulate unintended objects (variable c in this case).

 ```
