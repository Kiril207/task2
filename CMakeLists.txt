cmake_minimum_required(VERSION 3.5...3.25)
project(Snake LANGUAGES CXX)

set(PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_NAME "snake-game")
set(CPACK_PACKAGE_VENDOR "Your Name")
set(CPACK_PACKAGE_DESCRIPTION "Simple Snake Game using SFML")
set(CPACK_PACKAGE_CONTACT "your.email@example.com")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Your Name")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libsfml-dev (>= 2.5.1), libc6 (>= 2.27)")
set(CPACK_GENERATOR "DEB")

include(CPack)

set(SFML_VERSION 2.5.1)

if(WIN32)
    find_package(SFML ${SFML_VERSION} REQUIRED COMPONENTS graphics window system)
else()
    find_package(SFML ${SFML_VERSION} QUIET COMPONENTS graphics window system audio network)
    if(NOT SFML_FOUND)
        message(STATUS "SFML не найден, пробуем скачать...")
        include(FetchContent)
        set(SFML_URL_OPTIONS
            URL https://www.sfml-dev.org/files/SFML-${SFML_VERSION}-sources.zip
            URL https://github.com/SFML/SFML/archive/refs/tags/${SFML_VERSION}.zip
        )
        FetchContent_Declare(
            sfml
            ${SFML_URL_OPTIONS}
            SOURCE_DIR ${CMAKE_BINARY_DIR}/_deps/sfml-src
            BINARY_DIR ${CMAKE_BINARY_DIR}/_deps/sfml-build
            DOWNLOAD_EXTRACT_TIMESTAMP TRUE
        )
        set(BUILD_SHARED_LIBS OFF CACHE BOOL "Build static libraries")
        set(SFML_BUILD_DOC FALSE CACHE BOOL "Don't build documentation")
        set(SFML_BUILD_EXAMPLES FALSE CACHE BOOL "Don't build examples")
        FetchContent_MakeAvailable(sfml)
        set(SFML_ROOT ${CMAKE_BINARY_DIR}/_deps/sfml-build)
        set(SFML_INCLUDE_DIR ${CMAKE_BINARY_DIR}/_deps/sfml-src/include)
        find_package(SFML ${SFML_VERSION} REQUIRED COMPONENTS graphics window system audio network)
    endif()
    if(NOT SFML_FOUND)
        message(FATAL_ERROR "SFML не найден и не может быть загружен автоматически")
    endif()
endif()

add_executable(snake main.cpp)
target_compile_features(snake PRIVATE cxx_std_17)
if(SFML_INCLUDE_DIR)
    target_include_directories(snake PRIVATE ${SFML_INCLUDE_DIR})
endif()

foreach(COMPONENT graphics window system audio network)
    if(TARGET sfml-${COMPONENT})
        target_link_libraries(snake PRIVATE sfml-${COMPONENT})
    endif()
endforeach()

install(TARGETS snake 
    RUNTIME DESTINATION bin
    BUNDLE DESTINATION bin
)

if(WIN32)
    add_custom_command(TARGET snake POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "C:/ProgramData/chocolatey/lib/sfml/tools/bin/sfml-graphics-2.dll"
            "C:/ProgramData/chocolatey/lib/sfml/tools/bin/sfml-window-2.dll"
            "C:/ProgramData/chocolatey/lib/sfml/tools/bin/sfml-system-2.dll"
            "$<TARGET_FILE_DIR:snake>"
    )
    install(FILES
        "C:/ProgramData/chocolatey/lib/sfml/tools/bin/sfml-graphics-2.dll"
        "C:/ProgramData/chocolatey/lib/sfml/tools/bin/sfml-window-2.dll"
        "C:/ProgramData/chocolatey/lib/sfml/tools/bin/sfml-system-2.dll"
        DESTINATION bin
    )
endif()
