cmake_minimum_required(VERSION 3.3)
project(range_lib)

# Set output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")

# Include necessary directories
set(BASEPATH "${CMAKE_SOURCE_DIR}")
include_directories(${BASEPATH})
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
include_directories(${BASEPATH}/vendor/gflags)
add_subdirectory(${BASEPATH}/vendor/gflags)

# Check for C++11 support
include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-std=c++11" COMPILER_SUPPORTS_CXX11)
if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
else()
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support.")
endif()

# Compiler flags
set(CMAKE_CXX_FLAGS "-march=native -ffast-math -fno-math-errno ${CMAKE_CXX_FLAGS} -O3")
set(CMAKE_BUILD_TYPE Release)

# Options
option(WITH_CUDA "Compile CUDA ray cast variants" ON)
set(CHUNK_SIZE 262144)
set(CHUNK_THREADS 256)

if (WITH_CUDA)
    add_definitions(-DUSE_CUDA=1)
    add_definitions(-DCHUNK_SIZE=${CHUNK_SIZE})
    add_definitions(-DNUM_THREADS=${CHUNK_THREADS})

    file(GLOB CUDA_SRCS 
        main.cpp 
        "includes/kernels.cu"
        "vendor/lodepng/lodepng.cpp"
    )

    find_package(CUDA REQUIRED)
    cuda_add_library(range_lib SHARED ${CUDA_SRCS})
else()
    add_definitions(-DUSE_CUDA=0)

    file(GLOB SRCS 
        main.cpp 
        "vendor/lodepng/lodepng.cpp"
    )
    add_library(range_lib SHARED ${SRCS})
endif()

# Link libraries
target_link_libraries(range_lib gflags pthread)

# ----------------------------------
# Package Export Configuration
# ----------------------------------

# Install the headers
install(DIRECTORY ${BASEPATH}/includes/ DESTINATION include/range_lib)

# Install the vendor directory
install(DIRECTORY ${BASEPATH}/vendor/ DESTINATION include/range_lib/vendor)

# Install the library
install(TARGETS range_lib
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)

# Create and install the package configuration file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/range_libConfigVersion.cmake"
    VERSION 1.0
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/range_libConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/range_libConfig.cmake"
    INSTALL_DESTINATION lib/cmake/range_lib
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/range_libConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/range_libConfigVersion.cmake"
    DESTINATION lib/cmake/range_lib
)
