

## Unzipping the file

Unzipping the downloaded zip file.

```bash
$ unzip 86a6dc4f-df68-4208-bf9a-60f83c69d53c.zip 
Archive:  86a6dc4f-df68-4208-bf9a-60f83c69d53c.zip
  inflating: main
```

## Binary Information

Finding the type of binary we are dealing with.

```bash
$ file main                                                  
main: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=19668e4bb57624e34a91b4441c5372aec13972e1, for GNU/Linux 3.2.0, not stripped
```

## Dynamic Analysis

So after running the binary we understand that we need to find **flag1** and **flag2** to get the entire flag value.

```bash
$ ./main
Use debugging to find flag1 and flag2..
If you find flag1 and flag2, full flag is DH{flag1-flag2}
```

From the challenge instructions it says the `flag1` value is returned when `flag1()` function call returns and the value gets stored in `rax` register. We can set breakpoint at the `flag1()` and skip the function inner instructions using `ni` and then examine the value of `rax` to get the `flag1` value. 


## Running GDB

```bash
$ gdb -q main
```

There is no `main` label so run the program using `start` command to go to the entry point of the program and then analyze the instructions present using `x/20i $rip`. You can choose your own value and get a rough idea of the instructions present. Below is what to got after trying it myself.

```bash
pwndbg> x/20i $rip
=> 0x40115e <main+8>:   lea    rax,[rip+0xea3]        # 0x402008
   0x401165 <main+15>:  mov    rdi,rax
   0x401168 <main+18>:  call   0x401050 <puts@plt>
   0x40116d <main+23>:  call   0x401198 <flag_1>
   0x401172 <main+28>:  movzx  eax,BYTE PTR [rip+0x2ea7]        # 0x404020 <a>
   0x401179 <main+35>:  test   al,al
   0x40117b <main+37>:  jne    0x401182 <main+44>
   0x40117d <main+39>:  call   0x401211 <flag_2>
   0x401182 <main+44>:  lea    rax,[rip+0xea7]        # 0x402030
   0x401189 <main+51>:  mov    rdi,rax
   0x40118c <main+54>:  call   0x401050 <puts@plt>
   0x401191 <main+59>:  mov    eax,0x0
   0x401196 <main+64>:  pop    rbp
   0x401197 <main+65>:  ret
   0x401198 <flag_1>:   endbr64
   0x40119c <flag_1+4>: push   rbp
   0x40119d <flag_1+5>: mov    rbp,rsp
   0x4011a0 <flag_1+8>: jmp    0x4011a1 <flag_1+9>
   0x4011a2 <flag_1+10>:        ror    BYTE PTR [rax-0x39],0x45
   0x4011a6 <flag_1+14>:        loopne 0x4011a8 <flag_1+16>
```

## Getting `flag1` 

After setting the breakpoint, continue `c` to go to that breakpoint and then use `ni` to step over the function call and then examine the `rax` register value to get the `flag1` value.

```bash
pwndbg> b *0x40116d
Breakpoint 2 at 0x40116d
```

## Getting `flag2`

Here is something interesting....

```bash
pwndbg> x/20i $rip
=> 0x401172 <main+28>:  movzx  eax,BYTE PTR [rip+0x2ea7]        # 0x404020 <a>
   0x401179 <main+35>:  test   al,al
   0x40117b <main+37>:  jne    0x401182 <main+44>
   0x40117d <main+39>:  call   0x401211 <flag_2>
   0x401182 <main+44>:  lea    rax,[rip+0xea7]        # 0x402030
   0x401189 <main+51>:  mov    rdi,rax
   0x40118c <main+54>:  call   0x401050 <puts@plt>
   0x401191 <main+59>:  mov    eax,0x0
   0x401196 <main+64>:  pop    rbp
   0x401197 <main+65>:  ret
   0x401198 <flag_1>:   endbr64
   0x40119c <flag_1+4>: push   rbp
   0x40119d <flag_1+5>: mov    rbp,rsp
   0x4011a0 <flag_1+8>: jmp    0x4011a1 <flag_1+9>
   0x4011a2 <flag_1+10>:        ror    BYTE PTR [rax-0x39],0x45
   0x4011a6 <flag_1+14>:        loopne 0x4011a8 <flag_1+16>
   0x4011a8 <flag_1+16>:        add    BYTE PTR [rax],al
   0x4011aa <flag_1+18>:        add    BYTE PTR [rax-0x48],cl
   0x4011ad <flag_1+21>:        xchg   edi,eax
   0x4011ae <flag_1+22>:        or     BYTE PTR [rdx+rdi*8-0x543f769],bh
pwndbg> b *0x40117d
Breakpoint 3 at 0x40117d
```

Check the below 3 instructions carefully.

```bash
0x401172 <main+28>:  movzx  eax,BYTE PTR [rip+0x2ea7]        # 0x404020 <a>
   0x401179 <main+35>:  test   al,al
   0x40117b <main+37>:  jne    0x401182 <main+44>
   0x40117d <main+39>:  call   0x401211 <flag_2>
```

The value of `al` is compared using `test` instruction which performs `bitwise AND` operation and if value of `al` is `0` we will get a **zero flag** but if `al` is not `0` then **zero flag** will not be received and we will never reach the `flag2` label. So we need to use `set` command to change our `al` register value.

```bash
pwndbg> set $rax=0
```

Of course when you hit the `flag2`label you can step over it using `ni` but this time the value will be in `rsi` . And remember that both `flag1` and `flag2` are 18 character long hexadecimal values which include the `0x` sign in the flag as well.

**NOW COMBINE `DH{flag1-flag2}`** to get the flag !!!


