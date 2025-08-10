# updater/CMakeLists.txt

add_executable(updater
  updater.cpp
  updater.h
)

# C++ standard + ABI hygiene
target_compile_features(updater PRIVATE cxx_std_20)

set_target_properties(updater PROPERTIES
  CXX_VISIBILITY_PRESET hidden
  VISIBILITY_INLINES_HIDDEN ON
)

target_include_directories(updater
  PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/.."
  PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}"
)

target_link_libraries(updater
  PRIVATE
    common
    minizip
    ZLIB::ZLIB
)

# 🪟 Windows subsystem
if(WIN32)
  target_sources(updater PRIVATE
    updater.rc
    win32_main.cpp
    win32_progress_callback.cpp
    win32_progress_callback.h
    ../duckstation-qt/vcruntimecheck.cpp
  )
  target_link_libraries(updater PRIVATE Comctl32.lib)

  set_target_properties(updater PROPERTIES
    WIN32_EXECUTABLE TRUE
    DEBUG_POSTFIX "-debug"
  )
endif()

# 🍎 macOS bundle setup
if(APPLE)
  set(MAC_SOURCES
    cocoa_main.mm
    cocoa_progress_callback.mm
    cocoa_progress_callback.h
  )
  target_sources(updater PRIVATE ${MAC_SOURCES})
  set_source_files_properties(${MAC_SOURCES} PROPERTIES SKIP_PRECOMPILE_HEADERS TRUE)

  find_library(COCOA_LIBRARY Cocoa REQUIRED)
  target_link_libraries(updater PRIVATE ${COCOA_LIBRARY})

  if(NOT CMAKE_GENERATOR MATCHES "Xcode" AND NOT SKIP_POSTPROCESS_BUNDLE)
    set_target_properties(updater PROPERTIES
      MACOSX_BUNDLE TRUE
      MACOSX_BUNDLE_INFO_PLIST "${CMAKE_CURRENT_SOURCE_DIR}/Info.plist.in"
      OUTPUT_NAME Updater
      RUNTIME_OUTPUT_DIRECTORY "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/DuckStation.app/Contents/Resources"
    )
    target_sources(updater PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/Updater.icns")
    set_source_files_properties("${CMAKE_CURRENT_SOURCE_DIR}/Updater.icns" PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
  endif()
endif()

# 📦 Optional install hook (toggleable via USE=-update)
if(ALLOW_INSTALL)
  install(TARGETS updater RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")
endif()
