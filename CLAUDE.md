# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building and Testing
```bash
# Configure build (standard development build)
cmake -S . -B build

# Build all targets
cmake --build build

# Build only functional tests
cmake --build build -t ppc_func_tests

# Build only performance tests  
cmake --build build -t ppc_perf_tests

# Run all tests
python3 scripts/run_tests.py

# Run functional tests only
python3 scripts/run_tests.py --running-type="functional"

# Run performance tests only
python3 scripts/run_tests.py --running-type="performance"

# Generate performance results
scripts/generate_perf_results.sh
```

### Code Quality and Formatting
```bash
# Run pre-commit hooks (formats C++, CMake, Python, checks YAML, shell scripts)
pre-commit run --all-files

# Format C++ code manually
clang-format -i <file.cpp>

# Run clang-tidy on C++ code
clang-tidy <file.cpp> -- -I<include_dirs>
```

### Documentation
```bash
# Build documentation (requires pip install -r docs/requirements.txt)
cmake -S . -B build -DUSE_DOCS=ON
cmake --build build -t docs_html

# Generate API docs with Doxygen
doxygen Doxyfile

# Serve docs locally
cd docs/_build/html && python3 -m http.server 8080
```

## Project Architecture

### Repository Structure
- `tasks/`: Student implementations of parallel programming exercises
  - Each task follows pattern: `author_taskname/`
  - Contains subdirs: `seq/`, `mpi/`, `omp/`, `tbb/`, `stl/` (various parallel technologies)
  - Each implementation has: `src/`, `include/`, `tests/functional/`, `tests/performance/`
- `modules/`: Core framework modules and utilities
- `cmake/`: CMake configuration files for parallel technologies (MPI, OpenMP, oneTBB)
- `3rdparty/`: External dependencies (GoogleTest, JSON, oneTBB, etc.)
- `scripts/`: Python utilities for testing and performance analysis
- `scoreboard/`: Automated grading and performance tracking system
- `docs/`: Sphinx documentation with multi-language support

### Parallel Technologies Supported
- **MPI (Message Passing Interface)**: Distributed memory parallelism
- **OpenMP**: Shared memory parallelism with compiler directives  
- **oneTBB (Threading Building Blocks)**: Task-based parallelism
- **std::thread**: C++ standard threading
- **Sequential (seq)**: Baseline single-threaded implementations

### Task Structure Pattern
Each student task follows this structure:
```
tasks/author_taskname/
├── settings.json          # Build configuration
├── info.json             # Task metadata  
├── report.md             # Implementation report
├── common/               # Shared headers/utilities
├── seq/                  # Sequential implementation
├── mpi/                  # MPI parallel implementation  
├── omp/                  # OpenMP implementation
├── tbb/                  # oneTBB implementation
├── stl/                  # std::thread implementation
└── tests/
    ├── functional/       # Correctness tests
    └── performance/      # Performance benchmarks
```

### Testing Framework
- Uses GoogleTest for all test implementations
- Two test types: `ppc_func_tests` (functional) and `ppc_perf_tests` (performance)  
- Tests automatically discover and run all student implementations
- Performance tests generate timing and speedup metrics
- Test runner: `scripts/run_tests.py` with filtering options

### Code Style Requirements
- **C++ Standard**: C++20
- **Formatting**: Google style with clang-format (120 char line limit)
- **Naming Conventions** (enforced by clang-tidy):
  - Classes: `CamelCase`
  - Functions: `CamelCase` 
  - Variables/members: `lower_case`
  - Private members: suffix `_`
  - Constants: `kCamelCase`
  - Macros: `UPPER_CASE`
- **Quality Checks**: Extensive clang-tidy rules with warnings as errors
- **Pre-commit Hooks**: Automatic formatting for C++, CMake, Python, YAML

### Build System Details
- CMake-based with modular configuration
- Automatic discovery of student tasks in `tasks/` directory
- Selective building by parallel technology: `PPC_IMPLEMENTATIONS` variable
- Integration with external libraries via git submodules
- Support for sanitizers, different build modes, and cross-platform builds

When working with this codebase:
1. Always run pre-commit hooks before committing
2. Follow the established task directory structure for new implementations
3. Use the test runner scripts rather than direct CMake/CTest execution
4. Check that implementations build and pass tests for all target parallel technologies