+++
title="CMU Bomb Lab Phase 5"
date="2024-09-05T17:07:25.000Z"

[taxonomies]
tags = ["CMU Bomb", "Reverse Engineering", "Code"]
+++

# CMU Bomb Lab reverse engineering

## Phase 5

We've reached phase 5. Let's analyze the `phase_5` function:

```nasm
# Section 1
0x08048d2c <+0>:     push   ebp
0x08048d2d <+1>:     mov    ebp,esp
0x08048d2f <+3>:     sub    esp,0x10
0x08048d32 <+6>:     push   esi
0x08048d33 <+7>:     push   ebx
0x08048d34 <+8>:     mov    ebx,DWORD PTR [ebp+0x8]
0x08048d37 <+11>:    add    esp,0xfffffff4
0x08048d3a <+14>:    push   ebx
0x08048d3b <+15>:    call   0x8049018 <string_length>
0x08048d40 <+20>:    add    esp,0x10
0x08048d43 <+23>:    cmp    eax,0x6
0x08048d46 <+26>:    je     0x8048d4d <phase_5+33>
0x08048d48 <+28>:    call   0x80494fc <explode_bomb>

# Section 2
0x08048d4d <+33>:    xor    edx,edx # set edx to 0
0x08048d4f <+35>:    lea    ecx,[ebp-0x8] # Load address ebp-0x8 to ecx
0x08048d52 <+38>:    mov    esi,0x804b220 # load string pointer to esi

# Section 3
0x08048d57 <+43>:    mov    al,BYTE PTR [edx+ebx*1] # `ebx` stores the pointer to our string. This acceses character at position edx of the string and loads it to al
0x08048d5a <+46>:    and    al,0xf # Make & operation with the character and 0xf which is 15 in decimal
0x08048d5c <+48>:    movsx  eax,al # Move al register to eax register, extending from
0x08048d5f <+51>:    mov    al,BYTE PTR [eax+esi*1] # move character at position eax of string at esi to register al
0x08048d62 <+54>:    mov    BYTE PTR [edx+ecx*1],al # move al register to
0x08048d65 <+57>:    inc    edx # increase edx counter
0x08048d66 <+58>:    cmp    edx,0x5 # If we reached edx counter = 5
0x08048d69 <+61>:    jle    0x8048d57 <phase_5+43> jump to instruction 43

# Section 4
0x08048d6b <+63>:    mov    BYTE PTR [ebp-0x2],0x0
0x08048d6f <+67>:    add    esp,0xfffffff8
0x08048d72 <+70>:    push   0x804980b
0x08048d77 <+75>:    lea    eax,[ebp-0x8]
0x08048d7a <+78>:    push   eax
0x08048d7b <+79>:    call   0x8049030 <strings_not_equal>
0x08048d80 <+84>:    add    esp,0x10
0x08048d83 <+87>:    test   eax,eax
0x08048d85 <+89>:    je     0x8048d8c <phase_5+96>
0x08048d87 <+91>:    call   0x80494fc <explode_bomb>
0x08048d8c <+96>:    lea    esp,[ebp-0x18]
0x08048d8f <+99>:    pop    ebx
0x08048d90 <+100>:   pop    esi
0x08048d91 <+101>:   mov    esp,ebp
0x08048d93 <+103>:   pop    ebp
0x08048d94 <+104>:   ret
```

We can split the function in 4 main sections.

In the first section (instruction `0` to instruction `28`) we have the functino prologue, we allocate some space in the stack and we check the length of the input string we gave to the `phase_5` function using the `string_length` function. If the length is equal to `0x6` (`6` in decimal) we continue, if not, the bomb explotes.

```nasm
0x08048d2c <+0>:     push   ebp
0x08048d2d <+1>:     mov    ebp,esp
0x08048d2f <+3>:     sub    esp,0x10
0x08048d32 <+6>:     push   esi
0x08048d33 <+7>:     push   ebx
0x08048d34 <+8>:     mov    ebx,DWORD PTR [ebp+0x8]
0x08048d37 <+11>:    add    esp,0xfffffff4
0x08048d3a <+14>:    push   ebx
0x08048d3b <+15>:    call   0x8049018 <string_length>
0x08048d40 <+20>:    add    esp,0x10
0x08048d43 <+23>:    cmp    eax,0x6
0x08048d46 <+26>:    je     0x8048d4d <phase_5+33>
0x08048d48 <+28>:    call   0x80494fc <explode_bomb>
```

We also see that the `ebx` register has the input we passed on the phase 5. We can verify this in gdb doing:

```
(gdb) x/s $ebx
0x804b7c0 <input_strings+320>:  "hello"
```

Then we have the second section:

```nasm
# Section 2
0x08048d4d <+33>:    xor    edx,edx # set edx to 0
0x08048d4f <+35>:    lea    ecx,[ebp-0x8] # Load address ebp-0x8 to ecx
0x08048d52 <+38>:    mov    esi,0x804b220 # load string pointer to esi
```

On the first instruction we set the `edx` register to `0`, then we load the address `ebp-0x8` to the `ecx` register and finally we load the number `0x804b220` to the register `esi`, this address points to a string (`char *`) which we canc check by doing:

```
(gdb) x/s 0x804b220
0x804b220:      "isrveawhobpnutfg"
```

Now, in the third section, thing get interesting.

```nasm
0x08048d57 <+43>:    mov    al,BYTE PTR [edx+ebx*1] # `ebx` stores the pointer to our string. This acceses character at position edx of the string and loads it to al
0x08048d5a <+46>:    and    al,0xf # Make & operation with the character and 0xf which is 15 in decimal
0x08048d5c <+48>:    movsx  eax,al # Move al register to eax register, extending from
0x08048d5f <+51>:    mov    al,BYTE PTR [eax+esi*1] # move character at position eax of string at esi to register al
0x08048d62 <+54>:    mov    BYTE PTR [edx+ecx*1],al # move al register to
0x08048d65 <+57>:    inc    edx # increase edx counter
0x08048d66 <+58>:    cmp    edx,0x5 # If we reached edx counter = 5
0x08048d69 <+61>:    jle    0x8048d57 <phase_5+43> jump to instruction 43
```

If we look at the last third instructions of this section, we can asume that this section is a loop, since we increase a counter (`edx` register), check if the counter is bigger than some value and if it's not, we jump back to the beginning of the section.

Ok, so now we know that we are dealing with a function that has a loop inside, but what does the loop do? Let's figure it out.

We know the counter that increases for each loop is the `edx` register. We also use this register to access a specific char (`BYTE`) on the string we input on the phase 5 and move that char to the register `al`:

```nasm
0x08048d57 <+43>:    mov    al,BYTE PTR [edx+ebx*1] # `ebx` stores the pointer to our string. This acceses character at position edx of the string and loads it to al
0x08048d5a <+46>:    and    al,0xf # Make & operation with the character and 0xf which is 15 in decimal
```

We then make an `and` operation with that char and the number `0xf` (`15` in decimal) and store it back to the `al` register.

After this, we move the `al` value to the `eax` register.

Then on instruction `51` and `54` we have:

```nasm
0x08048d5f <+51>:    mov    al,BYTE PTR [eax+esi*1] # move character at position eax of string at esi to register al
0x08048d62 <+54>:    mov    BYTE PTR [edx+ecx*1],al # move al register to
```

Remember, the `esi` register has a pointer to the string literal `"isrveawhobpnutfg"`, so we are now accesing a character on that string by the index number that's on the `eax` register and moving that character to the `al` register.

Finally we move the value at the `al` register to the memory address that's on `ecx` with an offset given by the current `edx` value (which increases for each iteration on the loop)

After this, we increase the `edx` value and if it's greater than 5, we break the loop, if not, we continue.

After breaking the loop (`edx` register is now greater than 5) we go to the section 4 of the function:

```nasm
0x08048d6b <+63>:    mov    BYTE PTR [ebp-0x2],0x0
0x08048d6f <+67>:    add    esp,0xfffffff8
0x08048d72 <+70>:    push   0x804980b
0x08048d77 <+75>:    lea    eax,[ebp-0x8]
0x08048d7a <+78>:    push   eax
0x08048d7b <+79>:    call   0x8049030 <strings_not_equal>
0x08048d80 <+84>:    add    esp,0x10
0x08048d83 <+87>:    test   eax,eax
0x08048d85 <+89>:    je     0x8048d8c <phase_5+96>
0x08048d87 <+91>:    call   0x80494fc <explode_bomb>
0x08048d8c <+96>:    lea    esp,[ebp-0x18]
0x08048d8f <+99>:    pop    ebx
0x08048d90 <+100>:   pop    esi
0x08048d91 <+101>:   mov    esp,ebp
0x08048d93 <+103>:   pop    ebp
0x08048d94 <+104>:   ret
```

In this section we are calling a function `strings_not_equal` with a pointer to a string literal on address `0x804980b` and the string we build which memory address is
on the register `eax`

After calling this function, we test if the strings are equal, if they are, we continue execution on instruction `96`, if they are not equal, the bomb explotes.

We can check the value of the string literal we are comparing our new string:

```
(gdb) x/s 0x804980b
0x804980b:      "giants"
```

Hmm, so our new string has to be equal to the string `"giant"`.

Now we know that when we enter phase 6 and give an input, it has to be of length 6, the input will be used to iterate over each character and do an binary `and` operation with the character and `0xf` and the result of that will be the index we will use to access a char in the string `"isrveawhobpnutfg"`, then we will take that char and append it to the new string. Once our string has length 6, we break the loop and check if it's equal to `"giants"`, if it is, we pass phase 5, if it's not, the bomb explotes.

We can build the algorithm used to create the new string in python:

```python
new_str = "";
random_string = "isrveawhobpnutfg"
for i in my_str:
  index = i & 0xf
  new_str += random_string[index]
return new_str
```

To build our string, we have to access index `[15, 0, 5, 11, 13, 1]` in that order so for each of those values we have to find an `x` wich when doing `x & 0xf` returns the value on that array.

```
for each n in [15, 0, 5, 11, 13, 1] we need x such that x % 0xf == n
```

We also want `x` to be between `32` and `122` to be a nice printable ASCII value.

To get the list of valid number we can do a simple python script:

```python
from typing import List

values = [15, 0, 5, 11, 13, 1]


def get_valid_chars(x: int) -> List[str]:
    valid = []
    for i in range(32, 122, 1):
        if i & 0xF == x:
            valid.append(chr(i))
    return valid


for i in values:
    print(get_valid_chars(i))
```

Running the script

```
$ python solve.py
['/', '?', 'O', '_', 'o']
[' ', '0', '@', 'P', '`', 'p']
['%', '5', 'E', 'U', 'e', 'u']
['+', ';', 'K', '[', 'k']
['-', '=', 'M', ']', 'm']
['!', '1', 'A', 'Q', 'a', 'q']
```

So for each list on that output we choose one character, an arbitrary solution would be `"?pukmq"`
