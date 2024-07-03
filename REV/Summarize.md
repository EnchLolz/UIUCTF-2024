# Summarize
Category: Rev

Points: 381

Solves: 161

### Solution

We are given an ELF file to analyze which takes 6 9-digit numbers as input. It will then run a check function to verify these 9 numbers then output the flag as the hex values concatenated.

```c
puts(
    "To get the flag, you must correctly enter six 9-digit positive integers: a, b, c, d, e, and f ."
    );
putchar(10);
printf("a = ");
__isoc99_scanf(&DAT_0040206c,&a);
...
printf("f = ");
__isoc99_scanf(&DAT_0040206c,&f);
// renamed to check function. 
cVar1 = check(a,b,c,d,e,f);
if (cVar1 == '\0') {
    puts("Wrong.");
}
else {
    puts("Correct.");
    // calls sprintf
    FUN_004010e0(local_48,"uiuctf{%x%x%x%x%x%x}",a,b,c,d,e,f);
    puts(local_48);
}
```

Now that we know what we are looking for let's see the check function for `a b c d e f`

After renaming variables we get:

```c

if (((((a < 100000001) || (b < 0x5f5e101)) || (c < 0x5f5e101)) ||
    ((d < 0x5f5e101 || (e < 0x5f5e101)))) || (f < 0x5f5e101)) {
uVar1 = 0;
}
else if (((a < 1000000000) && (b < 1000000000)) &&
        ((c < 1000000000 && (((d < 1000000000 && (e < 1000000000)) && (f < 1000000000)))))) {
uVar1 = FUN_004016d8(a,b);
uVar2 = FUN_0040163d(uVar1,c);
uVar3 = FUN_0040163d(a,b);
uVar1 = FUN_004016fe(2,b);
uVar4 = FUN_004016fe(3,a);
uVar5 = FUN_004016d8(uVar4,uVar1);
uVar6 = FUN_0040174a(a,d);
uVar1 = FUN_0040163d(c,a);
uVar7 = FUN_004017a9(b,uVar1);
uVar11 = FUN_0040163d(b,d);
uVar1 = FUN_0040163d(d,f);
uVar8 = FUN_0040174a(c,uVar1);
uVar9 = FUN_004016d8(e,f);
uVar10 = FUN_0040163d(e,f);
if ((((uVar2 % 0x10ae961 == 0x3f29b9) && (uVar3 % 0x1093a1d == 0x8bdcd2)) &&
    ((uVar5 % uVar6 == 0x212c944d &&
        ((uVar7 % 0x6e22 == 0x31be && ((int)((uVar11 & 0xffffffff) % (ulong)a) == 0x2038c43c))))))
    && ((uVar8 % 0x1ce628 == 0x1386e2 &&
        ((uVar9 % 0x1172502 == 0x103cf4f && (uVar10 % 0x2e16f83 == 0x16ab0d7)))))) {
    uVar1 = 1;
}
else {
    uVar1 = 0;
}
}
else {
uVar1 = 0;
}
return uVar1;
```

The code first checks that all 6 numbers are 9 digits in the range [100000001,1000000000). Then we create 11 new vars through a series of convoluted functions. Finally, it runs a check through many modular equations.

**Reversing Functions**

Now lets go through each function to see what they do.

```c
void FUN_004016d8(undefined4 param_1,int param_2)
{
  FUN_0040163d(param_1,-param_2);
  return;
}
```

Darn, we need to go deeper.

```c
long FUN_0040163d(uint param_1,uint param_2)
{
  byte bVar1;
  uint uVar2;
  uint uVar3;
  uint local_30;
  uint local_2c;
  uint local_20;
  long local_10;
  
  local_10 = 0;
  local_20 = 0;
  bVar1 = 0;
  local_2c = param_1;
  for (local_30 = param_2; (local_2c != 0 || (local_30 != 0)); local_30 = local_30 >> 1) {
    // bit 1
    uVar2 = local_2c & 1;
    // bit 2
    uVar3 = local_30 & 1;
    local_2c = local_2c >> 1;
    // xor bit1 bit2 and carry
    local_10 = local_10 + (ulong)((uVar2 ^ uVar3 ^ local_20) << (bVar1 & 0x1f));
    // set carry is sum >= 2
    local_20 = uVar3 & local_20 | uVar2 & (uVar3 | local_20);
    bVar1 = bVar1 + 1;
  }
  return local_10 + ((ulong)local_20 << (bVar1 & 0x3f));
}
```

This seems like a huge pain, luckily for us we can just test values for param_1 and param_2 to see if any patterns show up. Putting this in a C sandbox, then trying 1 and 2 yields 3. Further testing confirms this as just adding param_1 and param_2. (You can also verify that this is addition since the code is just looping over all bits and using a [bit adder](https://en.wikipedia.org/wiki/Adder_(electronics)))

This also means that the first function we tested is just subtraction:
```c
void FUN_004016d8(undefined4 param_1,int param_2)
{
  add(param_1,-param_2);
  return;
}
```

Now let's go to the third function.

```c
int FUN_004016fe(uint param_1,int param_2)
{
  byte bVar1;
  uint local_1c;
  int local_14;
  
  local_14 = 0;
  bVar1 = 0;
  // loop through each bit in param_1
  for (local_1c = param_1; local_1c != 0; local_1c = local_1c >> 1) {
    // add param_2 << b if bth bit is set in param1
    local_14 = local_14 + (param_2 << (bVar1 & 0x1f)) * (local_1c & 1);
    bVar1 = bVar1 + 1;
  }
  return local_14;
}
```

Again let's just test random values. (1, 2) => 2 , (3,20) => 60. Multiplication.
(The code adds num2 * 2^b^ if the *b*th bit is set in num1)

There seems to be a pattern of elementary operations here, so lets keep going.

The last 2 functions look very similar with only a '^' being swapped with '&'

```c
int FUN_0040174a(uint param_1,uint param_2)
{
  byte bVar1;
  uint uVar2;
  uint local_20;
  uint local_1c;
  int local_18;
  
  local_18 = 0;
  bVar1 = 0;
  local_1c = param_1;
  for (local_20 = param_2; (local_1c != 0 || (local_20 != 0)); local_20 = local_20 >> 1) {
    uVar2 = local_1c & 1;
    local_1c = local_1c >> 1;
    local_18 = local_18 + ((uVar2 ^ local_20 & 1) << (bVar1 & 0x1f));
    bVar1 = bVar1 + 1;
  }
  return local_18;
}
```
```c
int FUN_004017a9(uint param_1,uint param_2)
{
  byte bVar1;
  uint uVar2;
  uint local_20;
  uint local_1c;
  int local_18;
  
  local_18 = 0;
  bVar1 = 0;
  local_1c = param_1;
  for (local_20 = param_2; (local_1c != 0 || (local_20 != 0)); local_20 = local_20 >> 1) {
    uVar2 = local_1c & 1;
    local_1c = local_1c >> 1;
    local_18 = local_18 + ((uVar2 & local_20 & 1) << (bVar1 & 0x1f));
    bVar1 = bVar1 + 1;
  }
  return local_18;
}
```

So again I tested random values in a sandbox. As expected I was able to deduce that these functions were bitwise xor and bitwise and respectively.
(Again in both these function it seems to just be looping over each bit, checking xor/and then appending that result to the return value)

**Simplifying Code**

Now that we figured out each function, we can simplify our code.
```c
// renamed functions
ok = subtract(a,b);
uVar1 = add(ok,c);
uVar2 = add(a,b);
ok = multiply(2,b);
uVar3 = multiply(3,a);
uVar4 = subtract(uVar3,ok);
uVar5 = xor(a,d);
ok = add(c,a);
uVar6 = and(b,ok);
uVar10 = add(b,d);
ok = add(d,f);
uVar7 = xor(c,ok);
uVar8 = subtract(e,f);
uVar9 = add(e,f);
if ((((uVar1 % 0x10ae961 == 0x3f29b9) && (uVar2 % 0x1093a1d == 0x8bdcd2)) &&
    ((uVar4 % uVar5 == 0x212c944d &&
        ((uVar6 % 0x6e22 == 0x31be && ((int)((uVar10 & 0xffffffff) % (ulong)a) == 0x2038c43c))))))
    && ((uVar7 % 0x1ce628 == 0x1386e2 &&
        ((uVar8 % 0x1172502 == 0x103cf4f && (uVar9 % 0x2e16f83 == 0x16ab0d7)))))) {
    ok = 1;
}
```

```c
// replace functions with operations and eliminate temp vars
uVar1 = a+c-b;
uVar2 = a+b;
uVar3 = 3*a;
uVar4 = 3*a-2*b;
uVar5 = a^d;
uVar6 = b&(c+a);
uVar10 = b+d;
uVar7 = c^(d+f);
uVar8 = e-f;
uVar9 = e+f;
```

```c
// clean up equations
uVar1 % 17492321 == 4139449
uVar2 % 17381917 == 9166034
uVar4 % uVar5 == 556569677
uVar6 % 28194 == 12734
uVar10 % a == 540591164 // 4294967295 is a string of 1's and masks nothing
uVar7 % 1893928 == 1279714
uVar8 % 18294018 == 17026895
uVar9 % 48328579 == 23769303
```

**Solve for Values**

Now that we have these equations, we can brute force valid ranges for some of the uVars. Then we use some math to solve for `a b c d e f`. (We also want to reduce brute force length.)

Somethings we considered while writing script:
- uVar2 and uVar4 nicely solves for a and b
    - We can solve d from uVar5
        - We can check a, b, and d using uVar10: uVar10 = b+d and uVar10 % a = *Res10*
    - We can solve c from uVar1
    - uVar5 and uVar4 are annoying to bruteforce, likely need to be done together
 
- uVar6 has too many options from a small mod, try not to brute force
- e and f are self contained in uVar8 and uVar9
    - We can check these numbers last since they aren't very useful
        - only f is used, but it's in combination with c and d
    - We can also precompute all possibilties

**Script**

```c++
#include <iostream>
#include <cassert>

// [MIN, MAX)
const int MIN = 100000001;
const int MAX = 1000000000;

const int MOD1 = 17492321;
const int RES1 = 4139449;

const int MOD2 = 17381917;
const int RES2 = 9166034;
const int MIN2 = RES2 + MOD2 * ((2 * MIN - RES2 + MOD2 - 1) / MOD2);
static_assert(2 * MIN <= MIN2 && 2 * MIN > MIN2 - MOD2 && MIN2 % MOD2 == RES2);

const int RES4 = 556569677;

const int MAX5 = 1073741824;  // 2**30

const int MOD6 = 28194;
const int RES6 = 12734;

const int RES10 = 540591164;

const int MOD7 = 1893928;
const int RES7 = 1279714;

const int MOD8 = 18294018;
const int RES8 = 17026895;

const int MOD9 = 48328579;
const int RES9 = 23769303;

const int EF_OPTIONS = 450;

int main() {
    // precompute valid E and F options since they are independant of other vars
    int n_ef = 0;
    int e_opts[EF_OPTIONS];
    int f_opts[EF_OPTIONS];
    for (int var8 = RES8; var8 < MAX; var8 += MOD8) {  // 54
        for (long long var9 = RES9; var9 < 2 * MAX; var9 += MOD9) {  // 41
            if ((var8 + var9) & 1) continue;
            const int e = (var8 + var9) >> 1;
            const int f = var9 - e;
            if (e < MIN || e >= MAX || f < MIN || f >= MAX) continue;
            e_opts[n_ef] = e;
            f_opts[n_ef++] = f;
        }
    }

    assert(n_ef == EF_OPTIONS);
    std::cout << "There are " << n_ef << " options for e and f." << std::endl;
    // loop through all uVar5 = a^d in `uVar4 % uVar5 == 556569677`
    for (int var5 = RES4 + 1; var5 < MAX5; ++var5) {  // 5.17e8
        if ((var5 & 0xffffff) == 0) std::cerr << "var5 = " << var5 << std::endl;
        // loop through all uVar4 = 3*a-2*b given uVar5
        for (unsigned int var4 = RES4; var4 < 3u * MAX; var4 += var5) {  // 5
            // loop through all uVar2 = a+b, `uVar2 % 17381917 == 9166034`
            for (unsigned int var2 = MIN2; var2 < 2 * MAX; var2 += MOD2) {  // 115
                if ((2 * var2 + var4) % 5 != 0) continue;
                // solve for a: 2*uVar2 + uVar4 = 2(a+b) + 3a-2b = 5a 
                const int a = (2 * var2 + var4) / 5;
                // solve for b: uVar2 = a+b --> uVar2 - a = b
                const int b = (int)var2 - a;
                // solve for d: uVar5 = a^d --> uVar5 ^ a = d
                const int d = var5 ^ a;

                if (a < MIN || a >= MAX || b < MIN || b >= MAX || d < MIN || d >= MAX) continue;

                // solve for uVar10: uVar10 = b+d;
                const int var10 = b + d;
                // check it is valid: uVar10 % a == 540591164
                if (var10 % a != RES10) continue;

                std::cerr << "a, b, d: " << a << ' ' << b << ' ' << d << std::endl;

                // loop through all uVar1 = a+c-b, `uVar1 % 17492321 == 4139449`
                for (int var1 = RES1; var1 < 2 * MAX; var1 += MOD1) {  // 114
                    // solve for c: uVar1 = a+c-b --> uVar1-a+b = c
                    const int c = var1 - a + b;
                    if (c < MIN || c >= MAX) continue;

                    // solve for uVar6: uVar6 = b&(c+a)
                    const int var6 = b & (c + a);
                    // check it is valid: uVar6 % 28194 == 12734
                    if (var6 % MOD6 != RES6) continue;

                    // loop through E F possibilities
                    for (int i = 0; i < EF_OPTIONS; ++i) {
                        // solve for uVar7: uVar7 = c^(d+f)
                        const int var7 = c ^ (d + f_opts[i]);
                        // check it is valid: uVar7 % 1893928 == 1279714
                        if (var7 % MOD7 != RES7) continue;

                        // found answer
                        const int e = e_opts[i];
                        const int f = f_opts[i];
                        std::cout << "Found an answer!\n";
                        std::cout << a << ' ' << b << ' ' << c << ' ' << d << ' ' << e << ' ' << f << std::endl;
                    }
                }
            }
        }
    }
}
```



After running this for a while we get:
`705965527 780663452 341222189 465893239 966221407 217433792`

Hexed:
`2a142dd7 2e87fa9c 1456a32d 1bc4f777 39975e5f cf5c6c0`

Add flag wrapper:
`uiuctf{2a142dd72e87fa9c1456a32d1bc4f77739975e5fcf5c6c0}`
### Flag

```uiuctf{2a142dd72e87fa9c1456a32d1bc4f77739975e5fcf5c6c0}```
