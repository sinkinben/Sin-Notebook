## CSAPP-datalab

## Integer

### bitXor

```c
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y)
{
  return ~(~(~x & y) & ~(~y & x));
}
```

解题思路：布尔代数。对于异或，从定义式出发，并使用「德摩根定律」变换：
$$
\begin{align}
A \oplus B &= \overline{A}B + \overline{B}A \\
&=\overline{(\overline{\overline{A}B}) \cdot (\overline{\overline{A}B})}
\end{align}
$$

### tmin

```c
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void)
{
  return 1 << 31;
}
```

解题思路：Nothing special .

### isTmax

```c
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x)
{
  return (!!(x + 1)) & (!(((x + 1) ^ x) + 1));
}
```

解题思路：`isTmax(x) = 1 iff x == 0x7fffffff` 。那么 `x+1 = 0x80000000` ，显然 `x ^ (x+1) == (int)(-1)` 。ti需要考虑的特殊情况是 `x = -1` ，因此加入 `!!(x + 1)` 的情况。



### allOddBits

```c
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x)
{
  int mask = (0xAA << 8) | 0xAA;
  mask = (mask << 16) | mask;
  return !((x & mask) ^ mask);
}
```

解题思路：偶数位上的 `1` 对本题毫无用处，因此构造掩码 `mask = 0xAAAAAAAA` 将偶数位全部清零，也就是 `y = x & mask` 。题目要求奇数位上必须全为 `1` ，就说明 `y == 0xAAAAAAAA == mask` ，因此 `(y ^ mask) == 0x0` 。



### negate

```c
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x)
{
  return (~x) + 1;
}
```

解题思路：补码的性质，相反数是「按位取反再加一」。

### isAsciiDigit

```c
/* 
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x)
{
  return ((((x + (~0x30) + 1) >> 31) & 1) ^ 1) & ((((0x39 + (~x) + 1) >> 31) & 1) ^ 1);
}
```

解题思路：思路很明显，就是需要判断 `0x30 <= x && x <= 0x30` 。下面来看如何判断 `a <= b` 。

```
    a <= b
->  b - a >= 0
->  b + (~a) + 1 >= 0
->  sign(b + ~a + 1) == 0
->  sign(b + ~a + 1) ^ 1 == 1
```

`sign(x) = (x >> 31) & 1` 表示 `x` 的符号位。

### conditional

````c
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z)
{
  x = ~(!!x) + 1;
  return (y & x) + (z & (~x));
}
````

解题思路：原本的想法是构造 `y*x + z*(!x)` 这种形式作为返回值，其中 `x` 为 1 或 0 。但是不能用乘法，所以重新思考了一下，构造 `(y & x) + (z & ~x)` ，其中 `x` 为 `0x0` 或者 `0xffffffff` 。

+ 当 `x == 0` 时，需要构造 `x = 0x0 = -1 + 1 = ~0 + 1` 。
+ 当 `x != 0` 时，需要构造 `x = 0xffffffff = -1 = negate(1) = negate(!!x) = ~(!!x) + 1` 。

关键点是要知道 `!!x` 可以将 `int` 映射为逻辑真值。



### isLessEqual

```c
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y)
{
  int signx = (x >> 31) & 1;
  int signy = (y >> 31) & 1;
  int k1 = (signx ^ signy ^ 1) & (((x + (~y)) >> 31) & 1);
  int k2 = (signx & (signy ^ 1));
  return k1 | k2;
}
```

解题思路：[看「最大数值」这一小节](https://www.cnblogs.com/sinkinben/p/12323784.html) 。

### logicalNeg

```c
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int logicalNeg(int x)
{
  int neg = (x >> 31) & 1;
  int plus = ((~x + 1) >> 31) & 1;
  return (neg ^ 1) & (plus ^ 1);
}
```

解题思路：根据「0 既不是正数，也不是负数」这一个规律。所以如果 `x` 为 0 ，那么 `x` 和 `-x` 的符号位都必然是 `0` 。

### howManyBits

```c
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x)
{
  int b16, b8, b4, b2, b1;
  x = (x >> 31) ^ x;
  b16 = (!!(x >> 16)) << 4, x >>= b16;
  b8 = (!!(x >> 8)) << 3, x >>= b8;
  b4 = (!!(x >> 4)) << 2, x >>= b4;
  b2 = (!!(x >> 2)) << 1, x >>= b2;
  b1 = (!!(x >> 1)), x >>= b1;
  return b16 + b8 + b4 + b2 + b1 + x + 1;
}
```

解题思路：有点类似二分查找，本题自己没做出来，是看了几个题解才写出来的。

## Float

### floatScale2

```c
/* 
 * floatScale2 - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned floatScale2(unsigned uf)
{
  unsigned sign = (uf & 0x80000000);
  unsigned exp = (uf & 0x7f800000);
  unsigned frac = (uf & 0x7fffff);

  //clear sign bit
  uf = uf & (0x7fffffff);

  //special cases: INF, NaN, zero
  if (exp == 0x7f800000 || uf == 0)
    return uf | sign;

  if (exp == 0) // uf is a denormalize number
  {
    if ((frac >> 22) == 1) // 2*uf change into a normalized number
    {
      exp = (1 << 23);
      frac = (frac << 1) & 0x7fffff;
      return sign | exp | frac;
    }
    else // 2*uf is still a denormalized number
      return sign | (frac << 1);
  }
  else // uf is a normalized number, so 2*uf must be a normalized number, too
  {
    exp += (1 << 23);
    return sign | exp | frac;
  }
}
```

解题思路：需要熟悉[浮点数编码「IEEE 754」](https://www.cnblogs.com/sinkinben/p/12387524.html)，然后是分类讨论。

+ `uf` 是非规格化数

  - `2*uf` 仍然是非规格化数
  - `2*uf` 是规格化数

+ `uf` 是规格化数

  那么 `2*uf` 必然是规格化数。

+ 其他特殊情况

  `uf` 是无穷大，NaN，0 。

### floatFloat2Int

```c
/* 
 * floatFloat2Int - Return bit-level equivalent of expression (int) f
 *   for floating point argument f.
 *   Argument is passed as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point value.
 *   Anything out of range (including NaN and infinity) should return
 *   0x80000000u.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
int floatFloat2Int(unsigned uf)
{
  unsigned sign = (uf & 0x80000000);
  unsigned exp = (uf >> 23) & 0xff;
  unsigned real = (uf & 0x7fffff) | (1 << 23);
  unsigned t;
  uf = uf & 0x7fffffff;

  if (exp < 127) // uf < 1.0
    return 0;
  if (exp >= 150) // we can keep all 23 frac
  {
    t = exp - 150;
    if (t < 8) // round(uf) is in int range, and we can put no more than 7 zeros after real
    {
      real = real << t;
      return (sign == 0) ? real : (-real);
    }
    return 0x80000000; // out of range, we can't put more than 7 zeros after real
  }
  real = real >> (150 - exp); // the last serval bits of frac should be thrown
  return (sign == 0) ? real : (-real);
}
```

本题其实仍然是分类讨论，但是分类的依据与某些特殊情况，需要不断测试才能写出代码，所以一定不能畏难，不会就马上找题解，这样收获会很小。上面的代码也是修改了好多次的，我也忘了最初的版本长什么样子了。解法也没什么特别的，就是不断地试，观察不通过的 Test Case ，然后修改。

首先有一个 trick 是把隐含的 1 还原的：

```c
unsigned real = (uf & 0x7fffff) | (1 << 23);
/*
 * Here is a sample:
 *   155.55 = 0 10000110 00110111000110011001101
 *   frac   = 1.00110111000110011001101
 *   exp = 10000110 = 134, E = 7
 *   so the float val is:
 *     fval = 1.00110111000110011001101 × 2^7
 *          = 10011011.1000110011001101
 *   when fval cast into int, the '1000110011001101' will be thrown:
 *     ival = 0...0 10011011 = 155
 */
```

此时，我们默认小数点的位置在第 23 bit 与 第 22 bit 之间。

后面我们要解决的就是**小数点移动到后面的哪一位的问题**，这就要根据 `E = exp - 127` 的具体数值来决定：

+ 如果 `E >= 23` ，那么 `real` 后面的所有小数位都得以保留，并且需要添加 `E - 23` 个 0 。
+ 如果 `E <= 22` ，那么说明 `real` 的小数点最远就只能移动到第 1 bit 与第 0 bit 之间，小数点后的位都需要「丢弃」。

「丢弃」和「补 0」都通过移位来完成。

下面解析代码中的几个 `if` 条件。

+ `exp < 127`

  对于非规格化数，一定是小于 1.0 的，因为其值为 $denormalied = 0.frac \times 2^{-126}$ 。对于规格化数，$normalized = 1.frac \times 2^{exp - 127}$ ，要使 `uf < 1.0` ，就要令 $exp - 127 \le -1$ ，所以可得 `uf < 1.0` 的等价条件为 `exp <= 126` 。

+ `exp >= 150`

  此情况是可保留 23 位 `frac` 。令 `E = exp - 127 >= 23` ，即可得：`exp >= 150` 。超出 23 的部分就要补上 `exp - 150` 个 0 ，但是最多只能补 7 个 0 ，因为 `real` 本身就有 24 位，`int` 的符号位占 1 位。

  ```c
  /*
   * For an example:
   *   999999999.0 = 0 10011100 11011100110101100101000
   *   frac =   11011100110101100101000
   *   real = 1.11011100110101100101000
   *   exp = 10011100 = 0x9c = 156, E = 29
   *   so the float val is:
   *     fval = 1.11011100110101100101000 × 2^(29)
   *          = 111011100110101100101000 × 2^(6)
   *          = 111011100110101100101000 000000
   *   when cast into int val:
   *     ival = 111011100110101100101000 000000
   *   ival has 24 + 6 = 30 bits, put it into 32 bits is:
   *     ival = (00 111011100110101100101000 000000)2 = 1000000000
   *   the number '1000000000' is decimal
   */
  ```

+ `127 <= exp <= 149`

  这里是就是单纯的「舍弃」小数点后面的数值。看上面提到的例子：

  ```c
  /*
   * Here is a sample:
   *   155.55 = 0 10000110 00110111000110011001101
   *   real   = 1.00110111000110011001101
   *   exp = 10000110 = 134, E = 7
   *   so the float val is:
   *     fval = 1.00110111000110011001101 × 2^7
   *          = 10011011.1000110011001101
   *   when fval cast into int, the '1000110011001101' will be thrown:
   *     ival = 0...0 10011011 = 155
   */
  ```

### floatPower2

```c
/* 
 * floatPower2 - Return bit-level equivalent of the expression 2.0^x
 *   (2.0 raised to the power x) for any 32-bit integer x.
 *
 *   The unsigned value that is returned should have the identical bit
 *   representation as the single-precision floating-point number 2.0^x.
 *   If the result is too small to be represented as a denorm, return
 *   0. If too large, return +INF.
 * 
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. Also if, while 
 *   Max ops: 30 
 *   Rating: 4
 */
unsigned floatPower2(int x)
{
  int exp = x + 127;
  if (1 <= exp && exp <= 254) // 2^x can be a normalized number
    return exp << 23;
  else if (exp >= 255) // 2^x out of range
    return 0x7f800000;
  else // x <= -127 (exp <= 0), 2^x may be a denormalized number, maybe zero
  {
    if (x >= -149)
      return 1 << (149 + x);
    else
      return 0;
  }
}
```

本题要求实现 $2^x$ 。显然，这就是浮点数的一种十分特殊的情况：$1.0...0 \times 2^{exp - 127}$ 。但是 $exp \in [1,254]$ ，这需要对 `exp = E + 127 = x + 127` 进行讨论。**注意 `exp` 必须要使用有符号数进行存储，因为 `x + 127` 仍可能为负数** 。

+ `1 <= exp && exp <= 254`

  说明 $2^x$ 在规格化数的范围内，即 $val = 1.0\cdot\cdot\cdot0 \times 2^x$ ，所以 `frac = 0...0, exp = x + 127` 。

+ `exp >= 255`

  说明 $2^x$ 超过了 `float` 的有效范围。

+ `exp <= 0`

  这时候 $2^x$ 就必须采用非规格化数来存储。非规格化数 $denormalized = 0.frac \times 2^{-126}$ ，所以：

  + $2^x$ 的最大值为 $max = 0.10\cdot\cdot\cdot0 \times 2^{-126} = 2^{-127}, x=-127$

  + $2^x$ 的最小值为 $min = 0.0\cdot\cdot\cdot01 \times 2^{-126} = 2^{-149}, x=-149$

  $2^x$ 这种形式决定了 `frac` 中必定只有一个 1 ，而这个 1 的后面 0 的数目取决于 $x$ 的大小，实际上不难发现是 $149 + x$ 。

## Evaluation

```bash
unix > ./driver.pl
Correctness Results     Perf Results
Points  Rating  Errors  Points  Ops     Puzzle
1       1       0       2       8       bitXor
1       1       0       2       1       tmin
1       1       0       2       8       isTmax
2       2       0       2       7       allOddBits
2       2       0       2       2       negate
3       3       0       2       13      isAsciiDigit
3       3       0       2       8       conditional
3       3       0       2       14      isLessOrEqual
4       4       0       2       9       logicalNeg
4       4       0       2       32      howManyBits
4       4       0       2       22      floatScale2
4       4       0       2       18      floatFloat2Int
4       4       0       2       10      floatPower2

Score = 62/62 [36/36 Corr + 26/26 Perf] (152 total operators)
```



## Summary

Enjoy the fun of writing !