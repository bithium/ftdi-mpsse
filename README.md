# ftdi-mpsse

This project contains the [FTDI MPSSE](http://www.ftdichip.com/Support/SoftwareExamples/MPSSE.htm) libraries,
in a format that can be easily used in [CMake](https://cmake.org/) projects.

## Usage

In order to use this library add this repository to a sub-folder (e.g. _lib/ftdi-mpsse_)
in your project and then add the following to your _CMakeLists.txt_

```cmake
# ...
add_subdirectory(lib/ftdi-mpsse)
# ...
include_directories(${FTDI_MPSSE_INCLUDE_DIRS})
# ...
```

 * for SPI:
    ```cmake
    # ...
    add_executable(foobar ...)
    target_link_libraries(foobar ftdi-mpsse-spi)
    # ...
    ```

 * for I2C:
    ```cmake
    # ...
    add_executable(foobar ...)
    target_link_libraries(foobar ftdi-mpsse-i2c)
    # ...
    ```

## License

See LICENSE file.

This license applies only to the CMake related files.
Other files are copyright of Future Technology Devices International Limited.
See each file header for further details.
