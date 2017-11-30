cmake_minimum_required(VERSION 3.6 FATAL_ERROR)
project(libuv C)

option(BUILD_SHARED_LIBS "" OFF)

include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
conan_basic_setup(TARGET)

add_library(uv
  src/fs-poll.c
  src/inet.c
  src/threadpool.c
  src/uv-common.c
  src/version.c

  $<$<PLATFORM_ID:Windows>:
    src/win/async.c
    src/win/core.c
    src/win/dl.c
    src/win/error.c
    src/win/fs.c
    src/win/fs-event.c
    src/win/getaddrinfo.c
    src/win/getnameinfo.c
    src/win/handle.c
    src/win/loop-watcher.c
    src/win/pipe.c
    src/win/thread.c
    src/win/poll.c
    src/win/process.c
    src/win/process-stdio.c
    src/win/req.c
    src/win/signal.c
    src/win/snprintf.c
    src/win/stream.c
    src/win/tcp.c
    src/win/tty.c
    src/win/timer.c
    src/win/udp.c
    src/win/util.c
    src/win/winapi.c
    src/win/winsock.c>
  $<$<NOT:$<PLATFORM_ID:Windows>>:
    src/unix/async.c
    src/unix/core.c
    src/unix/dl.c
    src/unix/fs.c
    src/unix/getaddrinfo.c
    src/unix/getnameinfo.c
    src/unix/loop.c
    src/unix/loop-watcher.c
    src/unix/pipe.c
    src/unix/poll.c
    src/unix/process.c
    src/unix/signal.c
    src/unix/stream.c
    src/unix/tcp.c
    src/unix/thread.c
    src/unix/timer.c
    src/unix/tty.c
    src/unix/proctitle.c
    src/unix/udp.c>
  $<$<PLATFORM_ID:Linux>:
    src/unix/linux-core.c
    src/unix/linux-inotify.c
    src/unix/linux-syscalls.c>
  $<$<PLATFORM_ID:Darwin>:
    src/unix/darwin.c
    src/unix/fsevents.c
    src/unix/darwin-proctitle.c
    src/unix/pthread-barrier.c>
  $<$<STREQUAL:${PLEX_PLATFORM},Android>:
    src/unix/pthread-fixes.c
    src/unix/android-ifaddrs.c
    src/unix/pthread-barrier.c>
)

target_compile_definitions(uv PRIVATE
  $<$<NOT:$<PLATFORM_ID:Windows>>:_LARGEFILE_SOURCE _FILE_OFFSET_BITS=64>
  $<$<PLATFORM_ID:Linux>:_GNU_SOURCE>
  $<$<PLATFORM_ID:Windows>:_GNU_SOURCE _WIN32_WINNT=0x0600>
  $<$<PLATFORM_ID:Darwin>:_DARWIN_USE_64_BIT_INODE=1 _DARWIN_UNLIMITED_SELECT=1>
)

set_target_properties(uv PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS ON)

find_package(Threads)

target_include_directories(uv PRIVATE
  ${CMAKE_SOURCE_DIR}/include
  ${CMAKE_SOURCE_DIR}/src
)

target_compile_options(uv PRIVATE
  $<$<NOT:$<PLATFORM_ID:Windows>>:
  -fvisibility=hidden
  --std=gnu89
  -pedantic
  -Wall
  -Wextra
  -Wno-unused-parameter>
)

target_link_libraries(uv Threads::Threads
  $<$<NOT:$<PLATFORM_ID:Windows>>:m>
  $<$<PLATFORM_ID:Linux>:dl>
  $<$<STREQUAL:${PLEX_PLATFORM},Linux>:rt>
  $<$<PLATFORM_ID:Windows>:advapi32>
  $<$<PLATFORM_ID:Windows>:iphlpapi>
  $<$<PLATFORM_ID:Windows>:psapi>
  $<$<PLATFORM_ID:Windows>:shell32>
  $<$<PLATFORM_ID:Windows>:user32>
  $<$<PLATFORM_ID:Windows>:userenv>
  $<$<PLATFORM_ID:Windows>:ws2_32>
)
