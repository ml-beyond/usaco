---
description: Jan 2025 Silver
---

# Farmer John's Favorite Operation

[https://usaco.org/index.php?page=viewproblem2\&cpid=1471 ](https://usaco.org/index.php?page=viewproblem2\&cpid=1471)

Brute-Force: 2 out of 16, others timed out

It loops over all `j` in `0..M-1`, which can be up to **1e9** → impossible.

<details>

<summary>Change to <strong>1-median on a circle problem</strong>.</summary>

You want to choose an integer ( `x` ) that minimizes: $$\sum_{i=1}^N \text{cost}(a_i, x)$$

where: $$\text{cost}(a_i, x)=\min\left( |(a_i \bmod M) - x|,\ M - |(a_i \bmod M) - x| \right)$$

This means:

* You have points on a **cycle of length M**
* You want to choose a single point ( `x` ) minimizing **sum of circular distances**.

This is exactly the **1-median on a circle problem**.

</details>

<details>

<summary><strong>⭐ Key trick: change median in N-length circle → median in 2N-length array</strong></summary>

Sort all remainders:

```
r[1], r[2], ..., r[N]
```

Duplicate them by adding `M`:

```
b[i] = r[i]
b[N+i] = r[i] + M
```

Now any circular window of length ≤ M corresponds to an interval in `b`.

We want a window of size **N** in `b` and choose the best `x` inside that window.

For each window of indices `[i, i+N-1]`:

* median is at position `mid = i + N//2`
* Cost is the sum of absolute distances to this median, computed linearly.

We do prefix sums to compute window cost fast.

</details>

<details>

<summary>How to compute cost if we can find median in 2N-length array</summary>

For a window (\[L, R]) of size (N) in the doubled array (b), we choose the **median index**:

```
mid = (L + R) / 2
median = b[mid]
```

The cost is: $$\sum_{i=L}^{R} | b[i] - \text{median} |$$

We need to compute that **fast**, not in O(N) per window.

*   **⭐ Split into left side and right side**

    We break the window into:

    * **Left side**: $$L \to mid-1$$
    * **Right side**: $$mid+1 \to R$$

    Because:

    * On the left side: $$b[i] \le \text{median}$$
    * On the right side: $$b[i] \ge \text{median}$$

    So we can remove absolute value:

    Left side cost: $$\sum_{i=L}^{mid-1} (\text{median} - b[i])$$

    Right side cost: $$\sum_{i=mid+1}^{R} (b[i] - \text{median})$$
*   **🧮 How to use Prefix sums**

    We have prefix sums:

    ```
    ps[k] = b[0] + b[1] + ... + b[k-1]
    ```

    So:

    ```
    sum(b[L..R]) = ps[R+1] - ps[L]
    ```
*   ✨ **Left Side Explanation**

    Left side indices: L to mid-1

    Number of elements:

    ```
    leftCount = mid - L
    ```

    Sum of those values:

    ```
    leftSum = ps[mid] - ps[L]
    ```

    Cost formula: $$\sum (\text{median} - b[i]) = \text{median} * \text{leftCount} - \text{leftSum}$$&#x20;

    Code:

    ```java
    long leftSum = ps[mid] - ps[L];
    long costLeft = median * (mid - L) - leftSum;
    ```

    This computes the entire left-side cost in **O(1)**.
*   ✨ **Right Side Explanation**

    Right side indices: mid+1 to R

    Number of elements:

    ```
    rightCount = R - mid
    ```

    Sum of those values:

    ```
    rightSum = ps[R+1] - ps[mid+1]
    ```

    Cost formula: $$\sum (b[i] - \text{median}) = \text{rightSum} - \text{median} * \text{rightCount}$$

    Code:

    ```java
    long rightSum = ps[R + 1] - ps[mid + 1];
    long costRight = rightSum - median * (R - mid);
    ```

    This computes the entire right-side cost also in **O(1)**.
*   **Final Cost**

    ```
    cost = costLeft + costRight
    ```

    Which equals: $$\sum_{i=L}^{R} |b[i] - \text{median}|$$

    This is exactly the total circular distance for this window.

</details>

<details>

<summary>⏰ <strong>Time Complexity</strong></summary>

* Sorting: `O(N log N)`
* Sliding window: `O(N)`

</details>

<pre class="language-java"><code class="lang-java"><strong>import java.io.*;
</strong>import java.util.*;

public class prob2_silver_jan25 {
    static int MAXN = 200005;
    static int[] r = new int[MAXN];
    static int[] b = new int[2*MAXN];
    static long[] ps = new long[2*MAXN+1]; //presum

    public static void main(String[] args) throws Exception {
        Kattio io = new Kattio(System.in, System.out);

        int T = io.nextInt();
        StringBuilder sb = new StringBuilder();

        while (T-- > 0) {
            int N = io.nextInt();
            int M = io.nextInt();

          	// Arrays.fill(r, 0, N, 0);
            // Arrays.fill(b, 0, 2*N, 0);
            // Arrays.fill(ps, 0, 2*N+1, 0L);
            ps[0] = 0;

            for (int i = 0; i &#x3C; N; i++) {
                r[i] = io.nextInt() % M;
            }
            Arrays.sort(r, 0, N);

            for (int i = 0; i &#x3C; N; i++) {
                b[i] = r[i];
                b[i + N] = r[i] + M;
            }

            for (int i = 0; i &#x3C; 2 * N; i++) {
                ps[i + 1] = ps[i] + b[i];
            }

            long ans = Long.MAX_VALUE;

            for (int start = 0; start &#x3C; N; start++) {
                int L = start;
                int R = start + N - 1;
                int mid = (L + R) / 2;

                int median = b[mid];

                // left side
                long leftSum = ps[mid] - ps[L];
                long costLeft = median * (mid - L) - leftSum;

                // right side
                long rightSum = ps[R + 1] - ps[mid + 1];
                long costRight = rightSum - median * (R - mid);

                ans = Math.min(ans, costLeft + costRight);
            }

            sb.append(ans).append('\\n');
        }

        io.print(sb.toString());
        io.flush();
    }

    static class Kattio extends PrintWriter {
        private BufferedReader r;
        private StringTokenizer st;

        public Kattio() {
            this(System.in, System.out);
        }
        public Kattio(InputStream i, OutputStream o) {
            super(o);
            r = new BufferedReader(new InputStreamReader(i));
        }
        public Kattio(String problemName) throws IOException {
            super(problemName + ".out");
            r = new BufferedReader(new FileReader(problemName + ".in"));
        }
        public String next() {
            try {
                while (st == null || !st.hasMoreTokens())
                    st = new StringTokenizer(r.readLine());
                return st.nextToken();
            } catch (Exception e) {
            }
            return null;
        }
        public int nextInt() { return Integer.parseInt(next()); }
        public long nextLong() { return Long.parseLong(next()); }
        public double nextDouble() { return Double.parseDouble(next()); }
    }
}
</code></pre>

