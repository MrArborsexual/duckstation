# duckstation-regtest/CMakeLists.txt

add_executable(duckstation-regtest
  regtest_host.cpp
)

set_target_properties(duckstation-regtest PROPERTIES
  CXX_VISIBILITY_PRESET hidden
  VISIBILITY_INLINES_HIDDEN ON
)

target_compile_features(duckstation-regtest PRIVATE cxx_std_20)

target_include_directories(duckstation-regtest
  PUBLIC  "${CMAKE_CURRENT_SOURCE_DIR}/.."
  PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}"
)

target_link_libraries(duckstation-regtest
  PRIVATE
    core
    common
    scmversion
)

add_core_resources(duckstation-regtest)
