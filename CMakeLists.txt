cmake_minimum_required(VERSION 3.5)
project(Snake LANGUAGES CXX)

set(PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_NAME "snake-game")
set(CPACK_PACKAGE_VENDOR "Your Name")
set(CPACK_PACKAGE_DESCRIPTION "Simple Snake Game using SFML")
set(CPACK_PACKAGE_CONTACT "your.email@example.com")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Your Name")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libsfml-dev (>= 2.5.1), libc6 (>= 2.27)")
set(CPACK_GENERATOR "DEB")
Ц
include(CPack)

set(SFML_VERSION 2.5.1)

# Путь к локальной сборке SFML (корректируй если структура внутри SFML-2.5.1 иная)
set(SFML_DIR "${CMAKE_SOURCE_DIR}/SFML-2.5.1/lib/cmake/SFML")

find_package(SFML ${SFML_VERSION} REQUIRED COMPONENTS graphics window system)

add_executable(snake main.cpp)
target_compile_features(snake PRIVATE cxx_std_17)
target_include_directories(snake PRIVATE ${CMAKE_SOURCE_DIR}/SFML-2.5.1/include)

target_link_libraries(snake PRIVATE sfml-graphics sfml-window sfml-system)

install(TARGETS snake 
    RUNTIME DESTINATION bin
    BUNDLE DESTINATION bin
)

if(WIN32)
    add_custom_command(TARGET snake POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${CMAKE_SOURCE_DIR}/SFML-2.5.1/bin/sfml-graphics-2.dll"
            "${CMAKE_SOURCE_DIR}/SFML-2.5.1/bin/sfml-window-2.dll"
            "${CMAKE_SOURCE_DIR}/SFML-2.5.1/bin/sfml-system-2.dll"
            $<TARGET_FILE_DIR:snake>
    )
    install(FILES
        "${CMAKE_SOURCE_DIR}/SFML-2.5.1/bin/sfml-graphics-2.dll"
        "${CMAKE_SOURCE_DIR}/SFML-2.5.1/bin/sfml-window-2.dll"
        "${CMAKE_SOURCE_DIR}/SFML-2.5.1/bin/sfml-system-2.dll"
        DESTINATION bin
    )
endif()
