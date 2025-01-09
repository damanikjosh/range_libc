# RangeLibc

This is the modified version of [range_libc](https://github.com/kctess5/range_libc) allowing it to be imported in ROS2 packages.

# Build with Docker

The package was tested on Docker with `osrf/ros:jazzy-desktop-full` image. To install the library with CUDA support, add the following at the top of your Dockerfile

```Dockerfile
FROM nvidia/cuda:12.6.3-devel-ubuntu24.04 as builder

# Install necessary dependencies for building rangelibc
RUN apt-get update && apt-get install -y \
    git \
    cmake \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Clone and build rangelibc
RUN git clone https://github.com/damanikjosh/range_libc.git \
    && cd range_libc \
    && mkdir build \
    && cd build \
    && cmake .. \
    && make \
    && make install

... the rest of the Dockerfile
```

Then, under the target image, add the following
```Dockerfile
COPY --from=builder /usr/local/lib/librange_lib.so /usr/local/lib/
COPY --from=builder /usr/local/include/range_lib /usr/local/include/range_lib
COPY --from=builder /usr/local/lib/cmake/range_lib /usr/local/lib/cmake/range_lib
```

To install the `range_lib` in the ROS package, add the following in the `CMakeLists.txt`

```cmake
find_package(range_lib REQUIRED)
include_directories(
  ...
  ${range_lib_INCLUDE_DIRS})
target_link_libraries(
  ...
  range_lib
)

```

In your code, you can import `range_lib` using

```cpp
#include <RangeLib.h>
```
