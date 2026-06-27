# ── libtorrent tests ──

lt_test_dir = '../../libtorrent/test'

# Include order: libtorrent paths must come first so libtorrent headers
# shadow rtorrent's (e.g. utils/functional.h).
lt_test_inc = include_directories(
  '../../libtorrent/test',
  '../../libtorrent',
  '../../libtorrent/src',
  '.',
  '..',
  '../..',
  '../../src',
)

lt_test_common = files(
  lt_test_dir / 'main.cc',
  lt_test_dir / 'helpers/mock_function.cc',
  lt_test_dir / 'helpers/network.cc',
  lt_test_dir / 'helpers/progress_listener.cc',
  lt_test_dir / 'helpers/protectors.cc',
  lt_test_dir / 'helpers/test_fixture.cc',
  lt_test_dir / 'helpers/test_main_thread.cc',
  lt_test_dir / 'helpers/test_thread.cc',
  lt_test_dir / 'helpers/tracker_test.cc',
)

lt_test_deps = [
  cppunit_dep,
  dependency('threads'),
]

lt_test_cpp_args = ['-DLT_CPPUNIT_TESTING']

# Helper to define a libtorrent test executable
lt_tests = {
  'LibTorrent_Test_Torrent_Net': [
    lt_test_dir / 'torrent/net/test_address_info.cc',
    lt_test_dir / 'torrent/net/test_fd.cc',
    lt_test_dir / 'torrent/net/test_socket_address.cc',
  ],
  'LibTorrent_Test_Torrent_Utils': [
    lt_test_dir / 'torrent/utils/test_extents.cc',
    lt_test_dir / 'torrent/utils/test_log.cc',
    lt_test_dir / 'torrent/utils/test_log_buffer.cc',
    lt_test_dir / 'torrent/utils/directory_events_test.cc',
    lt_test_dir / 'torrent/utils/test_option_strings.cc',
    lt_test_dir / 'torrent/utils/test_queue_buckets.cc',
    lt_test_dir / 'torrent/utils/test_thread_base.cc',
    lt_test_dir / 'torrent/utils/test_uri_parser.cc',
  ],
  'LibTorrent_Test_Torrent': [
    lt_test_dir / 'torrent/object_test.cc',
    lt_test_dir / 'torrent/object_test_utils.cc',
    lt_test_dir / 'torrent/object_static_map_test.cc',
    lt_test_dir / 'torrent/object_stream_test.cc',
    lt_test_dir / 'torrent/test_tracker_controller.cc',
    lt_test_dir / 'torrent/test_tracker_controller_features.cc',
    lt_test_dir / 'torrent/test_tracker_controller_requesting.cc',
    lt_test_dir / 'torrent/test_tracker_list.cc',
    lt_test_dir / 'torrent/test_tracker_list_features.cc',
    lt_test_dir / 'torrent/test_tracker_timeout.cc',
  ],
  'LibTorrent_Test_Data': [
    lt_test_dir / 'data/test_chunk_list.cc',
    lt_test_dir / 'data/test_hash_check_queue.cc',
    lt_test_dir / 'data/test_hash_queue.cc',
  ],
  'LibTorrent_Test_Net': [],
  'LibTorrent_Test_Tracker': [
    lt_test_dir / 'tracker/test_tracker_http.cc',
  ],
  'LibTorrent_Test': [
    lt_test_dir / 'rak/ranges_test.cc',
    lt_test_dir / 'protocol/test_request_list.cc',
  ],
}

foreach name, extra_src : lt_tests
  exe = executable(
    name,
    lt_test_common,
    extra_src,
    include_directories: lt_test_inc,
    link_with: [librtorrent, libtorrent],
    dependencies: lt_test_deps + lib_deps,
    cpp_args: lt_test_cpp_args,
  )
  test(name, exe)
endforeach
