# src/duckstation-mini/CMakeLists.txt

add_executable(duckstation-mini
  mini_host.cpp
  sdl_key_names.h
)

# Compiler and ABI hygiene
set_target_properties(duckstation-mini PROPERTIES
  CXX_VISIBILITY_PRESET hidden
  VISIBILITY_INLINES_HIDDEN ON
)

target_compile_features(duckstation-mini PRIVATE cxx_std_20)

# Includes: inherit from parent + local headers
target_include_directories(duckstation-mini
  PUBLIC  "${CMAKE_CURRENT_SOURCE_DIR}/.."
  PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}"
)

target_link_libraries(duckstation-mini
  PRIVATE
    core
    util
    common
    imgui
    scmversion
    SDL3::SDL3
)

# Optional: pass platform flags for toggleable modules (if defined upstream)
target_compile_definitions(duckstation-mini PRIVATE ENABLE_SDL=1)

# Desktop icon and resource installation (Wayland-safe, Flatpak-ready)
add_core_resources(duckstation-mini)

# 🪟 Platform tweaks: Windows subsystem, manifest, icon
if(WIN32)
  target_sources(duckstation-mini PRIVATE
    duckstation-mini.manifest
    resource.h
  )
  set_target_properties(duckstation-mini PROPERTIES
    WIN32_EXECUTABLE TRUE
    DEBUG_POSTFIX "-debug"
  )
endif()

# 🧩 Optional install hook
if(ALLOW_INSTALL)
  install(TARGETS duckstation-mini RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")
endif()
