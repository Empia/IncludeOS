set(CMAKE_ASM_NASM_OBJECT_FORMAT "elf")
enable_language(ASM_NASM)

include_directories(${INCLUDEOS_ROOT}/api/posix)
include_directories(${LIBCXX_INCLUDE_DIR})
include_directories(${NEWLIB_INCLUDE_DIR})
include_directories(${INCLUDEOS_ROOT}/src/include)
include_directories(${INCLUDEOS_ROOT}/api)
include_directories(${INCLUDEOS_ROOT}/mod/)
include_directories(${INCLUDEOS_ROOT}/mod/GSL/)
include_directories(${INCLUDEOS_ROOT}/mod/uzlib/src) # tinf.h for tar

add_custom_command(
	OUTPUT apic_boot.o
	COMMAND ${CMAKE_ASM_NASM_COMPILER} -f bin -o apic_boot.bin ${CMAKE_CURRENT_SOURCE_DIR}/hw/apic_boot.asm
	COMMAND ${CMAKE_OBJCOPY} -I binary -O elf32-i386 -B i386 apic_boot.bin apic_boot.o
	DEPENDS hw/apic_boot.asm
)

# TODO: i wanted to use a glob, but then found out that not everything is included
set(OS_OBJECTS
  kernel/start.asm kernel/kernel_start.cpp kernel/sanity_checks.cpp
  kernel/syscalls.cpp kernel/interrupts.asm kernel/os.cpp kernel/cpuid.cpp
  kernel/memmap.cpp kernel/irq_manager.cpp kernel/pci_manager.cpp
  kernel/service_stub.cpp kernel/main_call.cpp
  kernel/elf.cpp kernel/terminal.cpp kernel/terminal_disk.cpp
  kernel/vga.cpp kernel/context.cpp kernel/context_asm.asm kernel/block.cpp
  kernel/profile.cpp kernel/profile_intr.asm kernel/softreset.cpp
  kernel/timers.cpp kernel/rtc.cpp kernel/rng.cpp kernel/rdrand.cpp
  util/memstream.c util/crc32.cpp util/async.cpp util/statman.cpp util/logger.cpp
  util/os_statman.cpp
  util/syslog_facility.cpp util/syslogd.cpp util/uri.cpp util/percent_encoding.cpp
  util/tar.cpp util/path_to_regex.cpp
  crt/c_abi.c crt/string.c crt/quick_exit.cpp crt/cxx_abi.cpp
  hw/ide.cpp hw/pit.cpp hw/pic.cpp hw/pci_device.cpp hw/cpu_freq_sampling.cpp
  hw/ps2.cpp hw/serial.cpp hw/serial1.cpp hw/cmos.cpp hw/block_device.cpp
  hw/acpi.cpp hw/apic.cpp hw/apic_timer.cpp hw/ioapic.cpp hw/apic_asm.asm
  hw/apic_revenant.cpp hw/msi.cpp hw/pci_msi.cpp
  virtio/virtio.cpp virtio/virtio_queue.cpp
  net/ethernet/ethernet.cpp net/inet_common.cpp net/ip4/arp.cpp net/ip4/ip4.cpp
  net/tcp/tcp.cpp net/tcp/connection.cpp net/tcp/connection_states.cpp
  net/tcp/write_queue.cpp net/tcp/rttm.cpp net/tcp/listener.cpp
  net/ip4/icmpv4.cpp net/ip4/udp.cpp net/ip4/udp_socket.cpp
  net/dns/dns.cpp net/dns/client.cpp net/dhcp/dh4client.cpp
  net/buffer_store.cpp net/inet4.cpp
  net/super_stack.cpp
  net/http/header.cpp net/http/header_fields.cpp net/http/message.cpp net/http/request.cpp
  net/http/response.cpp net/http/status_codes.cpp net/http/time.cpp net/http/version.cpp
  net/http/mime_types.cpp net/http/client.cpp net/http/client_connection.cpp net/http/cookie.cpp
  fs/disk.cpp fs/filesystem.cpp fs/mbr.cpp fs/path.cpp
  fs/fat.cpp fs/fat_async.cpp fs/fat_sync.cpp fs/memdisk.cpp
  posix/fd.cpp posix/tcp_fd.cpp posix/udp_fd.cpp posix/unistd.cpp posix/fcntl.cpp posix/syslog.cpp
  posix/sys/socket.cpp posix/sys/select.cpp posix/sys/utsname.cpp posix/sys/mman.cpp posix/arpa/inet.cpp
  posix/ucontext.cpp posix/ucontext_asm.asm
  posix/sys/stat.cpp posix/ftw.cpp posix/file_fd.cpp posix/dlfcn.cpp
  posix/pwd.cpp
)

add_library(os STATIC ${OS_OBJECTS} apic_boot.o)
add_dependencies(os PrecompiledLibraries)

file(GLOB CXX_ABI crt/cxxabi/*.cpp)

add_library(c++abi STATIC ${CXX_ABI})
add_dependencies(c++abi libcxx)

add_subdirectory(crt)
add_subdirectory(boot)
add_subdirectory(drivers)
add_subdirectory(plugins)

install(TARGETS c++abi os DESTINATION includeos/lib)


install(DIRECTORY ${INCLUDEOS_ROOT}/src/memdisk/ DESTINATION includeos/memdisk
        FILES_MATCHING PATTERN "*.*")

install(FILES service_name.cpp DESTINATION includeos/src)
install(FILES linker.ld DESTINATION includeos)
