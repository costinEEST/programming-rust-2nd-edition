1) Open and pick the language

Go to godbolt.org.

In the top-left of the editor pane, change Language to C (for the C file) or C++ (for the C++ file).

2) Choose a compiler + standard

In the compiler dropdown (above the right pane), pick something recent, e.g.:

C: x86-64 gcc 14 or clang 18

In Compiler options, set: -std=c17 (or -std=c23 if you want char8_t).

C++: x86-64 gcc 14 / clang 18 / msvc 19

Set: -std=c++20 (or newer).

Tip: If you just want to run (not inspect assembly), turn Optimization to -O0 to keep things simple.

3) Enable “Run” (Execute Program)

Click the Run ▶️ button above the output pane.

If you don’t see it, click the + Add tool button and choose Execute the compiled output (or “Run”). This adds a run/output panel.

Your program’s stdout will appear in the Output panel.

4) Provide program input (optional)

Click + Add new… → Stdin to create an input panel.

Type input there; the Run tool will feed it to your program.

5) Paste one of these snippets
6) (Optional) Toggle assembly off

If you don’t care about the assembly, you can close the assembly pane or just focus on the Output pane after running.

That’s it—paste, pick standard, hit Run, and you’ll see the byte sizes printed.

```c 
#include <stdio.h>
#include <wchar.h>
#include <stdint.h>
#if __STDC_VERSION__ >= 201112L
  #include <uchar.h>
#endif

int main(void) {
    printf("C sizes (bytes):\n");
    printf("_Bool            : %zu\n", sizeof(_Bool));
    printf("char             : %zu\n", sizeof(char));
    printf("signed char      : %zu\n", sizeof(signed char));
    printf("unsigned char    : %zu\n", sizeof(unsigned char));
    printf("short            : %zu\n", sizeof(short));
    printf("int              : %zu\n", sizeof(int));
    printf("long             : %zu\n", sizeof(long));
    printf("long long        : %zu\n", sizeof(long long));
    printf("wchar_t          : %zu\n", sizeof(wchar_t));
    printf("float            : %zu\n", sizeof(float));
    printf("double           : %zu\n", sizeof(double));
#if __STDC_VERSION__ >= 201112L
    printf("char16_t         : %zu\n", sizeof(char16_t));
    printf("char32_t         : %zu\n", sizeof(char32_t));
#endif
#if __STDC_VERSION__ >= 202311L
    printf("char8_t          : %zu\n", sizeof(char8_t));
#endif
    return 0;
}
```
Use `-std=c17` first. If you want the `char8_t` line, switch to `-std=c23` and pick a compiler that supports it (recent GCC/Clang).

```c++ 
#include <iostream>
#include <cstdint>

int main() {
    std::cout << "C++ sizes (bytes):\n";
    std::cout << "bool      : " << sizeof(bool)      << "\n";
    std::cout << "char      : " << sizeof(char)      << "\n";
    std::cout << "signed char   : " << sizeof(signed char)   << "\n";
    std::cout << "unsigned char : " << sizeof(unsigned char) << "\n";
    std::cout << "short     : " << sizeof(short)     << "\n";
    std::cout << "int       : " << sizeof(int)       << "\n";
    std::cout << "long      : " << sizeof(long)      << "\n";
    std::cout << "long long : " << sizeof(long long) << "\n";
    std::cout << "wchar_t   : " << sizeof(wchar_t)   << "\n";
    std::cout << "char8_t   : " << sizeof(char8_t)   << "\n";
    std::cout << "char16_t  : " << sizeof(char16_t)  << "\n";
    std::cout << "char32_t  : " << sizeof(char32_t)  << "\n";
    std::cout << "float     : " << sizeof(float)     << "\n";
    std::cout << "double    : " << sizeof(double)    << "\n";
}
```
Set Compiler options to `-std=c++20` (or newer) so `char8_t` is recognized.

```rust 
use std::mem::size_of;

fn main() {
    println!("Rust sizes (bytes):");
    println!("bool     : {}", size_of::<bool>());
    println!("u8       : {}", size_of::<u8>());
    println!("char     : {}  // Unicode scalar", size_of::<char>());
    println!("u16      : {}", size_of::<u16>());
    println!("u32      : {}", size_of::<u32>());
    println!("i16      : {}", size_of::<i16>());
    println!("i32      : {}", size_of::<i32>());
    println!("i64      : {}", size_of::<i64>());
    println!("isize    : {}  // pointer-sized", size_of::<isize>());
    println!("usize    : {}", size_of::<usize>());
    println!("f32      : {}", size_of::<f32>());
    println!("f64      : {}", size_of::<f64>());
}
```

https://chatgpt.com/c/68d003b8-a180-832d-a0d7-0d06d2ed45ed

