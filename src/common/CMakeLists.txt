# src/common/CMakeLists.txt

add_library(common STATIC
  # ░░ Core error/reporting & assertion utilities ░░
  assert.cpp
  assert.h
  error.cpp
  error.h
  log.cpp
  log.h
  log_channels.h
  perf_scope.cpp
  perf_scope.h
  progress_callback.cpp
  progress_callback.h

  # ░░ Assertions, crash handlers, fastjmp ░░
  crash_handler.cpp
  crash_handler.h
  fastjmp.cpp
  fastjmp.h

  # ░░ File I/O and binary utilities ░░
  binary_reader_writer.cpp
  binary_reader_writer.h
  file_system.cpp
  file_system.h
  path.h

  # ░░ Digest and hash utils ░░
  md5_digest.cpp
  md5_digest.h
  sha1_digest.cpp
  sha1_digest.h
  sha256_digest.cpp
  sha256_digest.h
  hash_combine.h
  xorshift_prng.h

  # ░░ Threading and task primitives ░░
  threading.cpp
  threading.h
  task_queue.cpp
  task_queue.h
  scoped_guard.h

  # ░░ Settings and layered config interfaces ░░
  layered_settings_interface.cpp
  layered_settings_interface.h
  memory_settings_interface.cpp
  memory_settings_interface.h
  settings_interface.h

  # ░░ Small string and string utilities ░░
  small_string.cpp
  small_string.h
  string_util.cpp
  string_util.h

  # ░░ Time and performance ░░
  timer.cpp
  timer.h

  # ░░ Math, SIMD, and vector formats ░░
  gsvector.cpp
  gsvector.h
  gsvector_formatter.h
  gsvector_neon.h
  gsvector_nosimd.h
  gsvector_sse.h
  dimensional_array.h

  # ░░ Low-level memory and type utilities ░░
  align.h
  intrin.h
  heap_array.h
  types.h
  bitfield.h
  bitutils.h
  ryml_helpers.h

  # ░░ OS abstraction ░░
  dynamic_library.cpp
  dynamic_library.h
  memmap.cpp
  memmap.h
  minizip_helpers.h

  # ░░ Meta container utilities ░░
  heterogeneous_containers.h

  # ░░ Third-party embedded libs ░░
  thirdparty/SmallVector.cpp
  thirdparty/SmallVector.h
  thirdparty/aes.cpp
  thirdparty/aes.h
)

target_precompile_headers(common PRIVATE "${PROJECT_SOURCE_DIR}/src/common/pch.h")

target_include_directories(common
  PUBLIC "${PROJECT_SOURCE_DIR}/src"
  PUBLIC "${PROJECT_SOURCE_DIR}/src/common"
)

target_compile_features(common PRIVATE cxx_std_20)

target_compile_options(common PRIVATE
  $<$<CXX_COMPILER_ID:GNU,Clang,AppleClang>:-fexceptions>
)

target_link_libraries(common PRIVATE ds::fmt)

set_target_properties(common PROPERTIES
  CXX_VISIBILITY_PRESET hidden
  VISIBILITY_INLINES_HIDDEN ON
)
