# common-tests/CMakeLists.txt

add_executable(common-tests
  bitutils_tests.cpp
  file_system_tests.cpp
  gsvector_yuvtorgb_test.cpp
  path_tests.cpp
  rectangle_tests.cpp
  sha256_tests.cpp
  string_tests.cpp
)

# Enforce C++ standard and visibility for test ABI boundary
set_target_properties(common-tests PROPERTIES
  CXX_VISIBILITY_PRESET hidden
  VISIBILITY_INLINES_HIDDEN ON
)

target_compile_features(common-tests PRIVATE cxx_std_20)

target_include_directories(common-tests
  PUBLIC  "${CMAKE_CURRENT_SOURCE_DIR}/.."
  PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}"
)

target_link_libraries(common-tests
  PRIVATE
    common           # internal logic
    GTest::gtest     # unit test framework
    GTest::gtest_main
)
