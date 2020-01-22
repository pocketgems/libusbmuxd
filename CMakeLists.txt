cmake_minimum_required(VERSION 3.3.2)
project(libusbmuxd)

### libusbmuxd ###

add_subdirectory("../libplist/" libplist)

add_library(libusbmuxd
    common/collection.c
    common/socket.c
    common/thread.c
    src/libusbmuxd.c
)

target_include_directories(libusbmuxd
    PUBLIC
        include/
        common/
)

if (APPLE)
    target_compile_definitions(libusbmuxd
        PRIVATE
            -DSO_NOSIGPIPE
            -DHAVE_STPNCPY=1
    )
elseif(WIN32)
    target_compile_definitions(libusbmuxd
        PRIVATE
            -DSO_NOSIGPIPE
    )
else() # UNIX
    # SO_NOSIGPIPE not supported on linux
    target_compile_definitions(libusbmuxd
        PRIVATE
            -DHAVE_STPNCPY=1
    )
endif()

target_link_libraries(libusbmuxd libplist)

if (UNIX AND NOT APPLE)
    target_link_libraries(libusbmuxd pthread)
endif()

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
    set_property(
        TARGET iproxy
        APPEND_STRING PROPERTY LINK_FLAGS " Ws2_32.lib"
    )
endif()
