# OBJECTIVE 6 - Shellcode Primer #

## OBJECTIVE : ##
>Complete the [Shellcode Primer](https://tracer.kringlecastle.com/) in Jack’s Office.
>According to the last challenge, what is the secret to KringleCon success? **“All of our speakers and organizers, providing the gift of _____, free to the community.”**  Talk to Chimney Scissorsticks in the NetWars are for hints.

#  

## HINTS: ##
<details>
  <summary>Hints provided for Objective 6</summary>
  
>-	If you run into any shellcode primers at the North Pole, be sure to read the directions and the comments in the shellcode source!
>-	Also, troubleshooting shellcode can be difficult. Use the debugger step-by-step feature to watch values.
>-	Lastly, be careful not to overwrite any register values you need to reference later on in your shellcode.
</details>

#  

## PROCEDURE : ##

### 3. Getting Started ###
>Welcome! Are you ready to learn how to write shellcode? We hope so! First, some tips:
>
>Comments are denoted with a semicolon (;)
>Don't forget to look at the debugger, line by line, if something is wrong
>Really, don't forget to read the error list! We check each place where you might go wrong in your code
>Your code for each level is saved in your browser, so you can leave and come back, refresh the page, and hop back to previous levels to borrow code
>This level currently fails to build because it has no code. Can you add a **ret**urn statement at the end? Don't worry about what it's actually returning (yet!)
>
>Feel free to check previous levels!
```
ret
```

### 4. Returning a Value ###
>Now that we have an empty function, we can start building some code! Let's learn what a *register* is.
>
>A register is like a variable, except there are a small number of them - you have about eight general purpose 64-bit integers registers on amd64 (we won't talk about floating point or other special registers):
>
>-  rax
>-  rbx
>-  rcx
>-  rdx
>-  rdi
>-  rsi
>-  rbp
>-  rsp
>
>All mathy stuff that a computer does (add, subtract, xor, etc) operates on registers, not directly on memory. So they're super important!
>
>Specific registers have some implicit meaning, mostly by convention. For example, when a function returns, its return value is typically put in rax. Let's do that!
>
>To move a value into a register, use the `mov` instruction; for example:
>
>`mov rdx, 1`
>
>In a higher-level language this would be equivalent to:
>
>`rdx = 1 —`
>
>For this level, can you return the number '1337' from your function?
>
>That means that `rax` must equal `1337` when the function returns.
```
mov rax, 1337
ret
```

### 5. System Calls ###
>If you've made it this far, I bet you're wondering how to make your shellcode *do* something!
>
>If you're familiar with Python, you might know how to use the `open()` function. If you know C, you might know the `fopen()` function. But what these and similar functions have in common is one thing: they're library code. And because shellcode needs to be self contained, we don't have (easy) access to library code!
>
>So how do we deal with that?
>
>Linux has something called a `syscall`, or system call. A `syscall` is a request that a program makes that asks Linux - the kernel - to do something. And it turns out, at the end of the day, all of those library calls ultimately end with a syscall. [Here is a list of available syscalls on x64](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/) ([alternative](https://chromium.googlesource.com/chromiumos/docs/+/master/constants/syscalls.md))
>
>To perform a syscall:
>
>-  The number for the desired syscall is moved into rax
>-  The first parameter is moved into rdi, the second into rsi, and the third into rdx (there are others, but not many syscalls need more than 3 parameters)
>-  Execute the `syscall` instruction
>
>The second `syscall` executes, Linux flips into kernel mode and we can no longer debug it. When it's finished, it returns the result in `rax`.
>#  
>For this challenge, we're going to call `sys_exit` to exit the process with exit code 99.
>
>Can you prepare `rax` and `rdi` with the correct values to exit?
>
>As always, feel free to mess around as much as you like!
```
mov rax, 60
mov rdi, 99
syscall
```

### 6. Calling Into the Void ###
>Before we learn how to use the *Really Good* syscalls, let's try something fun: crash our shellcode on purpose!
>
>You might think I'm mad, but there's a method to my madness. Run the code below and watch what happens! No need to modify it, unless you want to. :)
>
>Be sure to look at the debugger to see what's going on! Especially notice the top of the stack at the `ret` instruction.
```
push 0x12345678
ret
```

### 7. Getting RIP ###
>What happened in the last exercise? Why did it crash at `0x12345678`? And did you notice that `0x12345678` was on top of the stack when `ret` happened?
>
>The short story is this: `call` pushes the return address onto the stack, and `ret` jumps to it. Whaaaat??
>
>This is going to be long, but hopefully will make it all clear!
>#  
>Let's back up a bit. At any given point, the instruction currently being executed is stored in a special register called the instruction pointer (`rip`), which you may also hear called a program counter (`pc`).
>
>What is the `rip` value at the first line in our code? Well, since we have a debugger, we know that it's `0x13370000`. But sometimes you don't know and need to find out.
>
>The most obvious answer is to treat it like a normal register, like this:
>
>`mov rax, rip`
>`ret`
>
>Does that work? Nope! You can't directly access `rip`. That means we need a trick!
>#  
>When you use `call` in x64, the CPU doesn't care where it's calling, or whether there's a `ret` waiting for it. The CPU assumes that, if the author put a `call` in, there will naturally be a `ret` on the other end. Doing anything else would just be silly! So call pushes the return address onto the stack before jumping into a function. When the function complete, the `ret` instruction uses the return address on the stack to know where to return to.
>
>The CPU assumes that, sometime later, a `ret` will execute. The ret assumes that at some point earlier a `call` happened, and that means that the top of the stack has the return address. The `ret` will retrieve the return address off the top of the stack (using `pop`) and jump to it.
>
>Of course, we can execute `pop` too! If we `pop` the return address off the stack, instead of jumping to it, the address goes into a register. Hmm! Does that also sound like `mov REG, rip` to you?
>#  
>For this exercise, can you `pop` the address after the call - the *No Op* (`nop`) instruction - into `rax` then return?

```
call place_below_the_nop
nop
place_below_the_nop:
pop rax
ret
```

### 8. Hello World! ###
>So remember how last level, we got the address of `nop` and returned it?
>
>Did you see that `nop` execute? Nope! We jumped right over it, but stored its address en-route. What can we do by knowing our own address?
>
>Well, since shellcode is, by definition, self-contained, you can do other fun stuff like include data alongside the code!
>
>What if the return address isn't an instruction at all, but a string?
>#  
>For this next exercise, we include a plaintext string - `'Hello World'` - as part of the code. It's just sitting there in memory. If you look at the compiled code, it's all basically `Hello World`, which doesn't run.
>
>Instead of trying to run it, can you `call` past it, and `pop` its address into `rax`?
>
>Don't forget to check the debugger after to see it in `rax`!
```
call something
db 'Hello World',0
something:
pop rax
ret
```

### 9. Hello World!! ###
>Remember syscalls? Earlier, we used them to call an exit. Now let's try another!
>
>This time, instead of getting a pointer to the string `Hello World`, we're going to print it to standard output (stdout).
>
>Have another look [at the syscall table](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/). Can you find `sys_write`, and use to to print the string `Hello World`! to stdout?
>
>Note: stdout's file descriptor is `1`.
```
call something
db 'Hello World!',0
something:
pop rsi
mov rax, 1
mov rdi,1
mov rdx, 12
syscall
ret
```

### 10. Opening a File ###
>We're getting dangerously close to do something interesting! How about that?
>
>Can you use the `sys_open` syscall to open `/etc/passwd`, then return the file handle (in `rax`)?
>
>Have another look [at the syscall table](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/). Can you call `sys_open` on the file `/etc/passwd`, then return the file handle? Here's the [syscall table](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/) again.
```
call placeholder
db '/etc/passwd',0
placeholder:
pop rdi

mov rax, 2
mov rsi, 0
mov rdx, 0
syscall
ret
```

### 11. Reading a File ###
>Do you feel ready to write some useful code? We hope so! You're mostly on your own this time! Don't forget that you can reference your solutions from other levels!
>
>For this exercise, we're going to read a specific file… let's say, `/var/northpolesecrets.txt`… and write it to stdout. No reason for the name, but since this is Jack Frost's troll-trainer, it might be related to a top-secret mission!
>
>Solving this is going to require three syscalls! Four if you decide to use `sys_exit` - you're welcome to return or exit, just don't forget to fix the stack if you return!
>
>First up, just like last exercise, call `sys_open`. This time, be sure to open `/var/northpolesecrets.txt`.
>
>Second, find the `sys_read` entry on [the syscall table](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/), and set up the call. Some tips:
>1.  The file descriptor is returned by sys_open
>2.  The buffer for reading the file can be any writeable memory - rsp is a great option, temporary storage is what the stack is meant for
>3.  You can experiment to find the right count, but if it's a bit too high, that's perfectly fine
>
>Third, find the `sys_write` entry, and use it to write to stdout. Some tips on that:
>1.  The file descriptor for stdout is always 1
>2.  The best value for `count` is the return value from `sys_read`, but you can experiment with that as well (if it's too long, you might get some garbage after; that's okay!)
>
>Finally, if you use `rsp` as a buffer, you won't be able to `ret` - you're going to overwrite the return address and `ret` will crash. That's okay! You remember how to `sys_exit`, right? :)
>
>(For an extra challenge, you can also subtract from `rsp`, use it, then add to `rsp` to protect the return address. That's how typical applications do it.)
>
>Good luck!
>

```
call placeholder
db '/var/northpolesecrets.txt',0
placeholder:
pop rdi

mov rax, 2
mov rsi, 0
mov rdx, 0
syscall

push rax
sub rsp, 16
mov rax, 0
mov rdi, [rsp+16]
mov rsi, rsp
mov rdx, 138
syscall

mov rax, 1
mov rdi, 1
mov rdx, 138
syscall;

mov rax, 60
mov rdi, 0
syscall
``` 

![image](https://github.com/beta-j/SANS-Holiday-Hack-Challenge-2021/assets/60655500/22ac0b30-e8df-4469-ae39-470b6a12ff87)


The **'Success!** notice at the end of the challenge lets us know that we can add `?cheat` after the URL (before the ``#``) to unlock the recommended solutions

