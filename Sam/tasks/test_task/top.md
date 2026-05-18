---
Rank: 2
BashTime: -1
Skills: local_env
GPU: no
Slurm: off
---

# Benchmark Sorting Algorithms

## Context
Benchmark three manually implemented sorting algorithms: bubble sort, merge sort, and quicksort. The benchmark should run on CPU only and use C++17 for performance. Do not use built-in sorting functions to perform the benchmarked sorts. Built-in helpers may be avoided entirely; correctness can be checked with a manual sortedness check.

Arrays should contain reproducible pseudo-random integers generated with a fixed seed. Test array sizes are exactly 1000, 10000, and 100000. Each algorithm must receive the same input values for a given array size and repeat so the timing comparison is fair.

Bubble sort at size 100000 may take a long time, so compile with optimization such as `-O3` and allow long bash execution. No GPU or SLURM is needed.

Expected outputs:
- `benchmark_results.csv`: detailed timing results
- `summary.txt`: fastest algorithm for each tested array size
- Source code and a build/run path sufficient to reproduce the results

Use the `local_env` skill if a C++17 compiler is not already available; create a local micromamba environment with compilers rather than installing into the system environment.

## Todo
1. Create a C++17 benchmark program, e.g. `sorting_benchmark.cpp`.
2. Manually implement bubble sort, merge sort, and quicksort in the program. Do not call `std::sort`, `qsort`, or another library sort for the benchmarked algorithms.
3. Generate reproducible random integer arrays for sizes `1000`, `10000`, and `100000` using a fixed seed.
4. For each size and each algorithm, run exactly one timed repeat. Give each algorithm a fresh copy of the same random base array for that size.
5. Time only the sorting work using a high-resolution monotonic clock such as `std::chrono::steady_clock` or `std::chrono::high_resolution_clock`.
6. Verify after every run that the resulting array is sorted in nondecreasing order.
7. Write `benchmark_results.csv` with exactly this header:
   `size,algorithm,repeat,time_seconds,sorted_correctly`
   Use algorithm names exactly `bubble_sort`, `merge_sort`, and `quicksort`; use repeat value `1`; write `sorted_correctly` as `true` or `false`.
8. Determine the fastest algorithm at each size using the measured `time_seconds` values from `benchmark_results.csv`.
9. Write `summary.txt` clearly stating the fastest algorithm for each array size, including its measured time in seconds.
10. Compile the benchmark with optimization, e.g. `g++ -O3 -std=c++17 sorting_benchmark.cpp -o sorting_benchmark`, and run it to generate the outputs.
11. Optionally write a short `README.md` documenting how to build and run the benchmark.

## Expect
- `sorting_benchmark.cpp` exists and contains manual implementations of bubble sort, merge sort, and quicksort.
- The compiled executable `sorting_benchmark` exists after the build step.
- `benchmark_results.csv` exists.
- The first line of `benchmark_results.csv` is exactly `size,algorithm,repeat,time_seconds,sorted_correctly`.
- `benchmark_results.csv` contains exactly 9 data rows: 3 array sizes × 3 algorithms × 1 repeat.
- `benchmark_results.csv` contains one row for every combination of size in `{1000,10000,100000}` and algorithm in `{bubble_sort,merge_sort,quicksort}`.
- Every data row has `repeat` equal to `1`.
- Every data row has a positive numeric `time_seconds` value.
- Every data row has `sorted_correctly` equal to `true`.
- `summary.txt` exists and clearly states the fastest algorithm for each size `1000`, `10000`, and `100000`.
- The fastest algorithms reported in `summary.txt` match the minimum `time_seconds` values in `benchmark_results.csv` for each size.
- The task completes without requiring GPU or SLURM resources.
