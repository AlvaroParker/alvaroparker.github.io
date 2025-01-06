+++
title="CMU Bomb Lab Phase 4"
date="2024-09-04T17:07:25.000Z"

[taxonomies]
tags = ["CMU Bomb", "Reverse Engineering", "Code"]
+++

# CMU Bomb Lab reverse engineering

## Phase 4

We are now on the phase 4, let's analyze the function `phase_4` by running `disass phase_4` on `gdb`:

```nasm
0x08048ce0 <+0>:     push   ebp
0x08048ce1 <+1>:     mov    ebp,esp
0x08048ce3 <+3>:     sub    esp,0x18
0x08048ce6 <+6>:     mov    edx,DWORD PTR [ebp+0x8]
0x08048ce9 <+9>:     add    esp,0xfffffffc
0x08048cec <+12>:    lea    eax,[ebp-0x4]
0x08048cef <+15>:    push   eax
0x08048cf0 <+16>:    push   0x8049808
0x08048cf5 <+21>:    push   edx
0x08048cf6 <+22>:    call   0x8048860 <sscanf@plt>
0x08048cfb <+27>:    add    esp,0x10
0x08048cfe <+30>:    cmp    eax,0x1
0x08048d01 <+33>:    jne    0x8048d09 <phase_4+41>
0x08048d03 <+35>:    cmp    DWORD PTR [ebp-0x4],0x0
0x08048d07 <+39>:    jg     0x8048d0e <phase_4+46>
0x08048d09 <+41>:    call   0x80494fc <explode_bomb>
0x08048d0e <+46>:    add    esp,0xfffffff4
0x08048d11 <+49>:    mov    eax,DWORD PTR [ebp-0x4]
0x08048d14 <+52>:    push   eax
0x08048d15 <+53>:    call   0x8048ca0 <func4>
0x08048d1a <+58>:    add    esp,0x10
0x08048d1d <+61>:    cmp    eax,0x37
0x08048d20 <+64>:    je     0x8048d27 <phase_4+71>
0x08048d22 <+66>:    call   0x80494fc <explode_bomb>
0x08048d27 <+71>:    mov    esp,ebp
0x08048d29 <+73>:    pop    ebp
0x08048d2a <+74>:    ret
```

Between instructions `0` and `9` is our function prologue. Then the next instructions push the parameters to call the `sscanf` function:

```nasm
0x08048cec <+12>:    lea    eax,[ebp-0x4]
0x08048cef <+15>:    push   eax
0x08048cf0 <+16>:    push   0x8049808
0x08048cf5 <+21>:    push   edx
0x08048cf6 <+22>:    call   0x8048860 <sscanf@plt>
# Instructions to checck the return value of sscanf
0x08048cfb <+27>:    add    esp,0x10
0x08048cfe <+30>:    cmp    eax,0x1
0x08048d01 <+33>:    jne    0x8048d09 <phase_4+41>
0x08048d03 <+35>:    cmp    DWORD PTR [ebp-0x4],0x0
0x08048d07 <+39>:    jg     0x8048d0e <phase_4+46>
0x08048d09 <+41>:    call   0x80494fc <explode_bomb>
```

We load the effective address `ebp-0x4` and we push that address in the stack. This is a pointer to the scanned value that `sscanf` will produce. Next we push an address `0x8049808`, let's check what's the content of this address using `gdb`:

```
(gdb) x/s 0x8049808
0x8049808:      "%d"
```

Huh, so the format string that we pass to `sscanf` is `"%d"` which means the function expects a number!

We can also check the memory that `edx` register is pointing to:

```
(gdb) x/s $edx
0x804b770 <input_strings+240>:  "10"
```

So we now know that we have a function call that in C might look like this:

```c
int num;
sscanf("<input string>", "%d", &num);
```

After this, the instructions `30` and `33` check if the function `sscanf` succesfully scanned the input string, and if not, it triggers the bomb by jumping to instruction `41`. After this, if our `sscanf` was a success, we go to instruction
instruction `35` which checks if our input is greater than `0`, if it's not, the bomb explotes by continuing to instruction `41`, if it is greater than 0, we go to instruction `46`

Let's analyze the next portion of the `phase_4` function, assuming that our input was correct and greater than 0:

```nasm
0x08048d0e <+46>:    add    esp,0xfffffff4
0x08048d11 <+49>:    mov    eax,DWORD PTR [ebp-0x4]
0x08048d14 <+52>:    push   eax
0x08048d15 <+53>:    call   0x8048ca0 <func4>
0x08048d1a <+58>:    add    esp,0x10
0x08048d1d <+61>:    cmp    eax,0x37
0x08048d20 <+64>:    je     0x8048d27 <phase_4+71>
0x08048d22 <+66>:    call   0x80494fc <explode_bomb>
0x08048d27 <+71>:    mov    esp,ebp
0x08048d29 <+73>:    pop    ebp
0x08048d2a <+74>:    ret
```

We allocate some space on the stack on the first instruction, we move the number we scanned with the previous `sscanf` call to the `eax` register, we push the register to the stack and we call the `func4` function.

After this function returns, we compare the return value it produced (that's on the `eax` register) with the literal `0x37`. If the return value is equal to `0x37` we jump to instruction `71` which returns the `phase_4` function, hence passing this phase.

Ok, so far we know that we have to input a number on phase 4, that number has to be greater than 0, and then this number is used as a parameter to call the `func4` function, which returns a value that is then compared with the literal `0x37`, if the return value is
equal to `0x37` we pass the phase, if not, we explote the bomb.

Let's analyze `func4` to check what our input value should be such that the `func4` returns a value equal to `0x37`. Disassembling the function with `disass func4` returns:

```nasm
0x08048ca0 <+0>:     push   ebp
0x08048ca1 <+1>:     mov    ebp,esp
0x08048ca3 <+3>:     sub    esp,0x10
0x08048ca6 <+6>:     push   esi
0x08048ca7 <+7>:     push   ebx
0x08048ca8 <+8>:     mov    ebx,DWORD PTR [ebp+0x8]
0x08048cab <+11>:    cmp    ebx,0x1
0x08048cae <+14>:    jle    0x8048cd0 <func4+48>
0x08048cb0 <+16>:    add    esp,0xfffffff4
0x08048cb3 <+19>:    lea    eax,[ebx-0x1]
0x08048cb6 <+22>:    push   eax
0x08048cb7 <+23>:    call   0x8048ca0 <func4>
0x08048cbc <+28>:    mov    esi,eax
0x08048cbe <+30>:    add    esp,0xfffffff4
0x08048cc1 <+33>:    lea    eax,[ebx-0x2]
0x08048cc4 <+36>:    push   eax
0x08048cc5 <+37>:    call   0x8048ca0 <func4>
0x08048cca <+42>:    add    eax,esi
0x08048ccc <+44>:    jmp    0x8048cd5 <func4+53>
0x08048cce <+46>:    mov    esi,esi
0x08048cd0 <+48>:    mov    eax,0x1
0x08048cd5 <+53>:    lea    esp,[ebp-0x18]
0x08048cd8 <+56>:    pop    ebx
0x08048cd9 <+57>:    pop    esi
0x08048cda <+58>:    mov    esp,ebp
0x08048cdc <+60>:    pop    ebp
0x08048cdd <+61>:    ret
```

There are 3 main part on this function, the prologue (+ some other instructions) the central part of the function which is the one that produces the return value, and the epilogue.

Let's analyze the prologue:

```nasm
0x08048ca0 <+0>:     push   ebp
0x08048ca1 <+1>:     mov    ebp,esp
0x08048ca3 <+3>:     sub    esp,0x10
0x08048ca6 <+6>:     push   esi
0x08048ca7 <+7>:     push   ebx
0x08048ca8 <+8>:     mov    ebx,DWORD PTR [ebp+0x8]
0x08048cab <+11>:    cmp    ebx,0x1
0x08048cae <+14>:    jle    0x8048cd0 <func4+48>
```

Between instructions `0` and `7` we see the function prologue and some stack allocation. After this, on instruction `8` we load the parameter of the function `func4` to the register `ebx`. This means that if in phase 4 we input a string `"5"`, then the ebx register will store an int value `5`.

After this, on instruction `11` it compares the function parameter with `1`, if they are less or equal than 1, then it jump to instructino `48` which is the start of the function prologue which returns the same value `ebx` .

Ok, so we now know that our function looks something like this in C code :

```C
int func4(int num) {
  if (num <= 1) {
    return num;
  } else {
    // Do some stuff
  }
}
```

Let's analyze what happens on the else, this is between instructions `16` and `44`

```nasm
0x08048cb0 <+16>:    add    esp,0xfffffff4
0x08048cb3 <+19>:    lea    eax,[ebx-0x1]
0x08048cb6 <+22>:    push   eax
0x08048cb7 <+23>:    call   0x8048ca0 <func4>
0x08048cbc <+28>:    mov    esi,eax
0x08048cbe <+30>:    add    esp,0xfffffff4
0x08048cc1 <+33>:    lea    eax,[ebx-0x2]
0x08048cc4 <+36>:    push   eax
0x08048cc5 <+37>:    call   0x8048ca0 <func4>
0x08048cca <+42>:    add    eax,esi
0x08048ccc <+44>:    jmp    0x8048cd5 <func4+53>
```

By taking a quick look, we see that we are calling the same `func4` function on `func4` body (on isntructions `23` and `37`), this means we are dealing with a recursive function.

On the first call, we substract `1` from `ebx` (our input parameter) by using `lea` instruction, we then store that result on `eax`, push `eax` to the stack and call `func4` with `eax` as a parameter. So we are calling `func4` with our original functino parameter minus 1.
After this, we save the return value of that function call into the `esi` register.

On the second call, we do something similar, but instead of calling `func4` with our original function parameter minus 1, we call it minus 2 (instruction `33` is `lea    eax,[ebx-0x2]` which substract `2` from our parameter and stores it in `eax`)

Finallly we add `esi` (return value of the first call) with `eax` (return value of the second call) and we jump to instruction `53`

Or C code now looks something like this:

```C
int func4(int num) {
  if (num <= 1) {
    return num;
  } else {
    return func4(num - 1) - func4(num - 2);
  }
}
```

Does this function sounds familiar? (Hint: It's the Fibonacci sequence)

Great, so now we know that our input on phase 4 has to be an integer greater than 0, that if we put it into the fibonacci function, it returns `0x37` which is `55` in decimal. This number is 10, because the 10th value of the Fibonacci sequence is 55.

However this function adds 1 from our input, so if we input 10, it will return the 11th number of the Fibonacci sequence, so the get `55` we have to input `9`.

Final solution then is `9`!

_note: Idk why it does one more iteration of the sequence, for example if 10 is provided, it returns 89 isntead of 55_
