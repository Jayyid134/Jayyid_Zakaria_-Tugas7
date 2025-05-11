# Program Multi-thread pada Simulator

## Program Python
```python

import threading
import time
import random

shared_memory = {'x': 0}  # Representasi dari memori utama
lock = threading.Lock()


# Simulasi delay baca/tulis
def delay():
    time.sleep(random.uniform(0.01, 0.05))


# Tanpa koherensi (cache lokal, tidak disinkronkan)
def thread_no_coherence(thread_id, local_cache):
    for _ in range(5):
        delay()
        local_cache['x'] = shared_memory['x']  # ambil dari memori utama
        print(f"[Thread-{thread_id}][NO-KOHR] Read x={local_cache['x']}")

        local_cache['x'] += 1  # update lokal
        delay()

        shared_memory['x'] = local_cache['x']  # tulis kembali ke memori
        print(f"[Thread-{thread_id}][NO-KOHR] Wrote x={local_cache['x']}")


# Dengan koherensi (pakai lock untuk sinkronisasi)
def thread_with_coherence(thread_id):
    for _ in range(5):
        delay()
        with lock:
            local_x = shared_memory['x']
            print(f"[Thread-{thread_id}][KOHR] Read x={local_x}")

            local_x += 1
            delay()

            shared_memory['x'] = local_x
            print(f"[Thread-{thread_id}][KOHR] Wrote x={local_x}")


def run_simulation(use_coherence=True):
    global shared_memory
    shared_memory = {'x': 0}  # Reset memory

    threads = []
    start = time.time()

    for i in range(2):
        if use_coherence:
            t = threading.Thread(target=thread_with_coherence, args=(i,))
        else:
            local_cache = {'x': 0}
            t = threading.Thread(target=thread_no_coherence, args=(i, local_cache))
        threads.append(t)
        t.start()

    for t in threads:
        t.join()

    end = time.time()
    print(f"\nFinal value of x = {shared_memory['x']}")
    print(f"Execution Time: {end - start:.4f} seconds\n")


# ==== RUN SIMULASI ====
print("=== TANPA PROTOKOL KOHERENSI ===")
run_simulation(use_coherence=False)

print("\n=== DENGAN PROTOKOL KOHERENSI (LOCK) ===")
run_simulation(use_coherence=True)
```

### Hasil Tanpa Protokol Koherensi

- [Thread-1][NO-KOHR] Read x=0
- [Thread-0][NO-KOHR] Read x=0
- [Thread-0][NO-KOHR] Wrote x=1
- [Thread-1][NO-KOHR] Wrote x=1
- [Thread-1][NO-KOHR] Read x=1
- [Thread-0][NO-KOHR] Read x=1
- [Thread-1][NO-KOHR] Wrote x=2
- [Thread-1][NO-KOHR] Read x=2
- [Thread-0][NO-KOHR] Wrote x=2
- [Thread-0][NO-KOHR] Read x=2
- [Thread-1][NO-KOHR] Wrote x=3
- [Thread-1][NO-KOHR] Read x=3
- [Thread-0][NO-KOHR] Wrote x=3
- [Thread-1][NO-KOHR] Wrote x=4
- [Thread-1][NO-KOHR] Read x=4
- [Thread-0][NO-KOHR] Read x=4
- [Thread-0][NO-KOHR] Wrote x=5
- [Thread-1][NO-KOHR] Wrote x=5
- [Thread-0][NO-KOHR] Read x=5
- [Thread-0][NO-KOHR] Wrote x=6

Final value of x = 6
Execution Time: 0.3307 seconds

### Hasil Dengan Protokol Koherensi(LOCK)

- [Thread-1][KOHR] Read x=0
- [Thread-1][KOHR] Wrote x=1
- [Thread-0][KOHR] Read x=1
- [Thread-0][KOHR] Wrote x=2
- [Thread-1][KOHR] Read x=2
- [Thread-1][KOHR] Wrote x=3
- [Thread-0][KOHR] Read x=3
- [Thread-0][KOHR] Wrote x=4
- [Thread-0][KOHR] Read x=4
- [Thread-0][KOHR] Wrote x=5
- [Thread-1][KOHR] Read x=5
- [Thread-1][KOHR] Wrote x=6
- [Thread-0][KOHR] Read x=6
- [Thread-0][KOHR] Wrote x=7
- [Thread-1][KOHR] Read x=7
- [Thread-1][KOHR] Wrote x=8
- [Thread-1][KOHR] Read x=8
- [Thread-1][KOHR] Wrote x=9
- [Thread-0][KOHR] Read x=9
- [Thread-0][KOHR] Wrote x=10

Final value of x = 10
Execution Time: 0.3383 seconds
