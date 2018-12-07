cmake_minimum_required(VERSION 3.3.2)
project(libusbmuxd)

### libusbmuxd ###

add_subdirectory("../libplist/" libplist)

add_library(libusbmuxd
    common/collection.c
    common/socket.c
    src/libusbmuxd.c
)

target_include_directories(libusbmuxd
    PUBLIC
        include/
        common/
)

if (NOT WIN32)
    target_compile_definitions(libusbmuxd
        PRIVATE
            -DSO_NOSIGPIPE
            -DHAVE_STPNCPY=1
    )
else()
    target_compile_definitions(libusbmuxd
        PRIVATE
            -DSO_NOSIGPIPE
    )
endif()

target_link_libraries(libusbmuxd libplist)

### iproxy ###

add_executable(iproxy
    tools/iproxy.c
)

target_include_directories(iproxy
    PUBLIC
        include/
    PRIVATE
        common/
)

target_link_libraries(iproxy libusbmuxd)

if (WIN32)
    set (WIN32_DEPS
        "${MINGW_BASE_DIR}/bin/libwinpthread-1.dll"
        "${MINGW_BASE_DIR}/bin/libstdc++-6.dll"
        "${MINGW_BASE_DIR}/bin/libgcc_s_dw2-1.dll"
    )
    add_custom_command(TARGET iproxy
        POST_BUILD
            COMMAND "${CMAKE_COMMAND}" -E copy ${WIN32_DEPS} "${CMAKE_CURRENT_BINARY_DIR}"
            COMMAND "${CMAKE_COMMAND}" -E copy ${WIN32_DEPS} "${CMAKE_CURRENT_BINARY_DIR}"
            COMMENT "Copy dll dependencies to build folder"
    )
endif()
