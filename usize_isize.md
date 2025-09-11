## Type equivalents 

| Rust Type | C Equivalent      | C++ Equivalent         | Header                                  | Description                                                                                 |
| --------- | ----------------- | ---------------------- | --------------------------------------- | ------------------------------------------------------------------------------------------- |
| `usize`   | `size_t`          | `std::size_t`          | C: `<stddef.h>`<br>C++: `<cstddef>`     | Unsigned integer, matches address space size (32-bit on 32-bit arch, 64-bit on 64-bit arch) |
| `isize`   | `ptrdiff_t`       | `std::ptrdiff_t`       | C: `<stddef.h>`<br>C++: `<cstddef>`     | Signed integer, matches address space size, used for pointer arithmetic                     |
| -         | `ssize_t` (POSIX) | `std::ssize()` (C++20) | C: `<sys/types.h>`<br>C++: `<iterator>` | Signed version of size_t, returns signed size                                               |

## Usage examples

| Language | Type             | Common Usage                    | Example Code                                               |
| -------- | ---------------- | ------------------------------- | ---------------------------------------------------------- |
| **Rust** | `usize`          | Array indexing (required)       | `let i: usize = 5;`<br>`arr[i] = value;`                   |
| **C**    | `size_t`         | Array sizes, loop counters      | `size_t len = 1024;`<br>`for (size_t i = 0; i < len; i++)` |
| **C**    | `ptrdiff_t`      | Pointer arithmetic              | `ptrdiff_t diff = ptr2 - ptr1;`                            |
| **C**    | `ssize_t`        | System calls with error returns | `ssize_t bytes = read(fd, buf, 1024);`                     |
| **C++**  | `std::size_t`    | Container sizes, indices        | `for (std::size_t i = 0; i < vec.size(); ++i)`             |
| **C++**  | `std::ptrdiff_t` | Iterator distances              | `auto dist = std::distance(begin, end);`                   |
| **C++**  | `std::ssize()`   | Get signed size (C++20)         | `auto signed_sz = std::ssize(vec);`                        |

## Platform size comparison

| Architecture | `usize`/`size_t` Size | `isize`/`ptrdiff_t` Size |
| ------------ | --------------------- | ------------------------ |
| 32-bit       | 32 bits (4 bytes)     | 32 bits (4 bytes)        |
| 64-bit       | 64 bits (8 bytes)     | 64 bits (8 bytes)        |

## Key differences from Rust

| Aspect                     | Rust                                | C/C++                                             |
| -------------------------- | ----------------------------------- | ------------------------------------------------- |
| **Index Type Enforcement** | Requires `usize` for array indexing | Accepts any integer type with implicit conversion |
| **Type Safety**            | Compiler enforces correct types     | Allows mixing signed/unsigned with warnings       |
| **Bounds Checking**        | Runtime bounds checking by default  | No automatic bounds checking                      |
| **Container Sizes**        | All use `usize`                     | STL uses `size_type` (typically `size_t`)         |
| **Negative Indices**       | Not allowed (compile error)         | Allowed but undefined behavior                    |
| **Standard Practice**      | Must use `usize` for indices        | Can use `int`, `size_t`, or any integer type      |

## Modern best practices

| Language  | Recommendation                              | Example                                  |
| --------- | ------------------------------------------- | ---------------------------------------- |
| **C**     | Use `size_t` for sizes and indices          | `for (size_t i = 0; i < n; i++)`         |
| **C++**   | Use `std::size_t` or `auto` with containers | `for (auto i = 0u; i < vec.size(); ++i)` |
| **C++**   | Prefer range-based for loops                | `for (auto& elem : vec)`                 |
| **C++20** | Use `std::ssize()` for signed sizes         | `if (std::ssize(vec) > 0)`               |