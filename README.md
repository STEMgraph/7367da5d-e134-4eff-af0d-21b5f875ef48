<!---
{
  "id": "7367da5d-e134-4eff-af0d-21b5f875ef48",
  "depends_on": ["76bd92a3-ba4d-4048-aa73-a0be33492c58", "2263c4f5-5034-48fa-beef-3d204473069c"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-12-18",
  "keywords": ["C++", "CLI11", "Command Line", "Arguments", "CMake", "FetchContent"],
  "license": "
Copyright 2025 Stephan Bökelmann

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
"
}
--->

# Using CLI11 via CMake FetchContent to Parse Command-Line Arguments in C++

> In this exercise you will learn how to integrate the `CLI11` command-line parsing library using CMake’s `FetchContent` module. Furthermore we will explore how to use CLI11 to parse positional arguments, optional switches, options with arguments, and how to automatically handle `--help` and `--version` flags.

## Introduction

CLI11 is a modern C++11 header-only library that simplifies the parsing of command-line interfaces in C++ programs. It replaces verbose and error-prone manual parsing using `getopt` or raw `argv[]` traversal with a declarative, readable, and flexible approach. The library supports positional arguments, flags, multi-option parameters, enums, and more — all while generating help and version outputs automatically.

In this exercise, you’ll use CMake’s `FetchContent` module to fetch CLI11 from GitHub and link it into your project without needing to download anything manually. This mirrors how professional developers manage dependencies in modern C++ projects.

### CLI11 Features Highlighted in This Exercise

1. **Automatic `--help` and `--version`**
2. **Positional arguments (required and optional)**
3. **Boolean flags (`--verbose`, `--debug`)**
4. **Options with values (`--output filename`)**
5. **Required options with validation and default values**

You will create a small CLI tool called `clitest` that takes inputs, controls verbosity, accepts an output file name, and prints its configuration using CLI11.

### Further Readings and Other Sources

- [CLI11 GitHub Repository](https://github.com/CLIUtils/CLI11)
- [CLI11 Full Documentation](https://cliutils.github.io/CLI11/book/)
- [CMake FetchContent Documentation](https://cmake.org/cmake/help/latest/module/FetchContent.html)
- YouTube: [Modern C++ CLI tools with CLI11](https://www.youtube.com/watch?v=GbVKX2dM8P0)

---

## Tasks

---

# **Task — Build a CLI Tool Using CLI11 via FetchContent**

### 1. Project Setup

Create this folder structure:

```

cli11_demo/
├── CMakeLists.txt
└── src/
└── main.cpp

````

### 2. Write `src/main.cpp`

```cpp
#include <CLI/CLI.hpp>
#include <iostream>
#include <string>
#include <vector>

int main(int argc, char** argv) {
    CLI::App app{"clitest: a CLI demo using CLI11"};

    // Feature 1: --version
    app.set_version_flag("--version", "1.0.0");

    // Feature 2: Positional arguments
    std::vector<std::string> input_files;
    app.add_option("files", input_files, "Input files")->required()->check(CLI::ExistingFile);

    // Feature 3: Option with value
    std::string output_file = "out.txt";
    app.add_option("-o,--output", output_file, "Output file name");

    // Feature 4: Boolean flag
    bool verbose = false;
    app.add_flag("-v,--verbose", verbose, "Enable verbose output");

    // Feature 5: Required option with default
    int repeat = 1;
    app.add_option("-r,--repeat", repeat, "Repeat count")->required()->check(CLI::PositiveNumber);

    // This is where the magical parsing happens!
    CLI11_PARSE(app, argc, argv);

    // Print configuration
    std::cout << "Verbose: " << std::boolalpha << verbose << "\n";
    std::cout << "Output: " << output_file << "\n";
    std::cout << "Repeat: " << repeat << "\n";
    std::cout << "Files:\n";
    for (const auto& file : input_files) {
        std::cout << " - " << file << "\n";
    }

    return 0;
}
````

### 3. Write `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.16)
project(CLI11Demo)

include(FetchContent)

FetchContent_Declare(
  CLI11
  GIT_REPOSITORY https://github.com/CLIUtils/CLI11.git
  GIT_TAG v2.6.1  # or use main if you want latest
)

FetchContent_MakeAvailable(CLI11)

add_executable(${PROJECT_NAME} src/main.cpp)
target_link_libraries(${PROJECT_NAME} PRIVATE CLI11::CLI11)
```

### 4. Build the Project

```bash
cmake -B build
cmake --build build
```

### 5. Run the Tool

```bash
./build/clitest --help
./build/clitest input.txt -v --output result.txt --repeat 3
./build/clitest --version
```

Expected output:

```
Verbose: true
Output: result.txt
Repeat: 3
Files:
 - input.txt
```

Try also running with missing required options to see CLI11’s error reporting and automatic help suggestions.

---

## Questions

**1. What happens if you run `clitest` without any arguments?**

<details>
<summary>Click to reveal answer</summary>

CLI11 will report an error because the required positional argument (`files`) and required option (`--repeat`) were not provided. It will also suggest running `--help`.

</details>

**2. How does CLI11 validate file existence?**

<details>
<summary>Click to reveal answer</summary>

The `check(CLI::ExistingFile)` call ensures that each argument passed to the `files` parameter exists as a file in the filesystem.

</details>

**3. What is the difference between `add_flag()` and `add_option()`?**

<details>
<summary>Click to reveal answer</summary>

`add_flag()` is for boolean flags (on/off), while `add_option()` is for arguments that take values, such as filenames or integers.

</details>

**4. How does CLI11 handle `--help` and `--version` flags?**

<details>
<summary>Click to reveal answer</summary>

They are automatically handled if registered. `--help` prints usage and all options. `--version` prints the string passed to `set_version_flag`.

</details>

**5. What does `CLI11_PARSE` do?**

<details>
<summary>Click to reveal answer</summary>

It parses `argc` and `argv`, applies validation, and throws an exception or exits the program with an error message if something is invalid or missing.

</details>

---

## Advice

Using `CLI11` streamlines command-line parsing in C++ and encourages you to think about your program’s interface from the start. By combining CMake's `FetchContent` with a header-only library like CLI11, you keep your codebase portable and build-friendly. Always test the CLI manually with edge cases (missing files, negative integers, no arguments). You can extend this pattern to subcommands, config files, or interactive modes. Start simple — then iterate with confidence.

---
