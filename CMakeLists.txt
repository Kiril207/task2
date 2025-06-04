cmake_minimum_required(VERSION 3.15)
project(Snake LANGUAGES CXX)

# 1. Автозагрузка SFML через FetchContent (если не найден в системе)
include(FetchContent)

set(SFML_VERSION 2.5.1)  # Желаемая версия SFML

# Проверка, установлен ли SFML в системе
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
    
    # Настройка путей для собранного SFML
    set(SFML_ROOT ${CMAKE_BINARY_DIR}/_deps/sfml-build)
    set(SFML_INCLUDE_DIR ${CMAKE_BINARY_DIR}/_deps/sfml-src/include)
endif()

# 2. Поиск SFML (после загрузки или в системе)
find_package(SFML ${SFML_VERSION} REQUIRED COMPONENTS 
    graphics 
    window 
    system 
    audio 
    network
)

# 3. Копирование ресурсов в бинарную директорию
file(COPY 
    ${CMAKE_CURRENT_SOURCE_DIR}/images 
    DESTINATION ${CMAKE_CURRENT_BINARY_DIR}
)

# 4. Создание исполняемого файла
add_executable(snake 
    main.cpp
    # Другие исходные файлы...
)

# 5. Настройка компиляции
target_compile_features(snake PRIVATE cxx_std_17)
target_include_directories(snake PRIVATE ${SFML_INCLUDE_DIR})

# 6. Линковка SFML
target_link_libraries(snake PRIVATE 
    sfml-graphics 
    sfml-window 
    sfml-system 
    sfml-audio 
    sfml-network
)

# 7. Автоматическое копирование DLL (Windows)
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

# 8. Установка (опционально)
install(TARGETS snake RUNTIME DESTINATION bin)
