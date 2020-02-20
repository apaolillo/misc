cmake_minimum_required(VERSION 3.10)
project(testlib C)

set(CMAKE_C_STANDARD 99)

add_library(testlib src/library.c)

target_include_directories(testlib
    PUBLIC
        $<BUILD_INTERFACE:
            ${CMAKE_SOURCE_DIR}/include1
            ${CMAKE_SOURCE_DIR}/include2
        >
        $<INSTALL_INTERFACE:include2>
    PRIVATE
        include
)

set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/installed")
install(
    TARGETS testlib
    EXPORT testlib
    DESTINATION lib)
install(
    EXPORT testlib
    NAMESPACE testlib::
    DESTINATION cmake
)
