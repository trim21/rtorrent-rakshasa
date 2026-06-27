# ── rtorrent tests ──

rt_test_inc = [
  include_directories(
    '.',
    '..',
    '../src',
    '../libtorrent',
    '../libtorrent/src',
    '../libtorrent/src/torrent',
  ),
  config_inc,
]

rt_test_common = files(
  'main.cc',
  'helpers/mock_function.cc',
  'helpers/progress_listener.cc',
  'helpers/protectors.cc',
  'helpers/test_fixture.cc',
  'helpers/test_main_thread.cc',
  'helpers/test_thread.cc',
)

rt_test_deps = [
  cppunit_dep,
  dependency('threads'),
]

# rtorrent_Test_Rpc
rtorrent_test_rpc = executable(
  'rtorrent_Test_Rpc',
  rt_test_common,
  'rpc/test_command.cc',
  'rpc/test_command_map.cc',
  'rpc/test_jsonrpc.cc',
  'rpc/test_xmlrpc.cc',
  'rpc/test_command_slot.cc',
  'rpc/test_object_storage.cc',
  'rpc/test_parse_options.cc',
  include_directories: rt_test_inc,
  link_with: [librtorrent, libtorrent],
  dependencies: rt_test_deps + lib_deps,
)

# rtorrent_Test_Src
rtorrent_test_src = executable(
  'rtorrent_Test_Src',
  rt_test_common,
  'src/test_command_dynamic.cc',
  'src/test_watch_ready_queue.cc',
  include_directories: rt_test_inc,
  link_with: [librtorrent, libtorrent],
  dependencies: rt_test_deps + lib_deps,
)

test('rtorrent_Test_Rpc', rtorrent_test_rpc)
test('rtorrent_Test_Src', rtorrent_test_src)

# ── libtorrent tests ──
subdir('libtorrent')
