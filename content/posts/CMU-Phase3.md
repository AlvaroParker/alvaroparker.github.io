+++
title="CMU Bomb Lab Phase 3"
date="2024-09-03T17:07:25.000Z"

[taxonomies]
tags = ["CMU Bomb", "Reverse Engineering", "Code"]
+++

# CMU Bomb Lab reverse engineering

## Phase 3

We have reached phase 3. Let's analyze the function `phase_3` that's on the binary.

_Note: We are using the Intel assembly syntax now_

```nasm
0x08048b98 <+0>:     push   ebp
0x08048b99 <+1>:     mov    ebp,esp
0x08048b9b <+3>:     sub    esp,0x14
0x08048b9e <+6>:     push   ebx

0x08048b9f <+7>:     mov    edx,DWORD PTR [ebp+0x8]
0x08048ba2 <+10>:    add    esp,0xfffffff4
0x08048ba5 <+13>:    lea    eax,[ebp-0x4]
0x08048ba8 <+16>:    push   eax
0x08048ba9 <+17>:    lea    eax,[ebp-0x5]
0x08048bac <+20>:    push   eax
0x08048bad <+21>:    lea    eax,[ebp-0xc]
0x08048bb0 <+24>:    push   eax
0x08048bb1 <+25>:    push   0x80497de
0x08048bb6 <+30>:    push   edx
0x08048bb7 <+31>:    call   0x8048860 <sscanf@plt>

0x08048bbc <+36>:    add    esp,0x20
0x08048bbf <+39>:    cmp    eax,0x2
0x08048bc2 <+42>:    jg     0x8048bc9 <phase_3+49>
0x08048bc4 <+44>:    call   0x80494fc <explode_bomb>
0x08048bc9 <+49>:    cmp    DWORD PTR [ebp-0xc],0x7
0x08048bcd <+53>:    ja     0x8048c88 <phase_3+240>
0x08048bd3 <+59>:    mov    eax,DWORD PTR [ebp-0xc]
0x08048bd6 <+62>:    jmp    DWORD PTR [eax*4+0x80497e8]
0x08048bdd <+69>:    lea    esi,[esi+0x0]
0x08048be0 <+72>:    mov    bl,0x71
0x08048be2 <+74>:    cmp    DWORD PTR [ebp-0x4],0x309
0x08048be9 <+81>:    je     0x8048c8f <phase_3+247>
0x08048bef <+87>:    call   0x80494fc <explode_bomb>
0x08048bf4 <+92>:    jmp    0x8048c8f <phase_3+247>
0x08048bf9 <+97>:    lea    esi,[esi+eiz*1+0x0]
0x08048c00 <+104>:   mov    bl,0x62
0x08048c02 <+106>:   cmp    DWORD PTR [ebp-0x4],0xd6
0x08048c09 <+113>:   je     0x8048c8f <phase_3+247>
0x08048c0f <+119>:   call   0x80494fc <explode_bomb>
0x08048c14 <+124>:   jmp    0x8048c8f <phase_3+247>
0x08048c16 <+126>:   mov    bl,0x62
0x08048c18 <+128>:   cmp    DWORD PTR [ebp-0x4],0x2f3
0x08048c1f <+135>:   je     0x8048c8f <phase_3+247>
0x08048c21 <+137>:   call   0x80494fc <explode_bomb>
0x08048c26 <+142>:   jmp    0x8048c8f <phase_3+247>
0x08048c28 <+144>:   mov    bl,0x6b
0x08048c2a <+146>:   cmp    DWORD PTR [ebp-0x4],0xfb
0x08048c31 <+153>:   je     0x8048c8f <phase_3+247>
0x08048c33 <+155>:   call   0x80494fc <explode_bomb>
0x08048c38 <+160>:   jmp    0x8048c8f <phase_3+247>
0x08048c3a <+162>:   lea    esi,[esi+0x0]
0x08048c40 <+168>:   mov    bl,0x6f
0x08048c42 <+170>:   cmp    DWORD PTR [ebp-0x4],0xa0
0x08048c49 <+177>:   je     0x8048c8f <phase_3+247>
0x08048c4b <+179>:   call   0x80494fc <explode_bomb>
0x08048c50 <+184>:   jmp    0x8048c8f <phase_3+247>
0x08048c52 <+186>:   mov    bl,0x74
0x08048c54 <+188>:   cmp    DWORD PTR [ebp-0x4],0x1ca
0x08048c5b <+195>:   je     0x8048c8f <phase_3+247>
0x08048c5d <+197>:   call   0x80494fc <explode_bomb>
0x08048c62 <+202>:   jmp    0x8048c8f <phase_3+247>
0x08048c64 <+204>:   mov    bl,0x76
0x08048c66 <+206>:   cmp    DWORD PTR [ebp-0x4],0x30c
0x08048c6d <+213>:   je     0x8048c8f <phase_3+247>
0x08048c6f <+215>:   call   0x80494fc <explode_bomb>
0x08048c74 <+220>:   jmp    0x8048c8f <phase_3+247>
0x08048c76 <+222>:   mov    bl,0x62
0x08048c78 <+224>:   cmp    DWORD PTR [ebp-0x4],0x20c
0x08048c7f <+231>:   je     0x8048c8f <phase_3+247>
0x08048c81 <+233>:   call   0x80494fc <explode_bomb>
0x08048c86 <+238>:   jmp    0x8048c8f <phase_3+247>
0x08048c88 <+240>:   mov    bl,0x78
0x08048c8a <+242>:   call   0x80494fc <explode_bomb>
0x08048c8f <+247>:   cmp    bl,BYTE PTR [ebp-0x5]
0x08048c92 <+250>:   je     0x8048c99 <phase_3+257>
0x08048c94 <+252>:   call   0x80494fc <explode_bomb>
0x08048c99 <+257>:   mov    ebx,DWORD PTR [ebp-0x18]
0x08048c9c <+260>:   mov    esp,ebp
0x08048c9e <+262>:   pop    ebp
0x08048c9f <+263>:   ret
```

We can see the function prologue from instruction `0` to instruction `6`. After this we set up a call to the function `sscanf` by pushing the parameters to the stack:

```nasm
0x08048b9f <+7>:     mov    edx,DWORD PTR [ebp+0x8]
0x08048ba2 <+10>:    add    esp,0xfffffff4
0x08048ba5 <+13>:    lea    eax,[ebp-0x4]
0x08048ba8 <+16>:    push   eax
0x08048ba9 <+17>:    lea    eax,[ebp-0x5]
0x08048bac <+20>:    push   eax
0x08048bad <+21>:    lea    eax,[ebp-0xc]
0x08048bb0 <+24>:    push   eax
0x08048bb1 <+25>:    push   0x80497de
0x08048bb6 <+30>:    push   edx
0x08048bb7 <+31>:    call   0x8048860 <sscanf@plt>
```

We push 5 parameters on reverse order, meaning that the first parameters pushed is the last function parameter. The last two parameters `%edx` and `0x80497de` are the memory address of the input string and the format string in that order.

The first 3 parameters are the pointers where we will save the scanned elements.

We can check the format string to see what the `phase_3` function expects our input to be:

```
(gdb) x/s 0x80497de
0x80497de:      "%d %c %d"
```

So we can see the `phase_3` expects an integer, followed by a spaces, then a character, then a space and finally an integer.

After scanning the two numbers and the char, we check that the scanned was sucesfull by checking the return value of `sscanf` (stored in the `eax` register) is greater than `2`:

```nasm
0x8048bbc <phase_3+36>  add    esp,0x20
0x8048bbf <phase_3+39>  cmp    eax,0x2 # <- we compare it with two
0x8048bc2 <phase_3+42>  jg     0x8048bc9 <phase_3+49>
0x8048bc4 <phase_3+44>  call   0x80494fc <explode_bomb>
>0x8048bc9 <phase_3+49>  cmp    DWORD PTR [ebp-0xc],0x7
```

If the value is greater than two, meaning the scan was a success, we continue, if not, we explode the bomb

Then we make a second compartion, with `0x7` and `ebp-0xc` (the first number we provided). If the first number we suplide is bigger than 7 the bomb explotes (we jump to instruction `240` which is the function to explode the bomb)

```nasm
0x8048bc9 <phase_3+49>  cmp    DWORD PTR [ebp-0xc],0x7
0x8048bcd <phase_3+53>  ja     0x8048c88 <phase_3+240>
0x8048bd3 <phase_3+59>  mov    eax,DWORD PTR [ebp-0xc]
0x8048bd6 <phase_3+62>  jmp    DWORD PTR [eax*4+0x80497e8]
```

If we provided a value less or equal than 7, we continue to instruction `59` which moves the first value we provided, to the register `%eax`. Then we find a `jpm` instruction that depends on the value that's at the `eax` register.
This is also known as a switch statement in C.

Ok, so far we figured that we have to provide an input in the format of `<integer> <character> <integer>` and that the first integer we provide has to be less or equal than 7.

Let's check what happens if the first number is, for example 6. We jump to the instruction `[0x6*4+0x80497e8]`. To check this, our input will be of `6 c 10`
We end on the following instrucion:

```nasm
>0x8048c64 <phase_3+204> mov    bl,0x76
0x8048c66 <phase_3+206> cmp    DWORD PTR [ebp-0x4],0x30c
0x8048c6d <phase_3+213> je     0x8048c8f <phase_3+247>
0x8048c6f <phase_3+215> call   0x80494fc <explode_bomb>
0x8048c74 <phase_3+220> jmp    0x8048c8f <phase_3+247>
```

The instruction `204` moves the value `0x76` to the `bl` register, the following expresion compares the second number that we input with the value `0x30c` and if they are equal we jump to the instruction `247`, and if they are not equal, the bomb explotes.

Great, now we now that if our first number has to be equal to `7` and our second number has to be equal to `0x30c` which is `780` in decimal value.

Let's restart our program with an input `"6 c 780"` to be able to jump to the instruction `247`:

```nasm
>0x8048c8f <phase_3+247> cmp    bl,BYTE PTR [ebp-0x5]
0x8048c92 <phase_3+250> je     0x8048c99 <phase_3+257>
0x8048c94 <phase_3+252> call   0x80494fc <explode_bomb>
0x8048c99 <phase_3+257> mov    ebx,DWORD PTR [ebp-0x18]
0x8048c9c <phase_3+260> mov    esp,ebp
0x8048c9e <phase_3+262> pop    ebp
0x8048c9f <phase_3+263> ret
```

We are almost at the end of the `phase_3` function! We can see that the instruction `247` compares the register we set on the previous section (`bl` register) with the value of the character we input (accesible via `BYTE PTR [ebp-0x5]`) and if they are equal, it jumps to the end of the function, and if not, the bomb explotes.

Awesome, so now we know that our character has to have the ASCII value of the `be` register we set on the previous section, which in this case is `0x76`. Note that if the first input number had being different, then the second number and the character would also have to be different, their value would be given by the corresponding switch statement evaluation.

The character that has `0x76` as ACII code is `v`.

Now let's test this phase solution!

```
$ ./bomb
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Public speaking is very easy.
Phase 1 defused. How about the next one?
1 2 6 24 120 720
That's number 2.  Keep going!
6 v 780
Halfway there!
```

Our solution works!!
