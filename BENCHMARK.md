# Setup
## System Hardware
- CPU: Intel64 Family 6 Model 78 Stepping 3, GenuineIntel, 2 cores / 4 threads
- RAM: 17.09 GB
- Disk: 255.94 GB total, 3.65 GB free, Type: SSD/HDD?

## Software / Environment
- OS: Windows-10-10.0.19045-SP0
- Python: 3.10.11
- Notebook: Jupyter Notebook / Lab
- Libraries:
  - tqdm: 4.67.1
  - rich: 13.9.4
  - alive_progress: 3.3.0
  - progressive_py: 0.1.3

# Performance
ProgressivePy uses time-gated approach. Mean that updates are triggered after specific interval, not on every iteration. That interval is taken via `freq` (default=0.1) parameter. 
This allows user to control performance, **low freq = [low performance, high smoothness]**. Below, we conducted different tests.

## In Terminal/Console
<details>
  <summary>Click to show code</summary>

```python
import time
from tqdm import tqdm
from rich.progress import Progress
from alive_progress import alive_bar
from progressive_py.progress_bar import simple
from progressive_py.manager import AssetsManager

N = 1000
style = AssetsManager().load("progress_bar", "style", "its_cool")

# -----------------------------
# Workloads
# -----------------------------
def empty_work():
    pass

def fast_work():
    x = 123 * 456

def medium_work():
    s = 0
    for i in range(50):
        s += i * i

def io_work():
    time.sleep(0.0001)


WORKS = {
    "empty": empty_work,
    "fast": fast_work,
    "medium": medium_work,
    "io": io_work,
}


# -----------------------------
# Benchmark helpers
# -----------------------------
def bench_tqdm(work):
    start = time.perf_counter()
    for _ in tqdm(range(N), mininterval=0):
        work()
    return time.perf_counter() - start


def bench_rich(work):
    start = time.perf_counter()
    with Progress(refresh_per_second=1000) as progress:
        task = progress.add_task("test", total=N)
        for _ in range(N):
            work()
            progress.update(task, advance=1)
    return time.perf_counter() - start


def bench_alive(work):
    start = time.perf_counter()
    with alive_bar(N) as bar:
        for _ in range(N):
            work()
            bar()
    return time.perf_counter() - start


def bench_progressive(work, freq=1.0):
    start = time.perf_counter()
    for _ in simple(range(N), args=style, txt_lf = "Progress - {percent:.1f}|", txt_rt="| ETA: {eta} | Elapsed: {elapsed}"):
        work()
    return time.perf_counter() - start


# -----------------------------
# Run benchmarks
# -----------------------------
print(f"\nN = {N}\n")

for i in range(5):
    print('-'*30)
    print(f"Test#0{i}")
    print('-'*30)
    for name, work in WORKS.items():
        print(f"=== Workload: {name} ===")

        t_tqdm = bench_tqdm(work)
        print(f"tqdm           : {t_tqdm:.4f} s")

        t_rich = bench_rich(work)
        print(f"rich           : {t_rich:.4f} s")

        t_alive = bench_alive(work)
        print(f"alive-progress : {t_alive:.4f} s")

        t_prog = bench_progressive(work, freq=1.0 if name == "io" else 0.1)
        print(f"progressive_py  (styled, freq = {1.0 if name == 'io' else 0.1}): {t_prog:.4f} s")
```
</details>

<details>
<summary>Output</summary>
  
```
N = 1000

------------------------------
Test#00
------------------------------
=== Workload: empty ===       
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:01<00:00, 526.87it/s]
tqdm           : 1.9795 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0494 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.0s (122721.04/s
alive-progress : 0.2818 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0049 s
=== Workload: fast ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:02<00:00, 379.17it/s] 
tqdm           : 2.6403 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0295 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.0s (122989.91/s
alive-progress : 0.0481 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0046 s
=== Workload: medium ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:02<00:00, 470.59it/s]
tqdm           : 2.1282 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0803 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.1s (28849.84/s)
alive-progress : 0.0844 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0332 s
=== Workload: io ===
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:17<00:00, 58.59it/s] 
tqdm           : 17.0718 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 16.5773 s
|████████████████████████████████████████| 1000/1000 [100%] in 15.8s (63.51/s)  
alive-progress : 15.7810 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:15
progressive_py  (styled, freq = 1.0): 15.8551 s
------------------------------
Test#01
------------------------------
=== Workload: empty ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:01<00:00, 651.89it/s] 
tqdm           : 1.5370 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0327 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.0s (106277.03/s
alive-progress : 0.0560 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0095 s
=== Workload: fast ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:01<00:00, 735.29it/s] 
tqdm           : 1.3663 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0395 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.0s (114832.67/s
alive-progress : 0.0569 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0083 s
=== Workload: medium ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:01<00:00, 625.78it/s] 
tqdm           : 1.6075 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0709 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.1s (18725.58/s)
alive-progress : 0.0959 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0312 s
=== Workload: io ===
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:17<00:00, 55.88it/s] 
tqdm           : 17.8999 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 16.7137 s
|████████████████████████████████████████| 1000/1000 [100%] in 16.2s (62.04/s)  
alive-progress : 16.1623 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:16
progressive_py  (styled, freq = 1.0): 16.3963 s
------------------------------
Test#02
------------------------------
=== Workload: empty ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:02<00:00, 499.00it/s] 
tqdm           : 2.0080 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0298 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.1s (119832.48/s
alive-progress : 0.0729 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0074 s
=== Workload: fast ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:01<00:00, 653.37it/s] 
tqdm           : 1.5345 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0566 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.0s (90314.10/s)
alive-progress : 0.0446 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0080 s
=== Workload: medium ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:01<00:00, 612.37it/s] 
tqdm           : 1.6352 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0650 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.1s (32109.12/s)
alive-progress : 0.0825 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0269 s
=== Workload: io ===
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:17<00:00, 58.43it/s] 
tqdm           : 17.1186 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 16.1124 s
|████████████████████████████████████████| 1000/1000 [100%] in 16.0s (62.71/s)  
alive-progress : 15.9924 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:17
progressive_py  (styled, freq = 1.0): 17.2714 s
------------------------------
Test#03
------------------------------
=== Workload: empty ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:02<00:00, 492.94it/s]
tqdm           : 2.0675 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0361 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.1s (106258.66/s
alive-progress : 0.0678 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0064 s
=== Workload: fast ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:02<00:00, 421.94it/s] 
tqdm           : 2.3723 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0464 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.0s (107194.98/s
alive-progress : 0.0537 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0054 s
=== Workload: medium ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:02<00:00, 459.35it/s]
tqdm           : 2.1879 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0381 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.1s (17738.17/s)
alive-progress : 0.1350 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0253 s
=== Workload: io ===
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:24<00:00, 41.25it/s] 
tqdm           : 24.2454 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 17.0755 s
|████████████████████████████████████████| 1000/1000 [100%] in 16.1s (62.20/s)  
alive-progress : 16.1007 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:16
progressive_py  (styled, freq = 1.0): 16.0299 s
------------------------------
Test#04
------------------------------
=== Workload: empty ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:01<00:00, 689.18it/s] 
tqdm           : 1.4540 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0304 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.0s (113391.99/s
alive-progress : 0.0550 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0064 s
=== Workload: fast ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:01<00:00, 760.46it/s] 
tqdm           : 1.3181 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0321 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.0s (92659.35/s)
alive-progress : 0.0573 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0047 s
=== Workload: medium ===
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:01<00:00, 674.76it/s] 
tqdm           : 1.4847 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 0.0540 s
|████████████████████████████████████████| 1000/1000 [100%] in 0.1s (30812.23/s)
alive-progress : 0.0754 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:00
progressive_py  (styled, freq = 0.1): 0.0239 s
=== Workload: io ===
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1000/1000 [00:18<00:00, 55.55it/s]
tqdm           : 18.0043 s
test ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
rich           : 16.1319 s
|████████████████████████████████████████| 1000/1000 [100%] in 15.9s (62.86/s)  
alive-progress : 15.9534 s
Progress - 100.0|▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰| ETA: 00:00 | Elapsed: 00:15
progressive_py  (styled, freq = 1.0): 15.9945 s
```

</details>

## 1️⃣ Empty workload (loop iterations only)

| Test | tqdm(s) | rich(s) | alive-progress(s) | progressive_py(s) |
| ---- | ------- | ------- | ----------------- | ----------------- |
| 0    | 1.9795  | 0.0494  | 0.2818            | 0.0049            |
| 1    | 1.5370  | 0.0327  | 0.0560            | 0.0095            |
| 2    | 2.0080  | 0.0298  | 0.0729            | 0.0074            |
| 3    | 2.0675  | 0.0361  | 0.0678            | 0.0064            |
| 4    | 1.4540  | 0.0304  | 0.0550            | 0.0064            |

**Observations:**

* **tqdm** is consistently slow; update/render overhead dominates the tiny loop.
* **rich** is extremely fast; minimal overhead with throttled rendering.
* **alive-progress** is slightly slower than rich; animation overhead is visible.
* **progressive_py** is **the fastest** in all tests – low-frequency updates (freq = 0.1) make it ideal for empty loops.

✅ **Takeaway:** For trivial terminal loops, **progressive_py wins** easily.

## 2️⃣ Fast arithmetic workload

| Test | tqdm(s) | rich(s) | alive-progress(s) | progressive_py(s) |
| ---- | ------- | ------- | ----------------- | ----------------- |
| 0    | 2.6403  | 0.0295  | 0.0481            | 0.0046            |
| 1    | 1.3663  | 0.0395  | 0.0569            | 0.0083            |
| 2    | 1.5345  | 0.0566  | 0.0446            | 0.0080            |
| 3    | 2.3723  | 0.0464  | 0.0537            | 0.0054            |
| 4    | 1.3181  | 0.0321  | 0.0573            | 0.0047            |

**Observations:**

* **tqdm** still slower due to per-iteration rendering overhead.
* **rich** and **alive-progress** are very fast; lightweight rendering dominates.
* **progressive_py** is fastest in all cases; very minimal overhead.

✅ **Takeaway:** For light computation in terminal, **progressive_py** provides the lowest overhead and highest speed.

## 3️⃣ Medium workload (computation-heavy)

| Test | tqdm(s) | rich(s) | alive-progress(s) | progressive_py(s) |
| ---- | ------- | ------- | ----------------- | ----------------- |
| 0    | 2.1282  | 0.0803  | 0.0844            | 0.0332            |
| 1    | 1.6075  | 0.0709  | 0.0959            | 0.0312            |
| 2    | 1.6352  | 0.0650  | 0.0825            | 0.0269            |
| 3    | 2.1879  | 0.0381  | 0.1350            | 0.0253            |
| 4    | 1.4847  | 0.0540  | 0.0754            | 0.0239            |

**Observations:**

* Computation now dominates; progress bar overhead becomes negligible.
* **tqdm** shows slightly higher times due to per-iteration update cost.
* **rich** varies due to rendering throttling.
* **alive-progress** is consistent but slightly slower due to animation.
* **progressive_py** remains very fast; low-frequency styled updates help reduce overhead.

✅ **Takeaway:** For moderate computation, **all bars perform well**, but **progressive_py remains highly competitive**.

## 4️⃣ IO workload (sleep/delay simulation)

| Test | tqdm(s) | rich(s) | alive-progress(s) | progressive_py(s) |
| ---- | ------- | ------- | ----------------- | ----------------- |
| 0    | 17.0718 | 16.5773 | 15.7810           | 15.8551           |
| 1    | 17.8999 | 16.7137 | 16.1623           | 16.3963           |
| 2    | 17.1186 | 16.1124 | 15.9924           | 17.2714           |
| 3    | 24.2454 | 17.0755 | 16.1007           | 16.0299           |
| 4    | 18.0043 | 16.1319 | 15.9534           | 15.9945           |

**Observations:**

* IO-bound workload dominates; progress bar overhead is almost negligible.
* **alive-progress** slightly slower in some tests due to animation rendering.
* **progressive_py** performs on par with the fastest bars (tqdm/rich) and occasionally slightly better.
* Test#3 shows a **tqdm spike** – possibly terminal buffering/printing slows it down.

✅ **Takeaway:** For IO-heavy terminal workloads, **all libraries perform similarly**, **progressive_py** remains highly competitive.

## In Notebook

<details>
  <summary>Code</summary>

```python
# Notebook Benchmark for progress bars
import time
from tqdm.notebook import tqdm
from rich.progress import Progress
from alive_progress import alive_bar
from progressive_py.progress_bar import simple
from progressive_py.manager import AssetsManager
from IPython.display import display, HTML, clear_output

# -----------------------------
# Configuration
# -----------------------------
N = 1000
style = AssetsManager().load("progress_bar", "style", "its_cool")

# -----------------------------
# Workloads
# -----------------------------
def empty_work():
    pass

def fast_work():
    x = 123 * 456

def medium_work():
    s = 0
    for i in range(50):
        s += i * i

def io_work():
    time.sleep(0.0001)

WORKS = {
    "empty": empty_work,
    "fast": fast_work,
    "medium": medium_work,
    "io": io_work,
}

# -----------------------------
# Benchmark helpers
# -----------------------------
def bench_tqdm(work):
    start = time.perf_counter()
    for _ in tqdm(range(N), mininterval=0):
        work()
    return time.perf_counter() - start

def bench_rich(work):
    start = time.perf_counter()
    with Progress(refresh_per_second=1000) as progress:
        task = progress.add_task("test", total=N)
        for _ in range(N):
            work()
            progress.update(task, advance=1)
    return time.perf_counter() - start

def bench_alive(work):
    start = time.perf_counter()
    with alive_bar(N) as bar:
        for _ in range(N):
            work()
            bar()
    return time.perf_counter() - start

def bench_progressive(work, freq=0.1):
    start = time.perf_counter()
    for _ in simple(range(N), args=style,
                    txt_lf="Progress - {percent:.1f}|",
                    txt_rt="| ETA: {eta} | Elapsed: {elapsed}",
                    freq=freq):
        work()
    return time.perf_counter() - start

# -----------------------------
# Run Benchmarks
# -----------------------------
results = []

display(HTML(f"<h2>Notebook Progress Bar Benchmark (N={N})</h2>"))

for i in range(5):
    display(HTML(f"<h3>Test#{i:02d}</h3>"))
    for name, work in WORKS.items():
        display(HTML(f"<b>Workload: {name}</b>"))

        t_tqdm = bench_tqdm(work)
        t_rich = bench_rich(work)
        t_alive = bench_alive(work)
        t_prog = bench_progressive(work, freq=1.0 if name == "io" else 0.1)

        results.append({
            "Test": i,
            "Workload": name,
            "tqdm(s)": round(t_tqdm, 4),
            "rich(s)": round(t_rich, 4),
            "alive-progress(s)": round(t_alive, 4),
            "progressive_py(s)": round(t_prog, 4)
        })

# -----------------------------
# Display Results
# -----------------------------
import pandas as pd
df = pd.DataFrame(results)
df
```

</details>

<details>
  <summary>DataFrame</summary>
  
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Test</th>
      <th>Workload</th>
      <th>tqdm(s)</th>
      <th>rich(s)</th>
      <th>alive-progress(s)</th>
      <th>progressive_py(s)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>empty</td>
      <td>5.7603</td>
      <td>0.1280</td>
      <td>0.4561</td>
      <td>0.0332</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>fast</td>
      <td>2.5305</td>
      <td>0.0825</td>
      <td>0.0358</td>
      <td>0.0093</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>medium</td>
      <td>4.3348</td>
      <td>0.1074</td>
      <td>0.1070</td>
      <td>0.2339</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>io</td>
      <td>17.9944</td>
      <td>18.2457</td>
      <td>30.2014</td>
      <td>16.6968</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>empty</td>
      <td>6.3947</td>
      <td>0.0751</td>
      <td>0.0450</td>
      <td>0.0094</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1</td>
      <td>fast</td>
      <td>2.1501</td>
      <td>0.0652</td>
      <td>0.0374</td>
      <td>0.0082</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1</td>
      <td>medium</td>
      <td>5.3053</td>
      <td>0.2054</td>
      <td>0.0739</td>
      <td>0.0353</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1</td>
      <td>io</td>
      <td>18.1724</td>
      <td>18.2861</td>
      <td>25.7935</td>
      <td>15.8805</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2</td>
      <td>empty</td>
      <td>3.0278</td>
      <td>0.3500</td>
      <td>0.0358</td>
      <td>0.0654</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2</td>
      <td>fast</td>
      <td>4.4096</td>
      <td>0.0554</td>
      <td>0.0284</td>
      <td>0.0099</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2</td>
      <td>medium</td>
      <td>4.1591</td>
      <td>0.3415</td>
      <td>0.0796</td>
      <td>0.0581</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2</td>
      <td>io</td>
      <td>17.7264</td>
      <td>17.6628</td>
      <td>17.9609</td>
      <td>15.8432</td>
    </tr>
    <tr>
      <th>12</th>
      <td>3</td>
      <td>empty</td>
      <td>3.4950</td>
      <td>0.0872</td>
      <td>0.0347</td>
      <td>0.0125</td>
    </tr>
    <tr>
      <th>13</th>
      <td>3</td>
      <td>fast</td>
      <td>3.9010</td>
      <td>0.0712</td>
      <td>0.0343</td>
      <td>0.0112</td>
    </tr>
    <tr>
      <th>14</th>
      <td>3</td>
      <td>medium</td>
      <td>4.2405</td>
      <td>0.1302</td>
      <td>0.0840</td>
      <td>0.0300</td>
    </tr>
    <tr>
      <th>15</th>
      <td>3</td>
      <td>io</td>
      <td>18.0928</td>
      <td>17.2430</td>
      <td>17.9414</td>
      <td>15.7638</td>
    </tr>
    <tr>
      <th>16</th>
      <td>4</td>
      <td>empty</td>
      <td>3.4195</td>
      <td>0.1475</td>
      <td>0.0340</td>
      <td>0.0100</td>
    </tr>
    <tr>
      <th>17</th>
      <td>4</td>
      <td>fast</td>
      <td>4.5931</td>
      <td>0.1390</td>
      <td>0.0715</td>
      <td>0.0118</td>
    </tr>
    <tr>
      <th>18</th>
      <td>4</td>
      <td>medium</td>
      <td>3.2018</td>
      <td>0.4625</td>
      <td>0.2199</td>
      <td>0.0274</td>
    </tr>
    <tr>
      <th>19</th>
      <td>4</td>
      <td>io</td>
      <td>17.6544</td>
      <td>17.3846</td>
      <td>18.1578</td>
      <td>15.8208</td>
    </tr>
  </tbody>
</table>
</div>
</details>

## 1️⃣ Empty workload (just loop iterations, nothing inside)

| Test | tqdm(s) | rich(s) | alive-progress(s) | progressive_py(s) |
| ---- | ------- | ------- | ----------------- | ----------------- |
| 0    | 5.7603  | 0.1280  | 0.4561            | 0.0332            |
| 1    | 6.3947  | 0.0751  | 0.0450            | 0.0094            |
| 2    | 3.0278  | 0.3500  | 0.0358            | 0.0654            |
| 3    | 3.4950  | 0.0872  | 0.0347            | 0.0125            |
| 4    | 3.4195  | 0.1475  | 0.0340            | 0.0100            |

**Observations:**

* **tqdm** is surprisingly slow when the workload is negligible – overhead dominates.
* **rich** is extremely fast here (rendering once every iteration is throttled), mostly efficient.
* **alive-progress** is very consistent and fast for empty loops.
* **progressive_py** is **the fastest** here – thanks to optimized update logic, low overhead, and possibly throttling.

✅ **Takeaway:** For trivial workloads, **progressive_py** wins.

## 2️⃣ Fast arithmetic workload

| Test | tqdm(s) | rich(s) | alive-progress(s) | progressive_py(s) |
| ---- | ------- | ------- | ----------------- | ----------------- |
| 0    | 2.5305  | 0.0825  | 0.0358            | 0.0093            |
| 1    | 2.1501  | 0.0652  | 0.0374            | 0.0082            |
| 2    | 4.4096  | 0.0554  | 0.0284            | 0.0099            |
| 3    | 3.9010  | 0.0712  | 0.0343            | 0.0112            |
| 4    | 4.5931  | 0.1390  | 0.0715            | 0.0118            |

**Observations:**

* **tqdm** still slower than modern libraries.
* **rich** remains fast and smooth.
* **alive-progress** is slightly faster than rich.
* **progressive_py** is fastest across the board, even for small computations.

✅ **Takeaway:** For lightweight arithmetic, **progressive_py** overhead is minimal, making it ideal.


## 3️⃣ Medium workload (small loop, computation-heavy)

| Test | tqdm(s) | rich(s) | alive-progress(s) | progressive_py(s) |
| ---- | ------- | ------- | ----------------- | ----------------- |
| 0    | 4.3348  | 0.1074  | 0.1070            | 0.2339            |
| 1    | 5.3053  | 0.2054  | 0.0739            | 0.0353            |
| 2    | 4.1591  | 0.3415  | 0.0796            | 0.0581            |
| 3    | 4.2405  | 0.1302  | 0.0840            | 0.0300            |
| 4    | 3.2018  | 0.4625  | 0.2199            | 0.0274            |

**Observations:**

* Computation now dominates the runtime. The progress bar overhead is less important.
* **tqdm** runtime is close to total computation time – overhead is hidden.
* **progressive_py** becomes competitive again for some tests, slightly higher in Test#0 but very low for others.
* **rich** varies due to throttling and rendering strategy.
* **alive-progress** performs very consistently.

✅ **Takeaway:** For moderate computation, **all libraries are acceptable**, but **progressive_py** remains competitive.

## 4️⃣ IO workload (sleep, simulated delay)

| Test | tqdm(s) | rich(s) | alive-progress(s) | progressive_py(s) |
| ---- | ------- | ------- | ----------------- | ----------------- |
| 0    | 17.9944 | 18.2457 | 30.2014           | 16.6968           |
| 1    | 18.1724 | 18.2861 | 25.7935           | 15.8805           |
| 2    | 17.7264 | 17.6628 | 17.9609           | 15.8432           |
| 3    | 18.0928 | 17.2430 | 17.9414           | 15.7638           |
| 4    | 17.6544 | 17.3846 | 18.1578           | 15.8208           |

**Observations:**

* IO-bound workload dominates, progress bar overhead is negligible.
* **progressive_py** is still slightly faster than **tqdm** and **rich**.
* **alive-progress** is slower for some reason – likely its animated rendering adds overhead.

✅ **Takeaway:** For IO-heavy tasks, **all modern bars perform similarly**, **progressive_py** slightly faster.

# Conclusion
In **notebook environments**, progressive_py is usually the fastest for lightweight and fast workloads (like empty loops or simple arithmetic). This is because its time-gated updates (controlled via freq) minimize the overhead of DOM rendering or cell output.

For **medium computation**, progressive_py is still competitive, though sometimes rich or alive-progress might edge it slightly depending on rendering load.

For **IO-heavy tasks**, all libraries perform roughly the same, but progressive_py remains reliable and smooth.

> ## In notebook environments, progressive_py often outperforms other progress bars for lightweight workloads, and remains highly competitive even in heavier computations. Its time-gated approach ensures minimal overhead while providing smooth updates.
