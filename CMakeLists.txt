cmake_minimum_required(VERSION 3.15)
project(Snake LANGUAGES CXX)

# 1. Настройка версии и метаданных для пакета
set(PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_NAME "snake-game")
set(CPACK_PACKAGE_VENDOR "Your Name")
set(CPACK_PACKAGE_DESCRIPTION "Simple Snake Game using SFML")
set(CPACK_PACKAGE_CONTACT "your.email@example.com")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Your Name")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libsfml-dev (>= 2.5.1), libc6 (>= 2.27)")  # Зависимости
set(CPACK_GENERATOR "DEB")  # Генератор .deb-пакетов

# 2. Автозагрузка SFML через FetchContent 
include(FetchContent)
include(CPack)  # Подключение модуля упаковки

set(SFML_VERSION 2.5.1)
find_package(SFML ${SFML_VERSION} QUIET COMPONENTS graphics window system audio network)

if(NOT SFML_FOUND)
    message(STATUS "SFML не найден, скачиваем автоматически...")
    FetchContent_Declare(
        sfml
        URL https://www.sfml-dev.org/files/SFML-${SFML_VERSION}-sources.zip
        SOURCE_DIR ${CMAKE_BINARY_DIR}/_deps/sfml-src
        BINARY_DIR ${CMAKE_BINARY_DIR}/_deps/sfml-build
    )
    FetchContent_MakeAvailable(sfml)
    set(SFML_ROOT ${CMAKE_BINARY_DIR}/_deps/sfml-build)
    set(SFML_INCLUDE_DIR ${CMAKE_BINARY_DIR}/_deps/sfml-src/include)
endif()

find_package(SFML ${SFML_VERSION} REQUIRED COMPONENTS graphics window system audio network)

# 3. Копирование ресурсов
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/images DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

# 4. Создание исполняемого файла
add_executable(snake main.cpp)
target_compile_features(snake PRIVATE cxx_std_17)
target_include_directories(snake PRIVATE ${SFML_INCLUDE_DIR})
target_link_libraries(snake PRIVATE 
    sfml-graphics sfml-window sfml-system sfml-audio sfml-network
)

# 5. Установка в систему (для make install)
install(TARGETS snake 
    RUNTIME DESTINATION bin
    BUNDLE DESTINATION bin
)

# 6. Установка ресурсов (из папки images)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/images
    DESTINATION share/${PROJECT_NAME}
)

# 7. Генерация .deb-пакета
set(CPACK_DEBIAN_FILE_NAME "DEB-DEFAULT")  # Имя файла: snake-game_1.0.0_amd64.deb
set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)  # Автоматический расчёт зависимостей

# 8. Копирование DLL (только для Windows)
if(WIN32)
    add_custom_command(TARGET snake POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        "$<TARGET_FILE:sfml-graphics>"
        "$<TARGET_FILE:sfml-window>"
        "$<TARGET_FILE:sfml-system>"
        "$<TARGET_FILE:sfml-audio>"
        "$<TARGET_FILE:sfml-network>"
        "$<TARGET_FILE_DIR:snake>"
        COMMENT "Копирование DLL SFML"
    )
endif()
