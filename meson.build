project(
  'rtorrent',
  'cpp',
  version: '0.16.14',
  default_options: [
    'cpp_std=c++20',
    'warning_level=2',
    'buildtype=plain',
    'optimization=2',
    'debug=true',
    'b_lto=true',
  ],
  meson_version: '>=1.1.0',
)

cc = meson.get_compiler('cpp')

add_project_arguments(
  cc.get_supported_arguments(
    '-fvisibility=hidden',
    '-fvisibility-inlines-hidden',
  ),
  language: 'cpp',
)

# ── Debug / Warnings / Extra debug ──

if get_option('build-profile') == 'debug'
  add_project_arguments('-DDEBUG', language: 'cpp')
else
  add_project_arguments('-DNDEBUG', language: 'cpp')
endif

if get_option('extra-debug')
  add_project_arguments('-DUSE_EXTRA_DEBUG', language: 'cpp')
endif

if get_option('instrumentation')
  add_project_arguments('-DLT_INSTRUMENTATION', language: 'cpp')
endif

host_system = host_machine.system()

# ── epoll / kqueue ──

has_epoll = cc.has_header_symbol('sys/epoll.h', 'epoll_create')
has_kqueue = cc.has_header_symbol(
  'sys/event.h',
  'kqueue',
  prefix: '#include <sys/time.h>',
)

if has_epoll
  add_project_arguments('-DUSE_EPOLL', language: 'cpp')
endif
if has_kqueue
  add_project_arguments('-DUSE_KQUEUE', language: 'cpp')
endif
if not has_epoll and not has_kqueue
  error('Must enable at least one of either kqueue or epoll')
endif

# ── inotify ──

has_inotify = cc.has_header('sys/inotify.h')
if has_inotify
  add_project_arguments('-DUSE_INOTIFY', language: 'cpp')
endif

# ── madvise / posix_fadvise / fallocate / mincore ──

has_madvise = cc.has_header_symbol('sys/mman.h', 'madvise')
if has_madvise
  add_project_arguments('-DUSE_MADVISE', language: 'cpp')
endif

has_posix_fadvise = cc.has_function('posix_fadvise', prefix: '#include <fcntl.h>')

if has_posix_fadvise
  add_project_arguments('-DUSE_POSIX_FADVISE', language: 'cpp')
endif

has_fallocate = cc.has_function(
  'fallocate',
  prefix: '#define _GNU_SOURCE\n#include <fcntl.h>',
)

if has_fallocate
  add_project_arguments('-DHAVE_FALLOCATE', language: 'cpp')
endif

has_posix_fallocate = cc.has_function(
  'posix_fallocate',
  prefix: '#define _GNU_SOURCE\n#include <fcntl.h>',
)
if has_posix_fallocate
  add_project_arguments('-DHAVE_POSIX_FALLOCATE', language: 'cpp')
endif

mincore_unsigned = cc.compiles(
  '''
  #include <sys/types.h>
  #include <sys/mman.h>
  #include <unistd.h>
  void f() { mincore((char*)0, 0, (unsigned char*)0); }
''',
)
mincore_signed = cc.compiles(
  '''
  #include <sys/types.h>
  #include <sys/mman.h>
  #include <unistd.h>
  void f() { mincore((char*)0, 0, (char*)0); }
''',
)
if mincore_unsigned or mincore_signed
  add_project_arguments('-DUSE_MINCORE', language: 'cpp')
endif
if mincore_unsigned
  add_project_arguments('-DUSE_MINCORE_UNSIGNED', language: 'cpp')
endif

# ── execinfo (backtrace) ──

has_execinfo = cc.has_header('execinfo.h')
if has_execinfo
  add_project_arguments(
    '-DHAVE_BACKTRACE',
    '-DHAVE_EXECINFO_H',
    language: 'cpp',
  )
endif

# ── endian (be64toh / htobe64) ──

endian_header = ''
has_native_be64 = cc.links(
  '''
  #include <stdint.h>
  #include <endian.h>
  int main() { uint64_t x = be64toh(1); (void)x; return 0; }
''',
  name: 'be64toh via endian.h',
)
if has_native_be64
  endian_header = '<endian.h>'
  add_project_arguments(
    '-DCUSTOM_ENDIAN_HEADER=@0@'.format(endian_header),
    language: 'cpp',
  )
  add_project_arguments('-Dcustom_ntohll(x)=(be64toh(x))', language: 'cpp')
  add_project_arguments('-Dcustom_htonll(x)=(htobe64(x))', language: 'cpp')
else
  has_sys_endian = cc.links(
    '''
    #include <stdint.h>
    #include <sys/endian.h>
    int main() { uint64_t x = be64toh(1); (void)x; return 0; }
  ''',
    name: 'be64toh via sys/endian.h',
  )
  if has_sys_endian
    endian_header = '<sys/endian.h>'
    add_project_arguments(
      '-DCUSTOM_ENDIAN_HEADER=@0@'.format(endian_header),
      language: 'cpp',
    )
    add_project_arguments('-Dcustom_ntohll(x)=(be64toh(x))', language: 'cpp')
    add_project_arguments('-Dcustom_htonll(x)=(htobe64(x))', language: 'cpp')
  else
    has_libkern_order = cc.has_header('libkern/OSByteOrder.h')
    if has_libkern_order
      add_project_arguments(
        '-DCUSTOM_ENDIAN_HEADER=<libkern/OSByteOrder.h>',
        language: 'cpp',
      )
      add_project_arguments(
        '-Dcustom_ntohll(x)=(OSSwapBigToHostInt64(x))',
        language: 'cpp',
      )
      add_project_arguments(
        '-Dcustom_htonll(x)=(OSSwapHostToBigInt64(x))',
        language: 'cpp',
      )
    else
      add_project_arguments(
        '-DCUSTOM_ENDIAN_HEADER=<stdint.h>',
        language: 'cpp',
      )
      if host_machine.endian() == 'big'
        add_project_arguments(
          '-Dcustom_ntohll(x)=((uint64_t)(x))',
          language: 'cpp',
        )
        add_project_arguments(
          '-Dcustom_htonll(x)=((uint64_t)(x))',
          language: 'cpp',
        )
      else
        add_project_arguments(
          '-Dcustom_ntohll(x)=__builtin_bswap64((uint64_t)(x))',
          language: 'cpp',
        )
        add_project_arguments(
          '-Dcustom_htonll(x)=__builtin_bswap64((uint64_t)(x))',
          language: 'cpp',
        )
      endif
    endif
  endif
endif

# ── cacheline size ──
# Default: 64 bytes on x86, 128 bytes on ARM/PPC/s390x.
# On Linux we query the kernel value via compute_int.

if host_system == 'linux'
  cacheline = cc.compute_int(
    'SMP_CACHE_BYTES',
    prefix: '#include <linux/cache.h>',
    guess: -1,
  )
else
  cacheline = -1
endif

if cacheline > 0
  add_project_arguments(
    '-DLT_SMP_CACHE_BYTES=@0@'.format(cacheline),
    language: 'cpp',
  )
else
  cpu = host_machine.cpu_family()
  if cpu in ['x86_64', 'x86', 'i686']
    add_project_arguments('-DLT_SMP_CACHE_BYTES=64', language: 'cpp')
  elif cpu in ['aarch64', 'arm', 'ppc64', 'ppc', 's390x']
    add_project_arguments('-DLT_SMP_CACHE_BYTES=128', language: 'cpp')
  else
    error('Unrecognized CPU architecture for cacheline: ' + cpu)
  endif
endif

# ── pthread_setname_np ──

has_pthread_setname_generic = cc.links(
  '''
  #define _GNU_SOURCE
  #include <pthread.h>
  int main() { pthread_t t; pthread_setname_np(t, "foo"); return 0; }
''',
  name: 'pthread_setname_np (generic)',
)
if has_pthread_setname_generic
  add_project_arguments('-DHAS_PTHREAD_SETNAME_NP_GENERIC', language: 'cpp')
else
  has_pthread_setname_darwin = cc.links(
    '''
    #include <pthread.h>
    int main() { pthread_t t; pthread_setname_np("foo"); return 0; }
  ''',
    name: 'pthread_setname_np (darwin)',
  )
  if has_pthread_setname_darwin
    add_project_arguments('-DHAS_PTHREAD_SETNAME_NP_DARWIN', language: 'cpp')
  endif
endif

# ── pthread stack size (auto-enabled on musl) ──

pthread_setstacksize_opt = get_option('pthread-setstacksize')
if pthread_setstacksize_opt.auto()
  # Check if we're on musl (musl has small default stack)
  is_musl = cc.get_define('__MUSL__') != ''
  enable_pthread_setstacksize = is_musl
else
  enable_pthread_setstacksize = pthread_setstacksize_opt.enabled()
endif

if enable_pthread_setstacksize
  add_project_arguments('-DUSE_PTHREAD_SETSTACKSIZE', language: 'cpp')
  add_project_arguments(
    '-DDEFAULT_PTHREAD_STACKSIZE=(8*1024*1024)',
    language: 'cpp',
  )
endif

# ── address space ──

addr_space_opt = get_option('address-space-size-mb')
is_64bit = cc.sizeof('long') == 8
if addr_space_opt > 0
  add_project_arguments(
    '-DDEFAULT_ADDRESS_SPACE_SIZE=@0@'.format(addr_space_opt),
    language: 'cpp',
  )
elif is_64bit
  add_project_arguments('-DDEFAULT_ADDRESS_SPACE_SIZE=32768', language: 'cpp')
else
  add_project_arguments('-DDEFAULT_ADDRESS_SPACE_SIZE=1024', language: 'cpp')
endif

# ── Atomic ──

atomic_lib = cc.find_library('atomic', required: false)
has_atomic_no_lib = cc.links(
  '''
  #include <cstdint>
  #include <atomic>
  int main() { std::atomic<uint64_t> x(0); return x.load(); }
''',
  name: '64-bit atomics without -latomic',
)
if has_atomic_no_lib
  atomic_dep = declare_dependency()
else
  has_atomic_with_lib = cc.links(
    '''
    #include <cstdint>
    #include <atomic>
    int main() { std::atomic<uint64_t> x(0); return x.load(); }
  ''',
    dependencies: atomic_lib,
    name: '64-bit atomics with -latomic',
  )
  if has_atomic_with_lib
    atomic_dep = atomic_lib
  else
    error('Compiler target lacks proper 64-bit atomic support')
  endif
endif

# ═══════════════════════════════════════════════════════════════════════════════
# Dependencies
# ═══════════════════════════════════════════════════════════════════════════════

thread_dep = dependency('threads')
curl_dep = dependency('libcurl')
zlib_dep = dependency('zlib')
crypto_dep = dependency('libcrypto', required: true)

add_project_arguments('-DUSE_OPENSSL', '-DUSE_OPENSSL_SHA', language: 'cpp')

# ── Optional: ncurses ──

ncurses_opt = get_option('ncurses')
ncurses_dep = disabler()
using_ncursesw = false

if ncurses_opt.allowed()
  ncursesw_dep = dependency('ncursesw', required: false)
  ncurses_dep = dependency('ncurses', required: false)
  if ncursesw_dep.found()
    ncurses_dep = ncursesw_dep
    using_ncursesw = true
  endif
endif

if ncurses_opt.enabled() and not ncurses_dep.found()
  error('ncurses support requested but neither ncursesw nor ncurses found')
endif
no_ncurses = not ncurses_dep.found()
if no_ncurses
  add_project_arguments('-DHAVE_NO_NCURSES', language: 'cpp')
endif
if ncurses_dep.found()
  # Detect which curses header is available
  has_curses_header = cc.has_header('ncursesw/curses.h')
  if has_curses_header
    add_project_arguments('-DHAVE_NCURSESW_CURSES_H', language: 'cpp')
  else
    has_curses_header = cc.has_header('ncursesw.h')
    if has_curses_header
      add_project_arguments('-DHAVE_NCURSESW_H', language: 'cpp')
    else
      has_curses_header = cc.has_header('ncurses/curses.h')
      if has_curses_header
        add_project_arguments('-DHAVE_NCURSES_CURSES_H', language: 'cpp')
      else
        has_curses_header = cc.has_header('ncurses.h')
        if has_curses_header
          add_project_arguments('-DHAVE_NCURSES_H', language: 'cpp')
        else
          has_curses_header = cc.has_header('curses.h')
          if has_curses_header
            add_project_arguments('-DHAVE_CURSES_H', language: 'cpp')
          endif
        endif
      endif
    endif
  endif
  if using_ncursesw
    add_project_arguments('-DHAVE_NCURSESW', language: 'cpp')
  elif ncurses_dep.found()
    add_project_arguments('-DHAVE_NCURSES', language: 'cpp')
  endif
  add_project_arguments(
    '-DHAVE_CURSES',
    '-DHAVE_CURSES_ENHANCED',
    language: 'cpp',
  )
endif

# ── Optional: Lua ──

lua_opt = get_option('lua')
lua_dep = disabler()

if lua_opt.allowed()
  foreach name : ['lua', 'luajit', 'lua5.4', 'lua5.3', 'lua5.2', 'lua5.1']
    if not lua_dep.found()
      lua_dep = dependency(name, required: false)
    endif
  endforeach
endif

if not lua_dep.found() and lua_opt.enabled()
  error('Lua not found')
endif
if lua_dep.found()
  add_project_arguments('-DHAVE_LUA', language: 'cpp')
  add_project_arguments(
    '-DLUA_DATADIR=PACKAGE_DATADIR "/lua"',
    language: 'cpp',
  )
  install_data(
    'lua/rtorrent.lua',
    install_dir: get_option('datadir') / 'rtorrent' / 'lua',
  )
endif

# ── Optional: XMLRPC ──

xmlrpc_c_opt = get_option('xmlrpc-c')
xmlrpc_tinyxml2_opt = get_option('xmlrpc-tinyxml2')

if xmlrpc_c_opt.enabled() and xmlrpc_tinyxml2_opt.enabled()
  error('xmlrpc-c and xmlrpc-tinyxml2 cannot be enabled together')
endif

if xmlrpc_c_opt.enabled()
  warning('xmlrpc-c is deprecated, consider using xmlrpc-tinyxml2 instead, which is bundled and support unicode')
endif

# tinyxml2 is bundled (src/rpc/tinyxml2/), no external dep needed.
# Prefer it over xmlrpc-c when both are auto (tinyxml2 has better UTF-8).
if xmlrpc_tinyxml2_opt.allowed()
  add_project_arguments('-DHAVE_XMLRPC_TINYXML2', language: 'cpp')
elif xmlrpc_c_opt.allowed()
  xmlrpc_c_dep = dependency('xmlrpc-c', required: xmlrpc_c_opt)
  if xmlrpc_c_opt.enabled() and not xmlrpc_c_dep.found()
    error('XMLRPC-C not found')
  endif
  if xmlrpc_c_dep.found()
    add_project_arguments('-DHAVE_XMLRPC_C', language: 'cpp')
    lib_deps += xmlrpc_c_dep
  endif
endif

# ── Optional: systemd ──

systemd_opt = get_option('systemd')
systemd_dep = dependency('libsystemd', required: systemd_opt)
if systemd_dep.found()
  add_project_arguments('-DHAVE_SYSTEMD', language: 'cpp')
endif

# ═══════════════════════════════════════════════════════════════════════════════
# Source files
# ═══════════════════════════════════════════════════════════════════════════════

libtorrent_sources = [
  # ── libtorrent ──
  'libtorrent/src/data/chunk.cc',
  'libtorrent/src/data/chunk_list.cc',
  'libtorrent/src/data/chunk_part.cc',
  'libtorrent/src/data/hash_check_queue.cc',
  'libtorrent/src/data/hash_chunk.cc',
  'libtorrent/src/data/hash_queue.cc',
  'libtorrent/src/data/hash_queue_node.cc',
  'libtorrent/src/data/hash_torrent.cc',
  'libtorrent/src/data/memory_chunk.cc',
  'libtorrent/src/data/socket_file.cc',
  'libtorrent/src/data/thread_disk.cc',
  'libtorrent/src/dht/dht_bucket.cc',
  'libtorrent/src/dht/dht_node.cc',
  'libtorrent/src/dht/dht_router.cc',
  'libtorrent/src/dht/dht_server.cc',
  'libtorrent/src/dht/dht_tracker.cc',
  'libtorrent/src/dht/dht_transaction.cc',
  'libtorrent/src/dht/transactions/dht_announce.cc',
  'libtorrent/src/dht/transactions/dht_search.cc',
  'libtorrent/src/download/available_list.cc',
  'libtorrent/src/download/chunk_selector.cc',
  'libtorrent/src/download/chunk_statistics.cc',
  'libtorrent/src/download/delegator.cc',
  'libtorrent/src/download/download_constructor.cc',
  'libtorrent/src/download/download_main.cc',
  'libtorrent/src/download/download_wrapper.cc',
  'libtorrent/src/manager.cc',
  'libtorrent/src/net/address_list.cc',
  'libtorrent/src/net/curl_get.cc',
  'libtorrent/src/net/curl_socket.cc',
  'libtorrent/src/net/curl_stack.cc',
  'libtorrent/src/net/dns_buffer.cc',
  'libtorrent/src/net/dns_cache.cc',
  'libtorrent/src/net/event_fd.cc',
  'libtorrent/src/net/listen.cc',
  'libtorrent/src/net/socket_datagram.cc',
  'libtorrent/src/net/socket_stream.cc',
  'libtorrent/src/net/thread_net.cc',
  'libtorrent/src/net/throttle_internal.cc',
  'libtorrent/src/net/throttle_list.cc',
  'libtorrent/src/net/udns_library.cc',
  'libtorrent/src/net/udns_resolver.cc',
  'libtorrent/src/protocol/extensions.cc',
  'libtorrent/src/protocol/handshake.cc',
  'libtorrent/src/protocol/handshake_encryption.cc',
  'libtorrent/src/protocol/handshake_manager.cc',
  'libtorrent/src/protocol/initial_seed.cc',
  'libtorrent/src/protocol/peer_connection_base.cc',
  'libtorrent/src/protocol/peer_connection_leech.cc',
  'libtorrent/src/protocol/peer_connection_metadata.cc',
  'libtorrent/src/protocol/peer_factory.cc',
  'libtorrent/src/protocol/request_list.cc',
  'libtorrent/src/runtime.cc',
  'libtorrent/src/thread_main.cc',
  'libtorrent/src/torrent/bitfield.cc',
  'libtorrent/src/torrent/chunk_manager.cc',
  'libtorrent/src/torrent/data/block.cc',
  'libtorrent/src/torrent/data/block_list.cc',
  'libtorrent/src/torrent/data/chunk_utils.cc',
  'libtorrent/src/torrent/data/download_data.cc',
  'libtorrent/src/torrent/data/file.cc',
  'libtorrent/src/torrent/data/file_list.cc',
  'libtorrent/src/torrent/data/file_list_iterator.cc',
  'libtorrent/src/torrent/data/file_manager.cc',
  'libtorrent/src/torrent/data/file_utils.cc',
  'libtorrent/src/torrent/data/transfer_list.cc',
  'libtorrent/src/torrent/download.cc',
  'libtorrent/src/torrent/download/choke_group.cc',
  'libtorrent/src/torrent/download/choke_queue.cc',
  'libtorrent/src/torrent/download/download_manager.cc',
  'libtorrent/src/torrent/download/resource_manager.cc',
  'libtorrent/src/torrent/event.cc',
  'libtorrent/src/torrent/exceptions.cc',
  'libtorrent/src/torrent/net/address_info.cc',
  'libtorrent/src/torrent/net/fd.cc',
  'libtorrent/src/torrent/net/http_get.cc',
  'libtorrent/src/torrent/net/http_stack.cc',
  'libtorrent/src/torrent/net/resolver.cc',
  'libtorrent/src/torrent/net/socket_address.cc',
  'libtorrent/src/torrent/net/socket_address_key.cc',
  'libtorrent/src/torrent/object.cc',
  'libtorrent/src/torrent/object_static_map.cc',
  'libtorrent/src/torrent/object_stream.cc',
  'libtorrent/src/torrent/path.cc',
  'libtorrent/src/torrent/peer/client_info.cc',
  'libtorrent/src/torrent/peer/client_list.cc',
  'libtorrent/src/torrent/peer/connection_list.cc',
  'libtorrent/src/torrent/peer/peer.cc',
  'libtorrent/src/torrent/peer/peer_info.cc',
  'libtorrent/src/torrent/peer/peer_list.cc',
  'libtorrent/src/torrent/rate.cc',
  'libtorrent/src/torrent/runtime/network_config.cc',
  'libtorrent/src/torrent/runtime/network_manager.cc',
  'libtorrent/src/torrent/runtime/runtime.cc',
  'libtorrent/src/torrent/runtime/socket_manager.cc',
  'libtorrent/src/torrent/system/poll_epoll.cc',
  'libtorrent/src/torrent/system/poll_kqueue.cc',
  'libtorrent/src/torrent/system/system.cc',
  'libtorrent/src/torrent/system/thread.cc',
  'libtorrent/src/torrent/throttle.cc',
  'libtorrent/src/torrent/torrent.cc',
  'libtorrent/src/torrent/tracker/dht_controller.cc',
  'libtorrent/src/torrent/tracker/manager.cc',
  'libtorrent/src/torrent/tracker/tracker.cc',
  'libtorrent/src/torrent/tracker/tracker_state.cc',
  'libtorrent/src/torrent/tracker/wrappers.cc',
  'libtorrent/src/torrent/types/string_utf8.cc',
  'libtorrent/src/torrent/utils/directory_events.cc',
  'libtorrent/src/torrent/utils/log.cc',
  'libtorrent/src/torrent/utils/log_buffer.cc',
  'libtorrent/src/torrent/utils/option_strings.cc',
  'libtorrent/src/torrent/utils/random.cc',
  'libtorrent/src/torrent/utils/resume.cc',
  'libtorrent/src/torrent/utils/scheduler.cc',
  'libtorrent/src/torrent/utils/string_manip.cc',
  'libtorrent/src/torrent/utils/uri_parser.cc',
  'libtorrent/src/tracker/thread_tracker.cc',
  'libtorrent/src/tracker/tracker_controller.cc',
  'libtorrent/src/tracker/tracker_dht.cc',
  'libtorrent/src/tracker/tracker_http.cc',
  'libtorrent/src/tracker/tracker_list.cc',
  'libtorrent/src/tracker/tracker_udp.cc',
  'libtorrent/src/tracker/tracker_worker.cc',
  'libtorrent/src/tracker/udp_router.cc',
  'libtorrent/src/utils/diffie_hellman.cc',
  'libtorrent/src/utils/instrumentation.cc',
]

rtorrent_sources = [
  # ── rtorrent ──
  'src/core/dht_manager.cc',
  'src/core/download.cc',
  'src/core/download_factory.cc',
  'src/core/download_list.cc',
  'src/core/http_queue.cc',
  'src/core/manager.cc',
  'src/core/view.cc',
  'src/core/view_manager.cc',
  'src/display/canvas.cc',
  'src/display/frame.cc',
  'src/display/manager.cc',
  'src/display/utils.cc',
  'src/display/text_element_list.cc',
  'src/display/text_element_string.cc',
  'src/display/text_element_value.cc',
  'src/display/window.cc',
  'src/display/window_download_chunks_seen.cc',
  'src/display/window_download_list.cc',
  'src/display/window_download_statusbar.cc',
  'src/display/window_download_transfer_list.cc',
  'src/display/window_file_list.cc',
  'src/display/window_http_queue.cc',
  'src/display/window_input.cc',
  'src/display/window_log.cc',
  'src/display/window_log_complete.cc',
  'src/display/window_peer_list.cc',
  'src/display/window_statusbar.cc',
  'src/display/window_string_list.cc',
  'src/display/window_text.cc',
  'src/display/window_title.cc',
  'src/display/window_tracker_list.cc',
  'src/input/bindings.cc',
  'src/input/input_event.cc',
  'src/input/manager.cc',
  'src/input/path_input.cc',
  'src/input/text_input.cc',
  'src/rpc/command.cc',
  'src/rpc/command_map.cc',
  'src/rpc/command_scheduler.cc',
  'src/rpc/command_scheduler_item.cc',
  'src/rpc/exec_file.cc',
  'src/rpc/lua.cc',
  'src/rpc/jsonrpc.cc',
  'src/rpc/rpc_manager.cc',
  'src/rpc/object_storage.cc',
  'src/rpc/parse.cc',
  'src/rpc/parse_commands.cc',
  'src/rpc/parse_options.cc',
  'src/rpc/scgi.cc',
  'src/rpc/scgi_task.cc',
  'src/rpc/xmlrpc.cc',
  'src/rpc/xmlrpc_c.cc',
  'src/rpc/xmlrpc_tinyxml2.cc',
  'src/rpc/tinyxml2/tinyxml2.cc',
  'src/scgi/thread_scgi.cc',
  'src/session/download_storer.cc',
  'src/session/session_manager.cc',
  'src/session/thread_session.cc',
  'src/ui/download.cc',
  'src/ui/download_list.cc',
  'src/ui/element_base.cc',
  'src/ui/element_chunks_seen.cc',
  'src/ui/element_download_list.cc',
  'src/ui/element_file_list.cc',
  'src/ui/element_log_complete.cc',
  'src/ui/element_menu.cc',
  'src/ui/element_peer_list.cc',
  'src/ui/element_string_list.cc',
  'src/ui/element_text.cc',
  'src/ui/element_tracker_list.cc',
  'src/ui/element_transfer_list.cc',
  'src/ui/root.cc',
  'src/utils/base64.cc',
  'src/utils/directory.cc',
  'src/utils/file_status_cache.cc',
  'src/utils/gzip.cc',
  'src/utils/lockfile.cc',
  'src/utils/watch_ready_queue.cc',
  'src/command_download.cc',
  'src/command_dynamic.cc',
  'src/command_events.cc',
  'src/command_file.cc',
  'src/command_ip.cc',
  'src/command_helpers.cc',
  'src/command_groups.cc',
  'src/command_local.cc',
  'src/command_logging.cc',
  'src/command_network.cc',
  'src/command_peer.cc',
  'src/command_throttle.cc',
  'src/command_tracker.cc',
  'src/command_scheduler.cc',
  'src/command_ui.cc',
  'src/control.cc',
  'src/globals.cc',
  'src/option_parser.cc',
  'src/setup.cc',
  'src/signal_handler.cc',
]

# Add stub sources when building without ncurses
if no_ncurses
  rtorrent_sources += [
    'src/display/curses_stub.cc',
  ]
endif

# ═══════════════════════════════════════════════════════════════════════════════
# Configuration header (config.h)
# ═══════════════════════════════════════════════════════════════════════════════

pkgdatadir = get_option('prefix') / get_option('datadir') / 'rtorrent'

conf_data = configuration_data()
conf_data.set('HAVE_CONFIG_H', 1)
conf_data.set_quoted('PACKAGE', 'rtorrent')
conf_data.set_quoted('PACKAGE_NAME', 'rtorrent')
conf_data.set_quoted('PACKAGE_VERSION', meson.project_version())
conf_data.set_quoted('PACKAGE_STRING', 'rtorrent ' + meson.project_version())
conf_data.set_quoted('PACKAGE_TARNAME', 'rtorrent')
conf_data.set_quoted('PACKAGE_BUGREPORT', 'sundell.software@gmail.com')
conf_data.set_quoted('VERSION', meson.project_version())
conf_data.set_quoted('PACKAGE_URL', '')
conf_data.set('API_VERSION', 22)
conf_data.set_quoted('PEER_NAME', '-lt100E-')
# Use backslash escapes for non-printable bytes in PEER_VERSION
conf_data.set('PEER_VERSION', '"lt\\x10\\x0E"')
conf_data.set('USER_AGENT', 'std::string(PACKAGE "/" VERSION)')
conf_data.set_quoted('PACKAGE_DATADIR', pkgdatadir)

if lua_dep.found()
  conf_data.set('LUA_DATADIR', 'PACKAGE_DATADIR "/lua"')
endif

configure_file(
  input: 'config.h.meson.in',
  output: 'config.h',
  configuration: conf_data,
)

# ═══════════════════════════════════════════════════════════════════════════════
# Include directories
# ═══════════════════════════════════════════════════════════════════════════════

# Include directories
# libtorrent: can only see libtorrent headers (no src/)
libtorrent_inc = include_directories(
  'libtorrent',
  'libtorrent/src',
  'libtorrent/src/torrent',
)
# rtorrent: can see rtorrent headers + libtorrent headers
rtorrent_inc = include_directories(
  'src',
  'libtorrent',
  'libtorrent/src',
  'libtorrent/src/torrent',
)
# Build directory for generated config.h
config_inc = include_directories('.')

# ═══════════════════════════════════════════════════════════════════════════════
# Libraries
# ═══════════════════════════════════════════════════════════════════════════════

lib_deps = [
  thread_dep,
  curl_dep,
  zlib_dep,
  crypto_dep,
  atomic_dep,
]

if ncurses_dep.found()
  lib_deps += ncurses_dep
endif
if lua_dep.found()
  lib_deps += lua_dep
endif
if systemd_dep.found()
  lib_deps += systemd_dep
endif

# libtorrent: can NOT include any rtorrent headers
libtorrent = static_library(
  'torrent',
  libtorrent_sources,
  include_directories: [libtorrent_inc, config_inc],
  dependencies: lib_deps,
  gnu_symbol_visibility: 'hidden',
)

# rtorrent: can include both rtorrent and libtorrent headers
librtorrent = static_library(
  'rtorrent_core',
  rtorrent_sources,
  include_directories: [rtorrent_inc, config_inc],
  dependencies: lib_deps,
  gnu_symbol_visibility: 'hidden',
)

# ═══════════════════════════════════════════════════════════════════════════════
# rtorrent executable
# ═══════════════════════════════════════════════════════════════════════════════

executable(
  'rtorrent',
  'src/main.cc',
  include_directories: [rtorrent_inc, config_inc],
  link_with: [librtorrent, libtorrent],
  dependencies: lib_deps,
  install: true,
)

# ═══════════════════════════════════════════════════════════════════════════════
# Tests
# ═══════════════════════════════════════════════════════════════════════════════

tests_enabled = get_option('tests')
cppunit_dep = disabler()
if tests_enabled
  cppunit_dep = dependency('cppunit', required: true)
  subdir('test')
endif

