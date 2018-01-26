cmake_minimum_required(VERSION 3.6 FATAL_ERROR)
project(libuv C)

option(BUILD_SHARED_LIBS "" OFF)
option(DISABLE_CONAN "" OFF)

message(STATUS PLEX_PLATFORM = ${PLEX_PLATFORM})

if (PLEX_PLATFORM STREQUAL "Windows")
  message(STATUS "No proctitle.c on Windows")
  set(USE_PROCTITLE FALSE)
elseif (PLEX_PLATFORM STREQUAL "FreeBSD")
  message(STATUS "No proctitle.c on FreeBSD")
  set(USE_PROCTITLE FALSE)
else()
  set(USE_PROCTITLE TRUE)
endif()

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
    src/unix/udp.c>

  $<$<BOOL:${USE_PROCTITLE}>:
    src/unix/proctitle.c>

  $<$<PLATFORM_ID:Linux>:
    src/unix/linux-core.c
    src/unix/linux-inotify.c
    src/unix/linux-syscalls.c>

  $<$<PLATFORM_ID:Darwin>:
    src/unix/darwin.c
    src/unix/kqueue.c
    src/unix/fsevents.c
    src/unix/darwin-proctitle.c
    src/unix/pthread-barrier.c>

  $<$<PLATFORM_ID:FreeBSD>:
    src/unix/freebsd.c
    src/unix/kqueue.c>

  $<$<STREQUAL:${PLEX_PLATFORM},Android>:
    src/unix/pthread-fixes.c
    src/unix/android-ifaddrs.c
    src/unix/pthread-barrier.c>
)

target_compile_definitions(uv PRIVATE
  $<$<NOT:$<PLATFORM_ID:Windows>>:_LARGEFILE_SOURCE _FILE_OFFSET_BITS=64>
  $<$<PLATFORM_ID:Linux>:_GNU_SOURCE _POSIX_C_SOURCE=200112>
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
  $<$<PLATFORM_ID:Linux>:-ansi>
)

target_link_libraries(uv Threads::Threads
  $<$<NOT:$<PLATFORM_ID:Windows>>:m>
  $<$<PLATFORM_ID:Linux>:dl>
  $<$<PLATFORM_ID:FreeBSD>:kvm>
  $<$<STREQUAL:${PLEX_PLATFORM},Linux>:rt>
  $<$<PLATFORM_ID:Windows>:advapi32>
  $<$<PLATFORM_ID:Windows>:iphlpapi>
  $<$<PLATFORM_ID:Windows>:psapi>
  $<$<PLATFORM_ID:Windows>:shell32>
  $<$<PLATFORM_ID:Windows>:user32>
  $<$<PLATFORM_ID:Windows>:userenv>
  $<$<PLATFORM_ID:Windows>:ws2_32>
)

option(ENABLE_TESTS "" OFF)
if(ENABLE_TESTS)
  enable_testing()

  add_executable(UVTest
    test/blackhole-server.c
    test/echo-server.c
    test/run-tests.c
    test/runner.c
    test/test-get-loadavg.c
    test/test-active.c
    test/test-async.c
    test/test-async-null-cb.c
    test/test-callback-stack.c
    test/test-callback-order.c
    test/test-close-fd.c
    test/test-close-order.c
    test/test-connection-fail.c
    test/test-cwd-and-chdir.c
    test/test-default-loop-close.c
    test/test-delayed-accept.c
    test/test-eintr-handling.c
    test/test-error.c
    test/test-embed.c
    test/test-emfile.c
    test/test-fail-always.c
    test/test-fs.c
    test/test-fs-event.c
    test/test-get-currentexe.c
    test/test-get-memory.c
    test/test-get-passwd.c
    test/test-getaddrinfo.c
    test/test-getnameinfo.c
    test/test-getsockname.c
    test/test-handle-fileno.c
    test/test-homedir.c
    test/test-hrtime.c
    test/test-idle.c
    test/test-ip6-addr.c
    test/test-ipc.c
    test/test-ipc-send-recv.c
    test/test-loop-handles.c
    test/test-loop-alive.c
    test/test-loop-close.c
    test/test-loop-stop.c
    test/test-loop-time.c
    test/test-loop-configure.c
    test/test-walk-handles.c
    test/test-watcher-cross-stop.c
    test/test-multiple-listen.c
    test/test-osx-select.c
    test/test-pass-always.c
    test/test-ping-pong.c
    test/test-pipe-bind-error.c
    test/test-pipe-connect-error.c
    test/test-pipe-connect-multiple.c
    test/test-pipe-connect-prepare.c
    test/test-pipe-getsockname.c
    test/test-pipe-pending-instances.c
    test/test-pipe-sendmsg.c
    test/test-pipe-server-close.c
    test/test-pipe-close-stdout-read-stdin.c
    test/test-pipe-set-non-blocking.c
    test/test-platform-output.c
    test/test-poll.c
    test/test-poll-close.c
    test/test-poll-close-doesnt-corrupt-stack.c
    test/test-poll-closesocket.c
    test/test-process-title.c
    test/test-queue-foreach-delete.c
    test/test-ref.c
    test/test-run-nowait.c
    test/test-run-once.c
    test/test-semaphore.c
    test/test-shutdown-close.c
    test/test-shutdown-eof.c
    test/test-shutdown-twice.c
    test/test-signal.c
    test/test-signal-multiple-loops.c
    test/test-socket-buffer-size.c
    test/test-spawn.c
    test/test-fs-poll.c
    test/test-stdio-over-pipes.c
    test/test-tcp-bind-error.c
    test/test-tcp-bind6-error.c
    test/test-tcp-close.c
    test/test-tcp-close-accept.c
    test/test-tcp-close-while-connecting.c
    test/test-tcp-create-socket-early.c
    test/test-tcp-connect-error-after-write.c
    test/test-tcp-shutdown-after-write.c
    test/test-tcp-flags.c
    test/test-tcp-connect-error.c
    test/test-tcp-connect-timeout.c
    test/test-tcp-connect6-error.c
    test/test-tcp-open.c
    test/test-tcp-write-to-half-open-connection.c
    test/test-tcp-write-after-connect.c
    test/test-tcp-writealot.c
    test/test-tcp-write-fail.c
    test/test-tcp-try-write.c
    test/test-tcp-unexpected-read.c
    test/test-tcp-oob.c
    test/test-tcp-read-stop.c
    test/test-tcp-write-queue-order.c
    test/test-threadpool.c
    test/test-threadpool-cancel.c
    test/test-thread-equal.c
    test/test-tmpdir.c
    test/test-mutexes.c
    test/test-thread.c
    test/test-barrier.c
    test/test-condvar.c
    test/test-timer-again.c
    test/test-timer-from-check.c
    test/test-timer.c
    test/test-tty.c
    test/test-udp-bind.c
    test/test-udp-create-socket-early.c
    test/test-udp-dgram-too-big.c
    test/test-udp-ipv6.c
    test/test-udp-open.c
    test/test-udp-options.c
    test/test-udp-send-and-recv.c
    test/test-udp-send-immediate.c
    test/test-udp-send-unreachable.c
    test/test-udp-multicast-join.c
    test/test-udp-multicast-join6.c
    test/test-dlerror.c
    test/test-udp-multicast-ttl.c
    test/test-ip4-addr.c
    test/test-ip6-addr.c
    test/test-udp-multicast-interface.c
    test/test-udp-multicast-interface6.c
    test/test-udp-try-send.c
    $<$<NOT:$<PLATFORM_ID:Windows>>:test/runner-unix.c>
  )
  target_compile_definitions(UVTest PRIVATE _GNU_SOURCE)
  target_link_libraries(UVTest uv -lutil)
  add_test(NAME UVTests COMMAND $<TARGET_FILE:UVTest> WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
endif()
