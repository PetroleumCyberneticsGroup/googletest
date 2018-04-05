########################################################################
# CMake build script for Google Test.
#
# To run the tests for Google Test itself on Linux, use 'make test' or
# ctest.  You can select which tests to run using 'ctest -R regex'.
# For more options, run 'ctest --help'.
#
# Modified by Einar J.M. Baumann (2018) <einar.baumann@gmail.com>
# Removed tests and samples, as well as the code used by them.
# Added install target.

# BUILD_SHARED_LIBS is a standard CMake variable, but we declare it here to
# make it prominent in the GUI.
option(BUILD_SHARED_LIBS "Build shared libraries (DLLs)." OFF)

# When other libraries are using a shared version of runtime libraries,
# Google Test also has to use one.
option(
  gtest_force_shared_crt
  "Use shared (DLL) run-time lib even when Google Test is built as static lib."
  OFF)

option(gtest_disable_pthreads "Disable uses of pthreads in gtest." OFF)

# Defines pre_project_set_up_hermetic_build() and set_up_hermetic_build().
include(cmake/hermetic_build.cmake OPTIONAL)

if (COMMAND pre_project_set_up_hermetic_build)
  pre_project_set_up_hermetic_build()
endif()

########################################################################
#
# Project-wide settings

# Name of the project.
#
# CMake files in this project can refer to the root source directory
# as ${gtest_SOURCE_DIR} and to the root binary directory as
# ${gtest_BINARY_DIR}.
# Language "C" is required for find_package(Threads).
project(gtest CXX C)
cmake_minimum_required(VERSION 2.6.2)

if (COMMAND set_up_hermetic_build)
  set_up_hermetic_build()
endif()

# Define helper functions and macros used by Google Test.
include(cmake/internal_utils.cmake)

config_compiler_and_linker()  # Defined in internal_utils.cmake.

# Where Google Test's .h files can be found.
include_directories(
  ${gtest_SOURCE_DIR}/include
  ${gtest_SOURCE_DIR})

# Where Google Test's libraries can be found.
link_directories(${gtest_BINARY_DIR}/src)

########################################################################
#
# Defines the gtest & gtest_main libraries.  User tests should link
# with one of them.

# Google Test libraries.  We build them using more strict warnings than what
# are used for other targets, to ensure that gtest can be compiled by a user
# aggressive about warnings.
cxx_library(gtest "${cxx_strict}" src/gtest-all.cc)
cxx_library(gtest_main "${cxx_strict}" src/gtest_main.cc)
target_link_libraries(gtest_main gtest)

#######################################################################
# Add install target
SET(GTEST_INTERFACES
    include/gtest/gtest-death-test.h
    include/gtest/gtest.h
    include/gtest/gtest-message.h
    include/gtest/gtest-param-test.h
    include/gtest/gtest-param-test.h.pump
    include/gtest/gtest_pred_impl.h
    include/gtest/gtest-printers.h
    include/gtest/gtest_prod.h
    include/gtest/gtest-spi.h
    include/gtest/gtest-test-part.h
    include/gtest/gtest-typed-test.h
)
SET(GTEST_PRIVATE_HEADERS
    include/gtest/internal/gtest-death-test-internal.h
    include/gtest/internal/gtest-filepath.h
    include/gtest/internal/gtest-internal.h
    include/gtest/internal/gtest-linked_ptr.h
    include/gtest/internal/gtest-param-util-generated.h
    include/gtest/internal/gtest-param-util-generated.h.pump
    include/gtest/internal/gtest-param-util.h
    include/gtest/internal/gtest-port.h
    include/gtest/internal/gtest-string.h
    include/gtest/internal/gtest-tuple.h
    include/gtest/internal/gtest-tuple.h.pump
    include/gtest/internal/gtest-type-util.h
    include/gtest/internal/gtest-type-util.h.pump
)
set_target_properties(gtest PROPERTIES PUBLIC_HEADER "${GTEST_INTERFACES}")
set_target_properties(gtest PROPERTIES PRIVATE_HEADER "${GTEST_PRIVATE_HEADERS}")
install(TARGETS gtest DESTINATION lib
        PRIVATE_HEADER DESTINATION include/gtest/internal
        PUBLIC_HEADER DESTINATION include/gtest)
install(TARGETS gtest_main DESTINATION lib)


