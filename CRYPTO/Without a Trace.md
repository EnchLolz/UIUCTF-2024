# Summarize
Category: Crypto

Points: 246

Solves: 298

### Solution

At first the challenge looks complicated but upon closer inspection it's not that bad.
```py
def inputs():
    print("[WAT] Define diag(u1, u2, u3. u4, u5)")
    M = [
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
    ]
    for i in range(5):
        try:
            M[i][i] = int(input(f"[WAT] u{i + 1} = "))
        except:
            return None
    return M
```

It just asks us to provide 5 numbers for the diagonal of the matrix.
```py
def check(M):
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
...
elif check(M) == 0:
    print("[WAT] It's not going to be that easy...")
```

This check just makes sure the determinant of our matrix isn't zero. All this means is that our inputs must be non-zero.

```py
def fun(M):
    f = [bytes_to_long(bytes(FLAG[5*i:5*(i+1)], 'utf-8')) for i in range(5)]
    F = [
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0],
    ]
    for i in range(5):
        F[i][i] = f[i]

    try:
        R = np.matmul(F, M)
        return np.trace(R)

    except:
        print("[WAT] You're trying too hard, try something simpler")
        return None
```
Finally they hide parts of the flag in the diagonal of another matrix, multiply the two matrices, and return the sum of the diagonal.

The matrix multiplication is rather simple:
\[
\mathbf{A} = \begin{pmatrix}
a & 0 & 0 & 0 & 0 \\
0 & b & 0 & 0 & 0 \\
0 & 0 & c & 0 & 0 \\
0 & 0 & 0 & d & 0 \\
0 & 0 & 0 & 0 & e
\end{pmatrix}
\]

\[
\mathbf{B} = \begin{pmatrix}
v & 0 & 0 & 0 & 0 \\
0 & w & 0 & 0 & 0 \\
0 & 0 & x & 0 & 0 \\
0 & 0 & 0 & y & 0 \\
0 & 0 & 0 & 0 & z
\end{pmatrix}
\]

The resulting matrix \(\mathbf{C}\) from the product \(\mathbf{A} \times \mathbf{B}\) is:

\[
\mathbf{C} = \mathbf{A} \times \mathbf{B} = \begin{pmatrix}
a \cdot v & 0 & 0 & 0 & 0 \\
0 & b \cdot w & 0 & 0 & 0 \\
0 & 0 & c \cdot x & 0 & 0 \\
0 & 0 & 0 & d \cdot y & 0 \\
0 & 0 & 0 & 0 & e \cdot z
\end{pmatrix}
\]

This means that we are just getting `flag1*inp1 + ... + flag5*inp5`. We can just enter a diagonal of 1's to gain the sum of flag parts. Next we enter diag (2, 1, 1, 1, 1). Subtracting the resulting numbers will give us the first flag part. We now do this for the other 4 indicies.

![diag of 1](/images/WithoutATrace1.png)

![diag with 2](/images/WithoutATrace2.png)

```py
>>> from Crypto.Util.number import long_to_bytes
>>> 2504408575853-2000128101369
504280474484
>>> long_to_bytes(504280474484)
b'uiuct'
>>> 2440285994541-2000128101369
440157893172
>>> long_to_bytes(440157893172)
b'f{tr4'
>>> 2426159182680-2000128101369
426031081311
>>> long_to_bytes(426031081311)
b'c1ng_'
>>> 2163980646766-2000128101369
163852545397
>>> long_to_bytes(163852545397)
b'&&_mu'
>>> 2465934208374-2000128101369
465806107005
>>> long_to_bytes(465806107005)
b'lt5!}'
```

Now we just put these parts together for the flag.


### Flag

```uiuctf{tr4c1ng_&&_mult5!}```
