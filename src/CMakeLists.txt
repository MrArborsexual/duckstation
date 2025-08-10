# src/CMakeLists.txt

add_subdirectory(common)
add_subdirectory(util)
add_subdirectory(core)

if(BUILD_QT_FRONTEND)
  add_subdirectory(duckstation-qt)
endif()

if(BUILD_MINI_FRONTEND)
  add_subdirectory(duckstation-mini)
endif()

if(BUILD_REGTEST)
  add_subdirectory(duckstation-regtest)
endif()

if(BUILD_TESTS)
  add_subdirectory(tests)
endif()

