cmake_minimum_required(VERSION 3.13)

project(lvgl_simulator)

if (USE_SDL_BACKEND)
    add_compile_definitions(USE_SDL_BACKEND=1 LV_USE_GPU_SDL=1 USE_MONITOR=0)
elseif (USE_SDL_GPU)
    add_compile_definitions(USE_SDL_GPU=1 LV_USE_GPU_SDL=1 LV_USE_EXTERNAL_RENDERER=1 USE_MONITOR=0)
elseif (USE_NANOVG_GPU)
    add_compile_definitions(USE_NANOVG_GPU=1 LV_USE_DRAW_NANOVG=1 USE_MONITOR=0)
endif()

if (NOT MINGW)
    add_compile_options(-fsanitize=address -g3 -O0)
    add_link_options(-fsanitize=address -static-libasan)
endif()

find_package(PkgConfig REQUIRED)
set(SDL_STATIC ON)
pkg_check_modules(SDL2 REQUIRED sdl2)

add_subdirectory(lvgl)
target_link_libraries(lvgl SDL2)
target_include_directories(lvgl PUBLIC ${CMAKE_CURRENT_LIST_DIR})

add_subdirectory(lv_examples)
target_include_directories(lv_demos PUBLIC ${CMAKE_CURRENT_LIST_DIR})
file(GLOB LVGL_EXAMPLE_ASSETS lvgl/examples/assets/*.c)
target_sources(lv_demos PRIVATE ${LVGL_EXAMPLE_ASSETS})

add_subdirectory(lv_drivers)
target_link_libraries(lv_drivers SDL2)
target_include_directories(lv_drivers PUBLIC ${CMAKE_CURRENT_LIST_DIR})
target_include_directories(lv_drivers PUBLIC ${SDL_INCLUDE_DIR} )

add_executable(lv_simulator
        main/src/main.c
        main/src/mouse_cursor_icon.c
        )
target_link_libraries(lv_simulator lvgl lv_demos lvgl_examples lv_drivers SDL2)
target_link_directories(lv_simulator PRIVATE ${SDL_LIB_DIR})