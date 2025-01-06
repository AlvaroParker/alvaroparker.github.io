+++
title="CMU Bomb Lab Phase 1"
date="2024-09-01T17:07:25.000Z"

[taxonomies]
tags = ["CMU Bomb", "Reverse Engineering", "Code"]
+++

# CMU Bomb Lab reverse engineering

This repo contains the explanation on how I solved the CMU bomb lab on reverse engineering. I mostly used `gdb` to analye and reverse engineer the code and I documented my thought process and explanations.

"The CMU bomb lab is a famous reverse engineering challenge from CMU.

This category's challenges will track your progress through the 32-bit x86 CMU bomb lab.

Welcome to my fiendish little bomb. You have 6 phases with which to blow yourself up. Have a nice day!"

If you wanna solve the challenge go to [reversing.ctfd.io](https://reversing.ctfd.io/challenges)

## Phase 1

_Note: We are using AT&T assembly syntax_

We have to defuse the phase 1 of the bomb

Using gdb

```gdb
disassemble main
```

We have the following output:

```nasm
0x08048a52 <+162>:   call   0x80491fc <read_line>
0x08048a57 <+167>:   add    $0xfffffff4,%esp
0x08048a5a <+170>:   push   %eax
0x08048a5b <+171>:   call   0x8048b20 <phase_1>
0x08048a60 <+176>:   call   0x804952c <phase_defused>
```

On instruction `162` we see a call to read line corresponding to the first input the program expects when executing `./bomb`. After this, a function `phase_1` is called.

disassembling the function `phase_1` on `gdb` gives us the following:

```nasm
0x08048b20 <+0>:     push   %ebp
0x08048b21 <+1>:     mov    %esp,%ebp
0x08048b23 <+3>:     sub    $0x8,%esp
0x08048b26 <+6>:     mov    0x8(%ebp),%eax
0x08048b29 <+9>:     add    $0xfffffff8,%esp
0x08048b2c <+12>:    push   $0x80497c0
0x08048b31 <+17>:    push   %eax
0x08048b32 <+18>:    call   0x8049030 <strings_not_equal>
0x08048b37 <+23>:    add    $0x10,%esp
0x08048b3a <+26>:    test   %eax,%eax
0x08048b3c <+28>:    je     0x8048b43 <phase_1+35>
0x08048b3e <+30>:    call   0x80494fc <explode_bomb>
0x08048b43 <+35>:    mov    %ebp,%esp
0x08048b45 <+37>:    pop    %ebp
0x08048b46 <+38>:    ret
```

We have the prologue on the first two instructions, then a subtraction to the `esp` pointer indicating that we are allocating space on the stack.

Let's put a breakpoint after the prologue and stack allocation:

```
b *(phase_1+12)
```

Now let's run our program!

```
run
```

We see that we our program started and now is waiting for standard input on the console:

```
Starting program: /home/user/Documents/reversing.ctfd.io/phase1/bomb
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/usr/lib/libthread_db.so.1".
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
>
```

So let's put a random string on the prompt!

```
> Hello world, this is a string!!!!
```

We now have reached our breakpoint that we set earlier:

```
Starting program: /home/user/Documents/reversing.ctfd.io/phase1/bomb
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/usr/lib/libthread_db.so.1".
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Hello world, this is a string!!!

Breakpoint 1, 0x08048b2c in phase_1 ()
```

You can see where in the program you are by doing

```
lay next
```

We see the following:

```nasm
B+>0x8048b2c <phase_1+12>  push   $0x80497c0
0x8048b31 <phase_1+17>  push   %eax
0x8048b32 <phase_1+18>  call   0x8049030 <strings_not_equal>
0x8048b37 <phase_1+23>  add    $0x10,%esp
0x8048b3a <phase_1+26>  test   %eax,%eax
0x8048b3c <phase_1+28>  je     0x8048b43 <phase_1+35>
0x8048b3e <phase_1+30>  call   0x80494fc <explode_bomb>
0x8048b43 <phase_1+35>  mov    %ebp,%esp
0x8048b45 <phase_1+37>  pop    %ebp
0x8048b46 <phase_1+38>  ret
```

We are at the instruction `12` on the function `phase_1`. We see that there's an incoming function call `strings_not_equal` so let's set a breakpoint there to analize the arguments that this function is called with:

```
b *(phase_1+18)
```

And let's continue:

```
c
```

We are now on the instruction `18` on the function `phase_1`:

```nasm
B+ 0x8048b2c <phase_1+12>  push   $0x80497c0
0x8048b31 <phase_1+17>  push   %eax
B+>0x8048b32 <phase_1+18>  call   0x8049030 <strings_not_equal>
0x8048b37 <phase_1+23>  add    $0x10,%esp
0x8048b3a <phase_1+26>  test   %eax,%eax
0x8048b3c <phase_1+28>  je     0x8048b43 <phase_1+35>
0x8048b3e <phase_1+30>  call   0x80494fc <explode_bomb>
0x8048b43 <phase_1+35>  mov    %ebp,%esp
0x8048b45 <phase_1+37>  pop    %ebp
0x8048b46 <phase_1+38>  ret
```

So let's check the arguments! We see that the arguments are passed using the stack (there are two `push`) instructions before our function is called:

```nasm
B+ 0x8048b2c <phase_1+12>  push   $0x80497c0
0x8048b31 <phase_1+17>  push   %eax
```

Let's check those values (`%eax` and `0x80497c0`) using gdb:

```
x/s $eax
> 0x804b680 <input_strings>:      "hello"

x/s 0x80497c0
> 0x80497c0:      "Public speaking is very easy."
```

Huh, we can see that the two arguments passed to the `strings_not_equal` function are the string we input in the beggining and another string. These must be the first phase flag!!!

Solution: `"Public speaking is very easy"`
