## bomblab

> **DO NOT READ THIS ARTICLE. I wrote bullshit in English.**

This lab I have finished once, so this article is just for fun.

## Phase-1

```asm
0000000000400ee0 <phase_1>:
  400ee0:	48 83 ec 08          	sub    $0x8,%rsp
  400ee4:	be 00 24 40 00       	mov    $0x402400,%esi
  400ee9:	e8 4a 04 00 00       	callq  401338 <strings_not_equal>
  400eee:	85 c0                	test   %eax,%eax
  400ef0:	74 05                	je     400ef7 <phase_1+0x17>
  400ef2:	e8 43 05 00 00       	callq  40143a <explode_bomb>
  400ef7:	48 83 c4 08          	add    $0x8,%rsp
  400efb:	c3                   	retq   
```

Just scan the memory of address `[0x402400]` . GDB command is `x /s 0x402400` .

```bash
(gdb) x /s 0x402400
0x402400:	"Border relations with Canada have never been better."
```

We can see that the answer is `Border relations with Canada have never been better.` .

## Phase-2

```asm
0000000000400efc <phase_2>:
  400efc:	55                   	push   %rbp
  400efd:	53                   	push   %rbx
  400efe:	48 83 ec 28          	sub    $0x28,%rsp
  400f02:	48 89 e6             	mov    %rsp,%rsi
  400f05:	e8 52 05 00 00       	callq  40145c <read_six_numbers>
  400f0a:	83 3c 24 01          	cmpl   $0x1,(%rsp)
  400f0e:	74 20                	je     400f30 <phase_2+0x34>
  400f10:	e8 25 05 00 00       	callq  40143a <explode_bomb>
  400f15:	eb 19                	jmp    400f30 <phase_2+0x34>
  400f17:	8b 43 fc             	mov    -0x4(%rbx),%eax
  400f1a:	01 c0                	add    %eax,%eax
  400f1c:	39 03                	cmp    %eax,(%rbx)
  400f1e:	74 05                	je     400f25 <phase_2+0x29>
  400f20:	e8 15 05 00 00       	callq  40143a <explode_bomb>
  400f25:	48 83 c3 04          	add    $0x4,%rbx
  400f29:	48 39 eb             	cmp    %rbp,%rbx
  400f2c:	75 e9                	jne    400f17 <phase_2+0x1b>
  400f2e:	eb 0c                	jmp    400f3c <phase_2+0x40>
  400f30:	48 8d 5c 24 04       	lea    0x4(%rsp),%rbx
  400f35:	48 8d 6c 24 18       	lea    0x18(%rsp),%rbp
  400f3a:	eb db                	jmp    400f17 <phase_2+0x1b>
  400f3c:	48 83 c4 28          	add    $0x28,%rsp
  400f40:	5b                   	pop    %rbx
  400f41:	5d                   	pop    %rbp
  400f42:	c3                   	retq   
```

From this code `read_six_number` , we can see that we should input six numbers. And the we can know that there is a loop, and the most important line of code is :

```asm
400f1a:	01 c0                	add    %eax,%eax
```

Now, we can guess that it should be a geometric series, whose ratio is 2. What is the first element of the number series? We can see:

```asm
400f0a:	83 3c 24 01          	cmpl   $0x1,(%rsp)
400f0e:	74 20                	je     400f30 <phase_2+0x34>
400f10:	e8 25 05 00 00       	callq  40143a <explode_bomb>
```

OK, we can see that if `(%rsp)` is not equal to `0x1` , it will call function `explode_bomb` .

Thus, the answer is : `1 2 4 8 16 32` .

## Phase-3

```asm
0000000000400f43 <phase_3>:
  400f43:	48 83 ec 18          	sub    $0x18,%rsp
  400f47:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  400f4c:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  400f51:	be cf 25 40 00       	mov    $0x4025cf,%esi
  400f56:	b8 00 00 00 00       	mov    $0x0,%eax
  400f5b:	e8 90 fc ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  400f60:	83 f8 01             	cmp    $0x1,%eax
  400f63:	7f 05                	jg     400f6a <phase_3+0x27>
  400f65:	e8 d0 04 00 00       	callq  40143a <explode_bomb>
  400f6a:	83 7c 24 08 07       	cmpl   $0x7,0x8(%rsp)
  400f6f:	77 3c                	ja     400fad <phase_3+0x6a>
  400f71:	8b 44 24 08          	mov    0x8(%rsp),%eax
  400f75:	ff 24 c5 70 24 40 00 	jmpq   *0x402470(,%rax,8)
  400f7c:	b8 cf 00 00 00       	mov    $0xcf,%eax
  400f81:	eb 3b                	jmp    400fbe <phase_3+0x7b>
  400f83:	b8 c3 02 00 00       	mov    $0x2c3,%eax
  400f88:	eb 34                	jmp    400fbe <phase_3+0x7b>
  400f8a:	b8 00 01 00 00       	mov    $0x100,%eax
  400f8f:	eb 2d                	jmp    400fbe <phase_3+0x7b>
  400f91:	b8 85 01 00 00       	mov    $0x185,%eax
  400f96:	eb 26                	jmp    400fbe <phase_3+0x7b>
  400f98:	b8 ce 00 00 00       	mov    $0xce,%eax
  400f9d:	eb 1f                	jmp    400fbe <phase_3+0x7b>
  400f9f:	b8 aa 02 00 00       	mov    $0x2aa,%eax
  400fa4:	eb 18                	jmp    400fbe <phase_3+0x7b>
  400fa6:	b8 47 01 00 00       	mov    $0x147,%eax
  400fab:	eb 11                	jmp    400fbe <phase_3+0x7b>
  400fad:	e8 88 04 00 00       	callq  40143a <explode_bomb>
  400fb2:	b8 00 00 00 00       	mov    $0x0,%eax
  400fb7:	eb 05                	jmp    400fbe <phase_3+0x7b>
  400fb9:	b8 37 01 00 00       	mov    $0x137,%eax
  400fbe:	3b 44 24 0c          	cmp    0xc(%rsp),%eax
  400fc2:	74 05                	je     400fc9 <phase_3+0x86>
  400fc4:	e8 71 04 00 00       	callq  40143a <explode_bomb>
  400fc9:	48 83 c4 18          	add    $0x18,%rsp
  400fcd:	c3                   	retq   
```

Look at this line of code:

```asm
400f51:	be cf 25 40 00       	mov    $0x4025cf,%esi
```

Run `x /s 0x4025cf` in GDB, it will print: `"%d %d"` . Thus, we should input 2 `int` in this phase.

And we can see that there are many annoying `jmp 0x400fb9` instructions in this assembly code, so we can infer that there is a `switch-case` structure. 

The first 2 lines of code below shows that the first number we input (stored in `[%rsp + 8]`) is less equal than 7, otherwise it will call `explode_bomb` .  The last 2 lines tell us that the program will jump to an address according to our first number.

```asm
400f6a:	83 7c 24 08 07       	cmpl   $0x7,0x8(%rsp)
400f6f:	77 3c                	ja     400fad <phase_3+0x6a>
400f71:	8b 44 24 08          	mov    0x8(%rsp),%eax
400f75:	ff 24 c5 70 24 40 00 	jmpq   *0x402470(,%rax,8)
```

We know that the first number (now it has been copied to `%rax`) can be one of number in `[0,1,2,3,4,5,6,7]`. So scan the address `[0x402470 + %rax * 8]` :

```bash
(gdb) x /x 0x402470 + 8 * 0  =>  0x402470:	0x00400f7c  => 0 0xcf
(gdb) x /x 0x402470 + 8 * 1  =>  0x402478:	0x00400fb9  => 1 0x137
(gdb) x /x 0x402470 + 8 * 2  =>  0x402480:	0x00400f83  => 2 0x2c3
(gdb) x /x 0x402470 + 8 * 3  =>  0x402488:	0x00400f8a  => 3 0x100
(gdb) x /x 0x402470 + 8 * 4  =>  0x402490:	0x00400f91  => 4 0x185
(gdb) x /x 0x402470 + 8 * 5  =>  0x402498:	0x00400f98  => 5 0xce
(gdb) x /x 0x402470 + 8 * 6  =>  0x4024a0:	0x00400f9f  => 6 0x2aa
(gdb) x /x 0x402470 + 8 * 7  =>  0x4024a8:	0x00400fa6  => 7 0x147
```

When the first number we input is `0`, the program will jump to `0x400f7c`. And the code at `0x400f7c` is :

```asm
400f7c:	b8 cf 00 00 00       	mov    $0xcf,%eax
```

So the second number is `0xcf = 207`. Total answer is `0 207`.

## Phase-4

```asm
; first call is func4(edx=a, esi=b, edi=x)
0000000000400fce <func4>:
  400fce:	48 83 ec 08          	sub    $0x8,%rsp
  400fd2:	89 d0                	mov    %edx,%eax              ; eax = a
  400fd4:	29 f0                	sub    %esi,%eax              ; eax = eax - esi = a - b = t1
  400fd6:	89 c1                	mov    %eax,%ecx              ; ecx = eax = a - b
  400fd8:	c1 e9 1f             	shr    $0x1f,%ecx             ; ecx = ecx >> 31 = t1 >> 31 = t2
  400fdb:	01 c8                	add    %ecx,%eax              ; eax = eax + ecx = t1 + t2
  400fdd:	d1 f8                	sar    %eax                   ; eax = eax >> 1 = (t1+t2)>>1, t1=(t1+t2)>>1
  400fdf:	8d 0c 30             	lea    (%rax,%rsi,1),%ecx     ; ecx = eax + esi = t1 + b, t2=t1+b
  400fe2:	39 f9                	cmp    %edi,%ecx              ; ecx <= x ? () : (2 * func4())
  400fe4:	7e 0c                	jle    400ff2 <func4+0x24>    ;
  400fe6:	8d 51 ff             	lea    -0x1(%rcx),%edx        ; edx = ecx - 1 = t2 - 1
  400fe9:	e8 e0 ff ff ff       	callq  400fce <func4>         ; func4(edx=t2-1, b, x)
  400fee:	01 c0                	add    %eax,%eax              ; return 2 * func4()
  400ff0:	eb 15                	jmp    401007 <func4+0x39>
  400ff2:	b8 00 00 00 00       	mov    $0x0,%eax              ; eax = 0
  400ff7:	39 f9                	cmp    %edi,%ecx              ; ecx >= edi ? (return 0) : ()
  400ff9:	7d 0c                	jge    401007 <func4+0x39>
  400ffb:	8d 71 01             	lea    0x1(%rcx),%esi         ; esi = ecx + 1
  400ffe:	e8 cb ff ff ff       	callq  400fce <func4>         ; func4()
  401003:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax  ; return 2 * func4() + 1
  401007:	48 83 c4 08          	add    $0x8,%rsp
  40100b:	c3                   	retq   

000000000040100c <phase_4>:
  40100c:	48 83 ec 18          	sub    $0x18,%rsp
  401010:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  401015:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  40101a:	be cf 25 40 00       	mov    $0x4025cf,%esi
  ; scan the address 0x4025cf
  40101f:	b8 00 00 00 00       	mov    $0x0,%eax
  401024:	e8 c7 fb ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  401029:	83 f8 02             	cmp    $0x2,%eax
  40102c:	75 07                	jne    401035 <phase_4+0x29>
  40102e:	83 7c 24 08 0e       	cmpl   $0xe,0x8(%rsp)
  ; the 1st number x should be x <= 15, otherwise explode
  401033:	76 05                	jbe    40103a <phase_4+0x2e>
  401035:	e8 00 04 00 00       	callq  40143a <explode_bomb>
  40103a:	ba 0e 00 00 00       	mov    $0xe,%edx
  40103f:	be 00 00 00 00       	mov    $0x0,%esi
  401044:	8b 7c 24 08          	mov    0x8(%rsp),%edi
  401048:	e8 81 ff ff ff       	callq  400fce <func4>
  ; func4(edx=0xe, esi=0x0, edi=x)
  40104d:	85 c0                	test   %eax,%eax
  ; the returned val of func4 shouble 0
  40104f:	75 07                	jne    401058 <phase_4+0x4c>
  401051:	83 7c 24 0c 00       	cmpl   $0x0,0xc(%rsp)
  ; the 2nd number y should be 0, otherwise explode
  ; so the total condition is func(0xe,0x0,x) == 0 and y = 0
  401056:	74 05                	je     40105d <phase_4+0x51>
  401058:	e8 dd 03 00 00       	callq  40143a <explode_bomb>
  40105d:	48 83 c4 18          	add    $0x18,%rsp
  401061:	c3                   	retq   
```

`phase_4` call a function `func4`, which is a recursion. Scan the address `0x4025cf`, GDB will print `"%d %d"`, which means we should input 2 `int` numbers. Assume that our input is `x y`. The `x` will be stored in `%rsp + 0x8` and y in `%rsp + 0xc`. Then `phase_4` would call `func4(edx=0xe, esi=0x0, edi=x)`, and this function call passes parameters by registers. Read the comment above.  

The conditions for passing `phase_4` is input `x y` to let `func(0xe, 0x0, x)` return 0 and  `x` should be less equal than 15 and `y` must be 0.

Read the code of `func4`, we can change it into C code:

```asm
int func4(int a, int b, int x)
{
    x;                                 // edi
    int t1 = a - b;                    // eax
    int t2 = ((unsigned int)t1) >> 31; // ecx
    t1 = (t1 + t2) >> 1;
    t2 = t1 + b;
    if (t2 <= x)
    {
        t1 = 0;
        if (t2 < x)
            return 2 * func4(a, t2 + 1, x) + 1;
        return t1;
    }
    else
        return 2 * func4(t2 - 1, b, x);
}
```

What we need to pay attention to is this instruction: `400fd8: shr   $0x1f,%ecx`. The `shr` implies that it will operation on a `unsigned` type (read the C code above) .

Therefore, we can find `x` by enumeration. 

```c
int main()
{
    int x = 0;
    for (x = 0; x <= 15; x++)
    {
        int t = func4(0xe, 0x0, x);
        if (t == 0)
            printf("%d ", x);
    }
}
```

The result is:

```cpp
(x,y) = (0,0), (1,0), (3,0), (7,0)
```



## Phase-5

```assembly
0000000000401062 <phase_5>:
  401062:	53                   	push   %rbx
  401063:	48 83 ec 20          	sub    $0x20,%rsp
  ; 32 bytes stack for temporary variable
  401067:	48 89 fb             	mov    %rdi,%rbx
  ; rdi is the address of strings we input
  40106a:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
  401071:	00 00 
  ; I don't understand here
  401073:	48 89 44 24 18       	mov    %rax,0x18(%rsp)
  401078:	31 c0                	xor    %eax,%eax
  40107a:	e8 9c 02 00 00       	callq  40131b <string_length>
  40107f:	83 f8 06             	cmp    $0x6,%eax
  ; we should input a string, whose len is 6
  ; do while loop begin, eax=0 is the counter
  401082:	74 4e                	je     4010d2 <phase_5+0x70>
  401084:	e8 b1 03 00 00       	callq  40143a <explode_bomb>
  401089:	eb 47                	jmp    4010d2 <phase_5+0x70>
  40108b:	0f b6 0c 03          	movzbl (%rbx,%rax,1),%ecx
  ; t = str[i]
  40108f:	88 0c 24             	mov    %cl,(%rsp)
  401092:	48 8b 14 24          	mov    (%rsp),%rdx
  401096:	83 e2 0f             	and    $0xf,%edx
  ; t = str[i] & 0xf
  401099:	0f b6 92 b0 24 40 00 	movzbl 0x4024b0(%rdx),%edx
  4010a0:	88 54 04 10          	mov    %dl,0x10(%rsp,%rax,1)
  ; array[rsp+0x10+rax] = str2[t]
  4010a4:	48 83 c0 01          	add    $0x1,%rax
  4010a8:	48 83 f8 06          	cmp    $0x6,%rax
  ; eax++ until out of str.length
  4010ac:	75 dd                	jne    40108b <phase_5+0x29>
  4010ae:	c6 44 24 16 00       	movb   $0x0,0x16(%rsp)
  4010b3:	be 5e 24 40 00       	mov    $0x40245e,%esi
  ; "flyers"
  4010b8:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
  4010bd:	e8 76 02 00 00       	callq  401338 <strings_not_equal>
  4010c2:	85 c0                	test   %eax,%eax
  4010c4:	74 13                	je     4010d9 <phase_5+0x77>
  4010c6:	e8 6f 03 00 00       	callq  40143a <explode_bomb>
  4010cb:	0f 1f 44 00 00       	nopl   0x0(%rax,%rax,1)
  4010d0:	eb 07                	jmp    4010d9 <phase_5+0x77>
  4010d2:	b8 00 00 00 00       	mov    $0x0,%eax
  4010d7:	eb b2                	jmp    40108b <phase_5+0x29>
  4010d9:	48 8b 44 24 18       	mov    0x18(%rsp),%rax
  4010de:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
  4010e5:	00 00 
  4010e7:	74 05                	je     4010ee <phase_5+0x8c>
  4010e9:	e8 42 fa ff ff       	callq  400b30 <__stack_chk_fail@plt>
  4010ee:	48 83 c4 20          	add    $0x20,%rsp
  4010f2:	5b                   	pop    %rbx
  4010f3:	c3                   	retq   
```

This `phase_5` want us to input a string `str` (its length is 6), which is a simple phase.

There is a hash table in the program: 

```
hex index: 0   1   2   3   4   5   6   7   8   9   A   B   C   D   E   F
hash str : m   a   d   u   i   e   r   s   n   f   o   t   v   b   y   l
```

`phase_5` construct a hash function is: `f(x) = hash_str[x & 0xf]`.

We aim to let `f(str) = "flyers"`.

`flyers` is equal to character `9, F, E, 4, 5, 6` in the hash table, thus, `str[i] & 0xf (i=0,...,5)`  should be `9, F, E, 4, 5, 6`. We can check the ASCII table to get characters: 

```bash
0x39  0x3F  0x3E  0x34  0x35  0x36
'9'   '?'   '>'   '4'   '5'   '6'
```

The string `"9?>456"` is the key to pass this phase.

If you still do not understand, see the C code:

```c
void phase5(const char *input)
{
    const char *hash_str = "maduiersnfotvbyl";
    char array[7];
    int i = 0;
    for (i = 0; i < 6; i++)
        array[i] = hash_str[input[i] & 0xf];
    if (sting_not_equal(array, "flyers"))
        explode();
}
```



## Evaluation

All the answers are in a text files `sin.ans`:

```
unix > cat sin.ans
Border relations with Canada have never been better.
1 2 4 8 16 32
0 207
7 0 DrEvil
9?>456
4 3 2 1 6 5
20
```

Execute the `bomb` binary file:

```
unix > ./bomb  sin.ans
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Phase 1 defused. How about the next one?
That's number 2.  Keep going!
Halfway there!
So you got that one.  Try this one.
Good work!  On to the next...
Curses, you've found the secret phase!
But finding it and solving it are quite different...
Wow! You've defused the secret stage!
Congratulations! You've defused the bomb!
```



## Summary 

好无聊，希望早点开学🤢，👴要🤮了。

