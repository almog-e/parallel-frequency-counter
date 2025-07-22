# Parallel Folding Count Algorithm

This Algorithm was developed as part of the **Parallel System Programming** course at Bar-Ilan University. It demonstrates a **parallel algorithm for counting the frequency of elements** in an array using folding, allowing efficient computation with **`n` threads and logarithmic time complexity**.
This Algorithm can be used as part of a Parallel Count Sort 
---

## Problem Overview

Given:

* An array `A` of size `N`
* Each value `A[i] ∈ [1, M]`
* Objective: Count how many times each number from `1` to `M` appears in `A`

### Naive Solutions:

* **Serial Count:** Linear scan, O(N) time
* **Parallel per-value prefix sum:** Requires O(M·log N) or O(N²) threads
* **Matrix Sum:** Build M×N matrix and sum each row — requires many threads or slow prefix scan

---

## The Idea: **Parallel Folding Count**

* Uses **N threads**
* Requires only **O(log N)** time
* Avoids prefix sums
* Avoids O(N²) thread usage

---

##  Algorithm Description

1. **Initialize a matrix** `B` of size `M × N`:
   * Rows represent each possible value in the input array `A` (from 1 to `M`).
   * Columns correspond to each index in `A`.
   * All cells are initialized to `0`.
     *(In practice, threads should distinguish between actual values and uninitialized memory if relevant.)*
2. **Populate the matrix in parallel**:
   For each thread `i`, write `1` to the cell `B[A[i]][i]`.

3. **Result**:
   After this step, each row `r` in matrix `B` contains a `1` in every column where the value `r` appeared in `A`.

---

##  Folding Phase (Key Idea)

Each thread performs a **fold** operation to move its value left in the row until all values accumulate at column 0.

In each iteration `i`, each element at column `j` does:

```math
B[r][j mod (N / 2^i)] += B[r][j]
```

* This merges elements **log N** times
* At the end, `B[r][0]` contains the total count of value `r`

---

##  Example Walkthrough

M is 6 so each value is a most 6
[1,6]

```math
A[N] = \begin{bmatrix}
3 & 1 & 5 & 4 & 2 & 1 & 3 & 4 & 1 & 6 & 1 & 3 & 4 & 6 & 1 & 4 
\end{bmatrix}
```
```math
B = \begin{bmatrix} 
 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\
 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0
\end{bmatrix}
```
We will look at B like this

```math
B[0] = \begin{bmatrix} 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}
```
```math
B[1] = \begin{bmatrix} 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}
```
```math
B[2] = \begin{bmatrix} 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}
```
```math
B[3] = \begin{bmatrix} 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}
```
```math
B[4] = \begin{bmatrix} 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}
```
```math
B[5] = \begin{bmatrix} 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}
```
```math
B[6] = \begin{bmatrix} 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}
```

Now each thread move its values to its row acording to the value

```math
A[N] = \begin{bmatrix}
3 & 1 & 5 & 4 & 2 & 1 & 3 & 4 & 1 & 6 & 1 & 3 & 4 & 6 & 1 & 4 \\
\downarrow & \downarrow & \downarrow & \downarrow & \downarrow & \downarrow & 
\downarrow & \downarrow & \downarrow & \downarrow & \downarrow & \downarrow & 
\downarrow & \downarrow & \downarrow & \downarrow
\end{bmatrix}
```

```math
B[0] = \begin{bmatrix} 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}
```
```math
B[1] = \begin{bmatrix} 0 & 1 & 0 & 0 & 0 & 1 & 0 & 0 & 1 & 0 & 1 & 0 & 0 & 0 & 1 & 0 \end{bmatrix}
```
```math
B[2] = \begin{bmatrix} 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}
```
```math
B[3] = \begin{bmatrix} 1 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 \end{bmatrix}
```
```math
B[4] = \begin{bmatrix} 0 & 0 & 0 & 1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 \end{bmatrix}
```
```math
B[5] = \begin{bmatrix} 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}
```
```math
B[6] = \begin{bmatrix} 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 1 & 0 & 0 \end{bmatrix}
```
now lets just look at an example
lets look just at row B[4] - so we can understand 

```math
B[4] = \begin{bmatrix} 0 & 0 & 0 & 1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 \end{bmatrix}
```
each 1 represents an appernce of trhe number 4 
lets call the frequency of number 4 to be $f_4$

we want to count the number of 1's without the need of n thread - insted we will only neet $f_4$ threads

```math
B[4] = \begin{bmatrix} 
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{black}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{black}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{black}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{black}{1} &
\end{bmatrix}
```
for each iteration i
we will now look at the half from the right side -> $\left[\frac{n}{2^i - 1},\ \frac{n}{2^i}\right]$
```math
B[4] = \begin{bmatrix} 
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{black}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{black}{1} &
\textcolor{blue}{\lvert} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{red}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{red}{1} &
\end{bmatrix}
```
At each folding step $i$, we divide the row into two halves of size $\frac{n}{2^i}$.

Any thread whose index $j$ satisfies:
$j \in \left[\frac{n}{2^i - 1},\ \frac{n}{2^i}\right]$  
(i.e., the **right half**) will:
- Take its value
- Move (or fold) it into the **left half** at index:
$j_{\text{target}} = j \bmod \left( \frac{n}{2^i} \right)$

<details>
<summary> Iteration 1 Folding of B[4]</summary>
 
```math
B[4] = \begin{bmatrix} 
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{black}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{black}{1} &
\textcolor{blue}{\lvert} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{red}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{red}{1} &
\end{bmatrix}
```
 
```math
B[4] = \begin{bmatrix} 
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{black}{1} & 
\textcolor{gray}{0} + \textcolor{red}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{black}{1} + \textcolor{red}{1} &
\textcolor{blue}{\lvert} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\end{bmatrix}
```
```math
B[4] = \begin{bmatrix} 
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{black}{1} & 
\textcolor{red}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{red}{2} &
\textcolor{blue}{\lvert} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\end{bmatrix}
```
</details>
<details>
<summary> Iteration 2 Folding of B[4]</summary>

 ```math
B[4] = \begin{bmatrix} 
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{black}{1} &
\textcolor{blue}{\lvert} & 
\textcolor{red}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{red}{2} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\end{bmatrix}
```
```math
B[4] = \begin{bmatrix} 
\textcolor{gray}{0} + \textcolor{red}{1} &
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{black}{1} + \textcolor{red}{2} &
\textcolor{blue}{\lvert} & 
\textcolor{red}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{2} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\end{bmatrix}
```
```math
B[4] = \begin{bmatrix} 
\textcolor{black}{1} &
\textcolor{gray}{0} &
\textcolor{gray}{0} &
\textcolor{black}{3} &
\textcolor{blue}{\lvert} & 
\textcolor{gray}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{2} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\end{bmatrix}
```
</details>

<details>
<summary> Iteration 3 Folding of B[4]</summary>

```math
B[4] = \begin{bmatrix} 
\textcolor{black}{1} &
\textcolor{gray}{0} &
\textcolor{blue}{\lvert} & 
\textcolor{gray}{0} &
\textcolor{red}{3} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{2} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\end{bmatrix}
```
```math
B[4] = \begin{bmatrix} 
\textcolor{black}{1} &
\textcolor{gray}{0}+ \textcolor{red}{3} &
\textcolor{blue}{\lvert} & 
\textcolor{gray}{0} &
\textcolor{gray}{3} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{2} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\end{bmatrix}
```
```math
B[4] = \begin{bmatrix} 
\textcolor{black}{1} &
\textcolor{black}{3} &
\textcolor{blue}{\lvert} & 
\textcolor{gray}{0} &
\textcolor{gray}{3} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{2} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\end{bmatrix}
```
</details>

<details>
<summary> Iteration 4 Folding of B[4]</summary>

```math
B[4] = \begin{bmatrix} 
\textcolor{black}{1} &
\textcolor{blue}{\lvert} & 
\textcolor{red}{3} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{0} &
\textcolor{gray}{3} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{2} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\end{bmatrix}
```
```math
B[4] = \begin{bmatrix} 
\textcolor{black}{1} + \textcolor{red}{3}&
\textcolor{blue}{\lvert} & 
\textcolor{gray}{3} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{0} &
\textcolor{gray}{3} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{2} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\end{bmatrix}
```
```math
B[4] = \begin{bmatrix} 
\textcolor{black}{4}&
\textcolor{blue}{\lvert} & 
\textcolor{gray}{3} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{0} &
\textcolor{gray}{3} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{1} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{2} &
\textcolor{gray}{\lvert} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\textcolor{gray}{0} & 
\textcolor{gray}{0} &
\textcolor{gray}{1} &
\end{bmatrix}
```
</details>

Now after at most iter
we got 
```math
B[4][0] = \begin{bmatrix} 
\textcolor{black}{4}
\end{bmatrix}
```


so now let's look at the martix (without row 0)


```math
B[1][0:\frac{n}{1}] = \begin{bmatrix} 0 & 1 & 0 & 0 & 0 & 1 & 0 & 0 & 1 & 0 & 1 & 0 & 0 & 0 & 1 & 0 \end{bmatrix}
```
```math
B[2][0:\frac{n}{1}] = \begin{bmatrix} 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}
```
```math
B[3][0:\frac{n}{1}] = \begin{bmatrix} 1 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 \end{bmatrix}
```
```math
B[4][0:\frac{n}{1}] = \begin{bmatrix} 0 & 0 & 0 & 1 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 \end{bmatrix}
```
```math
B[5][0:\frac{n}{1}] = \begin{bmatrix} 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}
```
```math
B[6][0:\frac{n}{1}] = \begin{bmatrix} 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 1 & 0 & 0 \end{bmatrix}
```
</details>

<details>
<summary> Iteration 1 Folding of B</summary>


```math
B[1][0:\frac{n}{2}] = \begin{bmatrix} 1 & 1 & 1 & 0 & 0 & 1 & 1 & 0\end{bmatrix}
```
```math
B[2][0:\frac{n}{2}] = \begin{bmatrix} 0 & 0 & 0 & 0 & 1 & 0 & 0 & 0\end{bmatrix}
```
```math
B[3][0:\frac{n}{2}] = \begin{bmatrix} 1 & 0 & 0 & 1 & 0 & 0 & 1 & 0\end{bmatrix}
```
```math
B[4][0:\frac{n}{2}] = \begin{bmatrix} 0 & 0 & 0 & 1 & 1 & 0 & 0 & 2\end{bmatrix}
```
```math
B[5][0:\frac{n}{2}] = \begin{bmatrix} 0 & 0 & 1 & 0 & 0 & 0 & 0 & 0\end{bmatrix}
```
```math
B[6][0:\frac{n}{2}] = \begin{bmatrix} 0 & 1 & 0 & 0 & 0 & 1 & 0 & 0\end{bmatrix}
```
</details>
<details>
<summary> Iteration 2 Folding of B</summary>

```math
B[1][0:\frac{n}{4}] = \begin{bmatrix} 1 & 2 & 2 & 0\end{bmatrix}
```
```math
B[2][0:\frac{n}{4}] = \begin{bmatrix} 1 & 0 & 0 & 0\end{bmatrix}
```
```math
B[3][0:\frac{n}{4}] = \begin{bmatrix} 1 & 0 & 1 & 1\end{bmatrix}
```
```math
B[4][0:\frac{n}{4}] = \begin{bmatrix} 1 & 0 & 0 & 3\end{bmatrix}
```
```math
B[5][0:\frac{n}{4}] = \begin{bmatrix} 0 & 0 & 1 & 0\end{bmatrix}
```
```math
B[6][0:\frac{n}{4}] = \begin{bmatrix} 0 & 2 & 0 & 0\end{bmatrix}
```
</details>

<details>
<summary> Iteration 3 Folding of B</summary>

```math
B[1][0:\frac{n}{8}] = \begin{bmatrix} 3 & 2\end{bmatrix}
```
```math
B[2][0:\frac{n}{8}] = \begin{bmatrix} 1 & 0 \end{bmatrix}
```
```math
B[3][0:\frac{n}{8}] = \begin{bmatrix} 2 & 1\end{bmatrix}
```
```math
B[4][0:\frac{n}{8}] = \begin{bmatrix} 1 & 3\end{bmatrix}
```
```math
B[5][0:\frac{n}{8}] = \begin{bmatrix} 1 & 0\end{bmatrix}
```
```math
B[6][0:\frac{n}{8}] = \begin{bmatrix} 0 & 2\end{bmatrix}
```
</details>

<details>
<summary> Iteration 4 Folding of B</summary>

```math
B[1][0:\frac{n}{16}] = \begin{bmatrix} 5\end{bmatrix}
```
```math
B[2][0:\frac{n}{16}] = \begin{bmatrix} 1 \end{bmatrix}
```
```math
B[3][0:\frac{n}{16}] = \begin{bmatrix} 3\end{bmatrix}
```
```math
B[4][0:\frac{n}{16}] = \begin{bmatrix} 4\end{bmatrix}
```
```math
B[5][0:\frac{n}{16}] = \begin{bmatrix} 1\end{bmatrix}
```
```math
B[6][0:\frac{n}{16}] = \begin{bmatrix} 2\end{bmatrix}
```
</details>
 
then we get $f_1 = 5$ $f_2 = 1$ $f_3 = 3$ $f_4 = 4$ $f_5 = 1$ $f_6 = 2$ 

You can see how each row folds values to the left, summing in log steps.

---

## Final Output

After all folding steps:

* Row `r` of `B` at column `0` holds the **frequency of number `r`**
* Total time complexity: **O(log N)**
* Total space: O(M · N)
* Total threads: **N**

---

##  Performance

| Metric           | Value                            |
| ---------------- | -------------------------------- |
| Time Complexity  | O(log N)                         |
| Space Complexity | O(M × N)                         |
| Thread Count     | N (one per input index)          |
| Avoids           | Prefix sum per element or number |

---

