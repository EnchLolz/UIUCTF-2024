# Determined
Category: Crypto

Points: 322

Solves: 222

### Solution

Looking at `gen.py` and `gen.txt` it seems like we have a standard RSA with a random `r` value:

```py
from SECRET import FLAG, p, q, r
from Crypto.Util.number import bytes_to_long
n = p * q
e = 65535
m = bytes_to_long(FLAG)
c = pow(m, e, n)
# printed to gen.txt
print(f"{n = }")
print(f"{e = }")
print(f"{c = }")
```

```
n = 158794636700752922781275926476194117856757725604680390949164778150869764326023702391967976086363365534718230514141547968577753309521188288428236024251993839560087229636799779157903650823700424848036276986652311165197569877428810358366358203174595667453056843209344115949077094799081260298678936223331932826351
e = 65535
c = 72186625991702159773441286864850566837138114624570350089877959520356759693054091827950124758916323653021925443200239303328819702117245200182521971965172749321771266746783797202515535351816124885833031875091162736190721470393029924557370228547165074694258453101355875242872797209141366404264775972151904835111
```

Now lets analyze `chal.py` to see how we can obtain `p` and `q`:

```py
def inputs():
    print("[DET] First things first, gimme some numbers:")
    M = [
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
    ]
    try:
        M[0][0] = p
        M[0][2] = int(input("[DET] M[0][2] = "))
        M[0][4] = int(input("[DET] M[0][4] = "))

        M[1][1] = int(input("[DET] M[1][1] = "))
        M[1][3] = int(input("[DET] M[1][3] = "))

        M[2][0] = int(input("[DET] M[2][0] = "))
        M[2][2] = int(input("[DET] M[2][2] = "))
        M[2][4] = int(input("[DET] M[2][4] = "))

        M[3][1] = q
        M[3][3] = r

        M[4][0] = int(input("[DET] M[4][0] = "))
```

We are asked to enter some values in a \(5 \times 5\) matrix. After carefully drawing out the \(5 \times 5\) matrix we get:

\[
\begin{pmatrix}
p & 0 & a & 0 & b \\
0 & c & 0 & d & 0 \\
e & 0 & f & 0 & g \\
0 & q & 0 & r & 0 \\
j & 0 & k & 0 & 0
\end{pmatrix}
\]

where $a,b,c,d,e,f,g,j,k$ are the inputs and $p,q,r$ are the hidden values.

Then the program outputs `fun` of this matrix:

```py
def fun(M):
    ...

def main():
    ...
    res = fun(M)
    print(f"[DET] Have fun: {res}")
```

Now let's see what `fun` does:

```py
def fun(M):
    def sign(sigma):
        l = 0
        for i in range(5):
            for j in range(i + 1, 5):
                if sigma[i] > sigma[j]:
                    l += 1
        return (-1)**l

    res = 0
    for sigma in permutations([0,1,2,3,4]):
        curr = 1
        for i in range(5):
            curr *= M[sigma[i]][i]
        res += sign(sigma) * curr
    return res
```


The function `fun` computes the determinant of a \(5 \times 5\) matrix \(M\) using the [definition involving permutations](https://en.wikipedia.org/wiki/Determinant#n_%C3%97_n_matrices).

1. **Define `sign(sigma)` function**:
    - This nested function computes the sign (or signature) of a permutation \(\sigma\). The sign is determined by the number of inversions in the permutation.
    - If the number of inversions is even, the sign is \(+1\); if odd, the sign is \(-1\).

2. **Initialize `res` to 0**:
    - `res` will store the sum of the terms of the determinant expansion.

3. **Iterate over all permutations of \([0,1,2,3,4]\)**:

    - For each permutation `sigma`, the function computes the corresponding term in the determinant expansion.

4. **Compute the product of elements for the current permutation**:
    - For each permutation `sigma`, compute the product of matrix elements \(M[\sigma[i]][i]\) for \(i\) from 0 to 4. This represents one term in the determinant formula.

5. **Multiply the product by the sign of the permutation**:
    - Multiply the computed product by the sign of the permutation (`sign(sigma)`).

6. **Add the result to `res`**:
    - Accumulate the result in `res`.

7. **Return `res`**:
    - Finally, return `res`, which is the determinant of the matrix \(M\).

Or we could just guess that it's the determinant from the name of this challenge!!!

Great! Now let's calculate our determinant algebraically, since there are a lot of zeros, the final expression is probably not that long.

Plugging the matrix into [WolframAlpha](https://www.wolframalpha.com/input?i=%5B%5Bp%2C0%2Ca%2C0%2Cb%5D%2C%5B0%2Cc%2C0%2Cd%2C0%5D%2C%5Be%2C0%2Cf%2C0%2Cg%5D%2C%5B0%2Cq%2C0%2Cr%2C0%5D%2C%5Bj%2C0%2Ck%2C0%2C0%5D) gets us the expression:

$(d q - c r) (-a g j - b e k + b f j + g k p)$

From here, we want to find a way to get `p` or `q`. The easiest way is likely to find some multiple of `p` or some multiple of `q` (but not both), then GCD it with `n`.

From here we have many options:

1. Multiple of q: we can set `c = 0`, and plug in random (non-zero) values for the other inputs to get $(dq-0)(smth)$

2. Multiple of p: we set `a = b = 0`, and random (non-zero) values for the rest to get $(smth)(0-0+0+gkp)$

![nc remote](/images/DeterminedNC.png)

Finally we just GCD this value with `n`.

```py
>>> res = 69143703864818353130371867063903344721913499237213762852365448402106351546379534901008421404055139746270928626550203483668654060066917995961156948563756763620082694010964627379305998217766380375890208111586340805642016329892173840181871705216180866148524640933007104761115889694872164169659985765092765835334
>>> n = 158794636700752922781275926476194117856757725604680390949164778150869764326023702391967976086363365534718230514141547968577753309521188288428236024251993839560087229636799779157903650823700424848036276986652311165197569877428810358366358203174595667453056843209344115949077094799081260298678936223331932826351
>>> math.gcd(res,n)
12664210650260034332394963174199526364356188908394557935639380180146845030597260557316300143360207302285226422265688727606524707066404145712652679087174119
>>> n//math.gcd(res,n)
12538849920148192679761652092105069246209474199128389797086660026385076472391166777813291228233723977872612370392782047693930516418386543325438897742835129
>>> assert math.gcd(res,n) * (n//math.gcd(res,n)) == n
>>>
```

Plugging in these values into dcode yields the flag:

![Dcode](/images/DeterminedDCODE.png)

### Flag

```uiuctf{h4rd_w0rk_&&_d3t3rm1n4t10n}```
