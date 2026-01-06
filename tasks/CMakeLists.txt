project(parallel_programming_course LANGUAGES C CXX)

message(STATUS "Student's tasks")

# Test runner executables
set(FUNC_TEST_EXEC ppc_func_tests)
set(PERF_TEST_EXEC ppc_perf_tests)

# ——— Include helper scripts ——————————————————————————————————————
include(${CMAKE_SOURCE_DIR}/cmake/functions.cmake)

# ——— Initialize test executables —————————————————————————————————————
ppc_add_test(${FUNC_TEST_EXEC} common/runners/functional.cpp USE_FUNC_TESTS)
ppc_add_test(${PERF_TEST_EXEC} common/runners/performance.cpp USE_PERF_TESTS)

# ——— List of implementations ————————————————————————————————————————
set(PPC_IMPLEMENTATIONS "all;mpi;omp;seq;stl;tbb" CACHE STRING "Implementations to build (semicolon-separated)")

# ——— Configure each subproject —————————————————————————————————————
file(
  GLOB subdirs
  RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}"
  "${CMAKE_CURRENT_SOURCE_DIR}/*")
# Filter out non-task directories
set(task_dirs)
foreach(sub IN LISTS subdirs)
  if(IS_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/${sub}" AND NOT sub STREQUAL "common")
    list(APPEND task_dirs ${sub})
  endif()
endforeach()

# For Windows: use batched build to avoid command line length limits
if(WIN32)
  message(STATUS "Using batched build for Windows (${CMAKE_CXX_COMPILER_ID})")
  
  # Split tasks into batches of 30
  set(BATCH_SIZE 30)
  set(batch_num 0)
  set(current_batch)
  set(task_count 0)
  
  foreach(task_dir ${task_dirs})
    list(APPEND current_batch ${task_dir})
    math(EXPR task_count "${task_count} + 1")
    
    # When batch is full, process it
    if(task_count EQUAL BATCH_SIZE)
      ppc_configure_subproject_batch(${batch_num} "${current_batch}")
      math(EXPR batch_num "${batch_num} + 1")
      set(current_batch)
      set(task_count 0)
    endif()
  endforeach()
  
  # Process remaining tasks in the last batch
  if(current_batch)
    ppc_configure_subproject_batch(${batch_num} "${current_batch}")
  endif()
  
else()
  # Normal build for non-Windows or non-MSVC
  foreach(sub IN LISTS task_dirs)
    ppc_configure_subproject(${sub})
  endforeach()
endif()