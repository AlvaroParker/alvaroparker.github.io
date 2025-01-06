+++
title="CMU Bomb Lab Phase 2"
date="2024-09-02T17:07:25.000Z"

[taxonomies]
tags = ["CMU Bomb", "Reverse Engineering", "Code"]
+++

# CMU Bomb Lab reverse engineering

## Phase 2

Now we have to disassemble the `phase_2` function to be able to defuse the bomb:

```nasm
 0x08048b48 <+0>:     push   %ebp
 0x08048b49 <+1>:     mov    %esp,%ebp
 0x08048b4b <+3>:     sub    $0x20,%esp # Ingrease the stack by 32 bytes
 0x08048b4e <+6>:     push   %esi # Push the number in %esi to the stack -> 4 bytes
 0x08048b4f <+7>:     push   %ebx # Push the number in %ebx to the stack -> 4 bytes
 0x08048b50 <+8>:     mov    0x8(%ebp),%edx # Move the number at %ebp + 8 bytes (epb offset 8 bytes lower on the stack) to the edx register
 0x08048b53 <+11>:    add    $0xfffffff8,%esp # increase the stack by 8 bytes
 0x08048b56 <+14>:    lea    -0x18(%ebp),%eax
 0x08048b59 <+17>:    push   %eax
 0x08048b5a <+18>:    push   %edx
 0x08048b5b <+19>:    call   0x8048fd8 <read_six_numbers>
 0x08048b60 <+24>:    add    $0x10,%esp
 0x08048b63 <+27>:    cmpl   $0x1,-0x18(%ebp)
 0x08048b67 <+31>:    je     0x8048b6e <phase_2+38>
 0x08048b69 <+33>:    call   0x80494fc <explode_bomb>
 0x08048b6e <+38>:    mov    $0x1,%ebx
 0x08048b73 <+43>:    lea    -0x18(%ebp),%esi
 0x08048b76 <+46>:    lea    0x1(%ebx),%eax
 0x08048b79 <+49>:    imul   -0x4(%esi,%ebx,4),%eax
 0x08048b7e <+54>:    cmp    %eax,(%esi,%ebx,4)
 0x08048b81 <+57>:    je     0x8048b88 <phase_2+64>
 0x08048b83 <+59>:    call   0x80494fc <explode_bomb>
 0x08048b88 <+64>:    inc    %ebx
 0x08048b89 <+65>:    cmp    $0x5,%ebx
 0x08048b8c <+68>:    jle    0x8048b76 <phase_2+46>
 0x08048b8e <+70>:    lea    -0x28(%ebp),%esp
 0x08048b91 <+73>:    pop    %ebx
 0x08048b92 <+74>:    pop    %esi
 0x08048b93 <+75>:    mov    %ebp,%esp
 0x08048b95 <+77>:    pop    %ebp
 0x08048b96 <+78>:    ret
```

Here we see that we are calling a function `read_six_numbers` and just before that, we are pushing two values to the stack just before calling the `read_six_numbers` function. So, the C source code might look like this:

```c
void* read_six_numbers(void *arg1, void *arg2) {}
```

We are passing two arguments to the `read_six_numbers`, let's analyze them by putting a brek point on the instruction at offset `19`:

```
b *(phase_2+19)
```

And let's run the executable by using the first discovered string "Public speaking is very easy." and the second string will be a random string "my random string" for now

```
(gdb) r
Starting program: /home/parker/Documents/reversing.ctfd.io/phase2/bomb
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/usr/lib/libthread_db.so.1".
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Public speaking is very easy.
Phase 1 defused. How about the next one?
this is my random string!!

Breakpoint 1, 0x08048b5b in phase_2 ()
(gdb) x/s $eax
0xffffd620:     "\200\266\004\b\300\227\004\bH\326\377\377\b\222\004\b`\313\377\367 \332\377\367h\326\377\377\203\212\004\bж\004\b$\265\004\bh\326\377\377z\212\004\b,\036\371\367\3442\327\367@x\356", <incomplete sequence \367>
(gdb) x/s $edx
0x804b6d0 <input_strings+80>:   "this is my random string!!"
```

We can see that the first argument (value pointed by the value of the `%eax` register) for the function `read_six_numbers` is the random strings we write on the terminal and some other pointer.
The register `ebx` doesn't seem to have any meaning so we could assume that it will be used to recover the returns values of the `read_six_numbers`

Ok, so we now that we pass the input to the `read_six_numbers`, now let's check what that function does by disassembling it:

```nasm
0x08048fd8 <+0>:     push   %ebp
0x08048fd9 <+1>:     mov    %esp,%ebp
0x08048fdb <+3>:     sub    $0x8,%esp
0x08048fde <+6>:     mov    0x8(%ebp),%ecx
0x08048fe1 <+9>:     mov    0xc(%ebp),%edx
0x08048fe4 <+12>:    lea    0x14(%edx),%eax
0x08048fe7 <+15>:    push   %eax
0x08048fe8 <+16>:    lea    0x10(%edx),%eax
0x08048feb <+19>:    push   %eax
0x08048fec <+20>:    lea    0xc(%edx),%eax
0x08048fef <+23>:    push   %eax
0x08048ff0 <+24>:    lea    0x8(%edx),%eax
0x08048ff3 <+27>:    push   %eax
0x08048ff4 <+28>:    lea    0x4(%edx),%eax
0x08048ff7 <+31>:    push   %eax
0x08048ff8 <+32>:    push   %edx
0x08048ff9 <+33>:    push   $0x8049b1b
0x08048ffe <+38>:    push   %ecx
0x08048fff <+39>:    call   0x8048860 <sscanf@plt>
0x08049004 <+44>:    add    $0x20,%esp
0x08049007 <+47>:    cmp    $0x5,%eax
0x0804900a <+50>:    jg     0x8049011 <read_six_numbers+57>
0x0804900c <+52>:    call   0x80494fc <explode_bomb>
0x08049011 <+57>:    mov    %ebp,%esp
0x08049013 <+59>:    pop    %ebp
0x08049014 <+60>:    ret
```

By doing a quick look at the function, we see that we are using the `sscanf` function. So this function probably parse our input to some values.
The instructions `6` and `9` recovers the functions arguments from the stack and moves them into the registers `ecx` and `edx` so our first argument is at memory address given by the value of the register `ecx` and the second is given by the value `edx`.
We can check this by putting a breakpoint at instructino `12` on the `read_six_numbers` function and checking the values at the address pointed by `ecx` and `edx`:

```
(gdb) b *(read_six_numbers+12)
Breakpoint 2 at 0x8048fe4
(gdb) c
Continuing.

Breakpoint 2, 0x08048fe4 in read_six_numbers ()
(gdb) x/s $ecx
0x804b6d0 <input_strings+80>:   "this is my random string!!"
(gdb) x/s $edx
0xffffd620:     "\200\266\004\b\300\227\004\bH\326\377\377\b\222\004\b`\313\377\367 \332\377\367h\326\377\377\203\212\004\bж\004\b$\265\004\bh\326\377\377z\212\004\b,\036\371\367\3442\327\367@x\356", <incomplete sequence \367>
(gdb)
```

After this instructions, we seem to push 8 elements to the stack, these are the arguments to our `sscanf` function. Which by looking at the manual can see that takes a `char *input`, a `char *format` and an arbitrary number of argument to which we put the scanned elements.
Let's put a breakpoint in instruction 39 to check what is the format string that we are passing to `sscanf`:

```
(gdb) b *(read_six_numbers+39)
Breakpoint 3 at 0x8048fff
(gdb) c
Continuing.

Breakpoint 3, 0x08048fff in read_six_numbers ()
(gdb) x/s 0x8049b1b
0x8049b1b:      "%d %d %d %d %d %d"
```

We are interested in the second argument of `sscanf` as this is the format string. We can see that the format string is `"%d %d %d %d %d %d"`. So we are parsing 6 numbers separated by space.
After executing the `sscanf` function, we are left with the following instructions

```nasm
0x08048fff <+39>:    call   0x8048860 <sscanf@plt>
0x08049004 <+44>:    add    $0x20,%esp
0x08049007 <+47>:    cmp    $0x5,%eax
0x0804900a <+50>:    jg     0x8049011 <read_six_numbers+57>
0x0804900c <+52>:    call   0x80494fc <explode_bomb>
0x08049011 <+57>:    mov    %ebp,%esp
0x08049013 <+59>:    pop    %ebp
0x08049014 <+60>:    ret
```

We can see that we are checking that the return value (stored at register `eax`) is `5`. So we are checking that the `scanff` function sucesfully scanned `6` elements. If we did scan all of them correctly,
we jump to the instruction `57` hence returning the function `read_six_numbers`. If we couldn't parse `6` numbers, the bomb explotes.

Ok, so we know that the seconds fase expect an input of 6 numbers separated by space `"%d %d %d %d %d %d"` this is parsed by the function `read_six_numbers` which then returns to the function `phase_2` if the parsing was succesfull.

Let's analyze now what happens on `phase_2` after returning from `read_six_numbers`:

```nasm
0x08048b5b <+19>:    call   0x8048fd8 <read_six_numbers>
0x08048b60 <+24>:    add    $0x10,%esp
0x08048b63 <+27>:    cmpl   $0x1,-0x18(%ebp)
0x08048b67 <+31>:    je     0x8048b6e <phase_2+38>
0x08048b69 <+33>:    call   0x80494fc <explode_bomb>
0x08048b6e <+38>:    mov    $0x1,%ebx
0x08048b73 <+43>:    lea    -0x18(%ebp),%esi
0x08048b76 <+46>:    lea    0x1(%ebx),%eax
0x08048b79 <+49>:    imul   -0x4(%esi,%ebx,4),%eax
0x08048b7e <+54>:    cmp    %eax,(%esi,%ebx,4)
0x08048b81 <+57>:    je     0x8048b88 <phase_2+64>
0x08048b83 <+59>:    call   0x80494fc <explode_bomb>
0x08048b88 <+64>:    inc    %ebx
0x08048b89 <+65>:    cmp    $0x5,%ebx
0x08048b8c <+68>:    jle    0x8048b76 <phase_2+46>
0x08048b8e <+70>:    lea    -0x28(%ebp),%esp
0x08048b91 <+73>:    pop    %ebx
0x08048b92 <+74>:    pop    %esi
0x08048b93 <+75>:    mov    %ebp,%esp
0x08048b95 <+77>:    pop    %ebp
0x08048b96 <+78>:    ret
```

We see that we adjust our `%esp` (Pointer to the top of the stack) by adding `0x10` (`16` in decimal) and hence decreacing the stack by 16 bytes. Next we compare the first scanned number that `sscanf` with `0x1` at instruction `27`:

```nasm
0x08048b63 <+27>:    cmpl   $0x1,-0x18(%ebp)
0x08048b67 <+31>:    je     0x8048b6e <phase_2+38>
```

If the first scanned number is equal to 1, we continue. If it's not equal, we explote the bomb. That's it! We got our first element! Our input are 6 numbers separated by space and now we know that the first number must be equal to 1 to prevent the bomb from exploting.

Now let's continue analyzing the next instructinos. After checking that our first number is `1`, we then jump to instruction 38:

```nasm
0x08048b6e <+38>:    mov    $0x1,%ebx
0x08048b73 <+43>:    lea    -0x18(%ebp),%esi
0x08048b76 <+46>:    lea    0x1(%ebx),%eax
0x08048b79 <+49>:    imul   -0x4(%esi,%ebx,4),%eax
0x08048b7e <+54>:    cmp    %eax,(%esi,%ebx,4)
0x08048b81 <+57>:    je     0x8048b88 <phase_2+64>
0x08048b83 <+59>:    call   0x80494fc <explode_bomb>
0x08048b88 <+64>:    inc    %ebx
0x08048b89 <+65>:    cmp    $0x5,%ebx
0x08048b8c <+68>:    jle    0x8048b76 <phase_2+46>
0x08048b8e <+70>:    lea    -0x28(%ebp),%esp
0x08048b91 <+73>:    pop    %ebx
0x08048b92 <+74>:    pop    %esi
0x08048b93 <+75>:    mov    %ebp,%esp
0x08048b95 <+77>:    pop    %ebp
0x08048b96 <+78>:    ret
```

Before anaylzing line by line, let's notice a thing first,on instruction `64`, `65` and `68` we are definining the conditions for a loop. We increase the `%ebx`, we compare it to `%0x5` and if if is
less or equal we continue the loop, if it's greater, we break the loop. To continue the loop we jump to the instruction at `46`. So we have a loop defined between instructions `46` and `68`. Now let's analyze
the 2 instructions before the loop (`38` and `43`):

```nasm
0x08048b6e <+38>:    mov    $0x1,%ebx
0x08048b73 <+43>:    lea    -0x18(%ebp),%esi
```

On instruction `38` we mov the value to `ebx`. Remember that `ebx` is used to break the loop we described before. So we initialize `ebx` and the load the effective address `-0x18(%ebp)` into the `esi` register. This address contains the first
element of our array of scanned number.

So after this 2 instruction we initialize the loop.

Let's analyze the loop, remeber that the initial value of our `ebx` register is `0x1` (`1` in decimal)

```nasm
0x08048b76 <+46>:    lea    0x1(%ebx),%eax
0x08048b79 <+49>:    imul   -0x4(%esi,%ebx,4),%eax
0x08048b7e <+54>:    cmp    %eax,(%esi,%ebx,4)
0x08048b81 <+57>:    je     0x8048b88 <phase_2+64>
0x08048b83 <+59>:    call   0x80494fc <explode_bomb>
0x08048b88 <+64>:    inc    %ebx
0x08048b89 <+65>:    cmp    $0x5,%ebx
0x08048b8c <+68>:    jle    0x8048b76 <phase_2+46>
```

So let's iterative one time and let's consider `arr` is the array of the numbers we passed as input.

- Instruction `46` adds `1` to `ebx` and stores it in `eax`, so in our current iteration `eax` is `2`
- `imul` we multiply the element at `arr[ebx - 1]` with `eax` and store it in `eax`. So in this case `arr[ebx - 1] * eax = arr[1 - 1] * 2 = arr[0]*2= 1*2 = 2`
- We compare the previous multiplication stored at `eax` with the element at index `ebx` (`arr[ebx]`) and if they are equal we continue, so we compare `2` with `arr[1]`. If `arr[1] == 2` we continue, else, we explote the bomb

Great so we have an understanding of how it works. Let's make it in C:

```C
int arr[6] = load_six_numbers();
int counter = 1
while (counter <= 5) {
 int mult = arr[counter - 1]*(counter + 1);
 if (mult == arr[counter]) {
  // continue
 } else {
  explote_bomb();
 }
 counter++;
}
```

We have to create an array that can pass this while loop so our function returns. For each element index `i` at the array, the following condition has to be true

```c
arr[i] == arr[i - 1] * (i + 1)
```

Let's calculate `arr[1]`:

```c
arr[1] = arr[1 - 1] * (1 + 1) = arr[0] * 2 =  1 * 2 = 2
```

We now know that `arr[1] = 2`, let's do it for `arr[2]`:

```c
arr[2] = arr[2 - 1] (2 + 1) = arr[1] * 3 = 2 * 3 = 6
```

We have that `arr[2] = 6`. Now for the rest of the elements:

```c
arr[3] = arr[2] * (4) = 6 * 4 = 24
arr[4] = arr[3] * (5) = 24 * 5 = 120
arr [5] = arr[4] * 6 = 120 * 6 = 720
```

So the solutions gives us: `1 2 6 24 120 720`
