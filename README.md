---

# Parallel Folding Count Algorithm

This project was developed as part of the **Parallel System Programming** course at Bar-Ilan University. It demonstrates a **parallel algorithm for counting the frequency of elements** in an array using folding, allowing efficient computation with **`n` threads and logarithmic time complexity**.

---

## 🧠 Problem Overview

Given:

* An array `A` of size `N`
* Each value `A[i] ∈ [1, M]`
* Objective: Count how many times each number from `1` to `M` appears in `A`

### Naive Solutions:

* **Serial Count:** Linear scan, O(N) time
* **Parallel per-value prefix sum:** Requires O(M·log N) or O(N²) threads
* **Matrix Sum:** Build M×N matrix and sum each row — requires many threads or slow prefix scan

---

## 💡 Our Idea: **Parallel Folding Count**

We propose a method that:

* Uses **N threads**
* Requires only **O(log N)** time
* Avoids prefix sums
* Avoids O(N²) thread usage

---

## 📐 Algorithm Description

1. Create a matrix `B` of size `M × N`

   * Rows: One for each possible value in `A` (from 1 to M)
   * Columns: One for each index in `A`
2. For each thread `i`, place the value `A[i]` in `B[A[i]][i]`
3. Now each row `r` in `B` contains all occurrences of the number `r`

---

## 🔁 Folding Phase (Key Idea)

Each thread performs a **fold** operation to move its value left in the row until all values accumulate at column 0.

In each iteration `i`, each element at column `j` does:

```math
B[r][j mod (N / 2^i)] += B[r][j]
```

* This merges elements **log N** times
* At the end, `B[r][0]` contains the total count of value `r`

---

## 💻 Example Walkthrough

> Suppose `A = [3, 1, 5, 4, 2, 1, 3, 4, 1, 6, 1, 3, 4, 6, 1, 4]`

We initialize `B` and fold it step by step:

<p align="center">
  <img src="./your_image_path_1.png" width="500"/>
  <img src="./your_image_path_2.png" width="500"/>
  <img src="./your_image_path_3.png" width="500"/>
</p>

You can see how each row folds values to the left, summing in log steps.

---

## 🧮 Final Output

After all folding steps:

* Row `r` of `B` at column `0` holds the **frequency of number `r`**
* Total time complexity: **O(log N)**
* Total space: O(M · N)
* Total threads: **N**

---
<pre> A = [3 1 5 4 2 1 3 4 1 6 1 3 4 6 1 4] M is [1,6] N is 16 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 | 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 2 | 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 3 | 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 4 | 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 5 | 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 6 | 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 </pre>
## 📊 Performance

| Metric           | Value                            |
| ---------------- | -------------------------------- |
| Time Complexity  | O(log N)                         |
| Space Complexity | O(M × N)                         |
| Thread Count     | N (one per input index)          |
| Avoids           | Prefix sum per element or number |

---

