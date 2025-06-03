cmake_minimum_required(VERSION 3.15)
project(Snake)

# Установка политик CMake
if(POLICY CMP0077)
  cmake_policy(SET CMP0077 NEW)
endif()

set(SFML_DIR "path/to/SFML/lib/cmake/SFML")
find_package(SFML 2.5 COMPONENTS graphics window system REQUIRED)

find_package(SFML REQUIRED network audio graphics window system)

file(COPY "${CMAKE_CURRENT_SOURCE_DIR}/images" DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/")

add_executable(snake main.cpp)

target_link_libraries(snake PRIVATE sfml-system sfml-window sfml-graphics sfml-network sfml-audio)
