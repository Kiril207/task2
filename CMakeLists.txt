cmake_minimum_required(VERSION 3.16)
project(Snake LANGUAGES CXX)

set(PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_NAME "snake-game")
set(CPACK_PACKAGE_VENDOR "Your Name")
set(CPACK_PACKAGE_DESCRIPTION "Simple Snake Game using SFML 3.0.0")
set(CPACK_PACKAGE_CONTACT "your.email@example.com")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Your Name")
set(CPACK_GENERATOR "DEB")
include(CPack)

# Добавляем SFML как подмодуль
add_subdirectory(external/SFML)

add_executable(snake main.cpp)
target_compile_features(snake PRIVATE cxx_std_17)
target_link_libraries(snake PRIVATE sfml-graphics sfml-window sfml-system)
target_include_directories(snake PRIVATE external/SFML/include)

install(TARGETS snake RUNTIME DESTINATION bin)