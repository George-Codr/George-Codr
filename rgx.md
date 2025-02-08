Creating a shared object (`.so`) file from multiple C++ source files is very similar to the process for C, but there are a few differences due to name mangling and other C++-specific features. Below is a step-by-step guide on how to create a shared library from multiple C++ source files.

### Example

Let's assume you have three C++ source files: `file1.cpp`, `file2.cpp`, and `file3.cpp`.

#### Step 1: Write your C++ source files

**file1.cpp**
```cpp
#include <iostream>

void function1() {
    std::cout << "This is function1" << std::endl;
}
```

**file2.cpp**
```cpp
#include <iostream>

void function2() {
    std::cout << "This is function2" << std::endl;
}
```

**file3.cpp**
```cpp
#include <iostream>

void function3() {
    std::cout << "This is function3" << std::endl;
}
```

#### Step 2: Compile each source file into an object file

In C++, you need to use `g++` instead of `gcc` to handle C++-specific features like name mangling. Use the `-fPIC` flag to generate position-independent code, which is required for shared libraries.

```bash
g++ -fPIC -c file1.cpp -o file1.o
g++ -fPIC -c file2.cpp -o file2.o
g++ -fPIC -c file3.cpp -o file3.o
```

#### Step 3: Link the object files into a shared library

Use the `-shared` flag to create a shared object (`.so`) file.

```bash
g++ -shared file1.o file2.o file3.o -o libmylib.so
```

This will create a shared library named `libmylib.so`.

### Using the Shared Library

To use the shared library in another C++ program, you need to link against it during compilation.

#### Example Program (`main.cpp`)

```cpp
#include <iostream>

// Declare the functions from the shared library
extern "C" void function1();
extern "C" void function2();
extern "C" void function3();

int main() {
    function1();
    function2();
    function3();
    return 0;
}
```

> **Note**: The `extern "C"` keyword is used to prevent name mangling, which is necessary if you want to call C++ functions from a C-like environment or if you want to ensure that the function names remain unchanged in the shared library. If you don't use `extern "C"`, the function names will be mangled, and you may have trouble linking them correctly.

#### Compile and Link with the Shared Library

You need to tell the compiler where to find the shared library using the `-L` flag and specify the library using the `-l` flag.

```bash
g++ main.cpp -L. -lmylib -o main
```

Here:
- `-L.` tells the linker to look in the current directory for libraries.
- `-lmylib` tells the linker to link against `libmylib.so`.

#### Run the Program

Before running the program, make sure the dynamic linker can find the shared library. You can do this by setting the `LD_LIBRARY_PATH` environment variable:

```bash
export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH
./main
```

This should output:

```
This is function1
This is function2
This is function3
```

### Summary of Commands

1. **Compile source files into object files**:
   ```bash
   g++ -fPIC -c file1.cpp -o file1.o
   g++ -fPIC -c file2.cpp -o file2.o
   g++ -fPIC -c file3.cpp -o file3.o
   ```

2. **Create the shared library**:
   ```bash
   g++ -shared file1.o file2.o file3.o -o libmylib.so
   ```

3. **Compile and link the main program**:
   ```bash
   g++ main.cpp -L. -lmylib -o main
   ```

4. **Run the program**:
   ```bash
   export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH
   ./main
   ```

### Important Notes

1. **Name Mangling**: In C++, function names are "mangled" to include information about the function's parameters and return type. This allows function overloading but can cause issues when linking. To avoid this, you can use `extern "C"` around function declarations in your header files or source files if you want to prevent name mangling.

   Example of using `extern "C"` in a header file:

   ```cpp
   #ifdef __cplusplus
   extern "C" {
   #endif

   void function1();
   void function2();
   void function3();

   #ifdef __cplusplus
   }
   #endif
   ```

2. **C++ Standard Library**: If your shared library uses parts of the C++ standard library (e.g., `std::string`, `std::vector`), you don't need to do anything special because the C++ runtime is linked automatically. However, ensure that the program using the shared library is compiled with the same C++ standard (e.g., C++11, C++17) to avoid compatibility issues.

3. **ABI Compatibility**: Ensure that both the shared library and the program using it are compiled with the same compiler version and C++ ABI (Application Binary Interface). Mixing different compiler versions or ABIs can lead to runtime errors.

### Conclusion

Creating a shared library from multiple C++ source files is straightforward once you understand the process. By compiling each source file into an object file, linking them together into a shared object, and ensuring proper linkage with `extern "C"` if needed, you can easily create reusable libraries for your C++ projects.
