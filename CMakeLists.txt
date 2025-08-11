cmake_minimum_required(VERSION 3.19)

# Linux-only, C++20 baseline
project(duckstation LANGUAGES C CXX)

if(WIN32 OR APPLE)
  message(FATAL_ERROR "This fork only supports Linux.")
endif()
if(NOT UNIX)
  message(FATAL_ERROR "This fork requires a UNIX-like system (Linux).")
endif()
# 64-bit only
if(NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
  message(FATAL_ERROR "This fork only supports 64-bit builds.")
endif()

# Enforce out-of-source builds
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_BINARY_DIR)
  message(FATAL_ERROR "In-tree builds are not supported. Use a separate build directory (-B).")
endif()

# Build type default
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Build type" FORCE)
endif()

# Language/flags
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Always enable PIC on Linux and exceptions (SoundTouch and friends require exceptions)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
add_compile_options($<$<CXX_COMPILER_ID:GNU,Clang,AppleClang>:-fexceptions>)
add_compile_options($<$<CXX_COMPILER_ID:GNU,Clang,AppleClang>:-include assert.h>)

# ------------------------------------------------------------------------------
# Options (align with your package.use defaults)
# ------------------------------------------------------------------------------

# Frontends
option(BUILD_QT_FRONTEND       "Build the Qt UI frontend"            ON)
option(BUILD_MINI_FRONTEND     "Build the imgui-based fullscreen UI" OFF)  # aka 'big picture'
option(BUILD_REGTEST           "Build regression test frontend"      OFF)
option(BUILD_TESTS             "Enable unit tests"                    OFF)

# Renderers
option(ENABLE_OPENGL           "Enable OpenGL renderer"               ON)
option(ENABLE_VULKAN           "Enable Vulkan renderer"               ON)

# Media/features
option(ENABLE_FFMPEG           "Enable FFmpeg capture/encode"         ON)
option(ENABLE_RETROACHIEVEMENTS "Enable RetroAchievements"            OFF)
option(ENABLE_DISCORD_PRESENCE "Enable Discord Rich Presence"         ON)

# Input/window stacks
option(ENABLE_SDL2             "Use SDL2 for input/windowing"         OFF)
option(ENABLE_SDL3             "Use SDL3 for input/windowing"         ON)

# Platform libs/toggles (flow mostly through Qt/SDL, but we expose them explicitly)
option(ENABLE_WAYLAND          "Enable Wayland support"               ON)
option(ENABLE_X11              "Enable X11 support"                   ON)
option(ENABLE_EGL              "Enable EGL"                           ON)
option(ENABLE_GBM              "Enable GBM"                           ON)
option(ENABLE_EVDEV            "Enable libevdev"                      ON)
option(ENABLE_UDEV             "Enable libudev"                       ON)

# Audio backends
option(ENABLE_ALSA             "Enable ALSA"                          ON)
option(ENABLE_PULSEAUDIO       "Enable PulseAudio"                    ON)
option(ENABLE_PIPEWIRE         "Enable PipeWire"                      ON)

# Other system libs
option(ENABLE_CURL             "Use libcurl for networking"           ON)
option(ENABLE_ZSTD             "Use zstd compression"                 ON)
option(ENABLE_BACKTRACE        "Use libbacktrace"                     ON)

# Sanity checks roughly reflecting REQUIRED_USE semantics
if(NOT BUILD_QT_FRONTEND AND NOT BUILD_MINI_FRONTEND)
  message(FATAL_ERROR "At least one of BUILD_QT_FRONTEND or BUILD_MINI_FRONTEND must be ON.")
endif()
if(ENABLE_SDL2 AND ENABLE_SDL3)
  message(FATAL_ERROR "ENABLE_SDL2 and ENABLE_SDL3 are mutually exclusive. Choose one.")
endif()

# ------------------------------------------------------------------------------
# Helpers to create imported targets from pkg-config where CMake packages are absent
# ------------------------------------------------------------------------------

find_package(PkgConfig REQUIRED)

function(ds_import_from_pkg pkg target)
  # Creates an INTERFACE IMPORTED target named ${target} from pkg-config ${pkg}
  pkg_check_modules(_PC REQUIRED ${pkg})
  add_library(${target} INTERFACE IMPORTED)
  set_target_properties(${target} PROPERTIES
    INTERFACE_INCLUDE_DIRECTORIES "${_PC_INCLUDE_DIRS}"
    INTERFACE_LINK_LIBRARIES      "${_PC_LINK_LIBRARIES}"
    INTERFACE_COMPILE_OPTIONS     "${_PC_CFLAGS_OTHER}"
  )
endfunction()

function(ds_import_from_pkg_optional pkg target out_var)
  # Optional variant; sets ${out_var} to TRUE/FALSE
  pkg_check_modules(_PC QUIET ${pkg})
  if(_PC_FOUND)
    add_library(${target} INTERFACE IMPORTED)
    set_target_properties(${target} PROPERTIES
      INTERFACE_INCLUDE_DIRECTORIES "${_PC_INCLUDE_DIRS}"
      INTERFACE_LINK_LIBRARIES      "${_PC_LINK_LIBRARIES}"
      INTERFACE_COMPILE_OPTIONS     "${_PC_CFLAGS_OTHER}"
    )
    set(${out_var} TRUE PARENT_SCOPE)
  else()
    set(${out_var} FALSE PARENT_SCOPE)
  endif()
endfunction()

# ------------------------------------------------------------------------------
# Core third-party dependencies (system-first)
# Expose canonical targets that subdirs can link against.
# ------------------------------------------------------------------------------

# fmt (CMake config widely available)
find_package(fmt REQUIRED)
add_library(ds::fmt ALIAS fmt::fmt)

# xxhash (pkg-config: libxxhash)
ds_import_from_pkg(libxxhash ds::xxhash)

# simpleini (header-only, uncommon on system; if absent, future subdir can vendor)
# Provide a lightweight header-only INTERFACE if found via pkg-config, otherwise leave out.
# If you want to force system-only, uncomment the REQUIRED-style shim.
# ds_import_from_pkg(simpleini ds::simpleini)  # Uncomment if you ship a .pc for it

# imgui (Gentoo provides dev-libs/imgui with pkg-config 'imgui' in some overlays)
if(BUILD_MINI_FRONTEND)
  ds_import_from_pkg_optional(imgui ds::imgui HAVE_IMGUI)
  if(NOT HAVE_IMGUI)
    message(FATAL_ERROR "BUILD_MINI_FRONTEND=ON but pkg-config 'imgui' not found.")
  endif()
endif()

# ReShade FX (no standard system package; keep for future if you externalize it)
# ds_import_from_pkg_optional(reshadefx ds::reshadefx HAVE_RESHADEFX)

# rcheevos (retroachievements library; if you intend system usage)
if(ENABLE_RETROACHIEVEMENTS)
  ds_import_from_pkg_optional(rcheevos ds::rcheevos HAVE_RCHEEVOS)
  if(NOT HAVE_RCHEEVOS)
    message(FATAL_ERROR "RetroAchievements requested but 'rcheevos' not found via pkg-config.")
  endif()
endif()

# rapidyaml (ryml) if system-provided
ds_import_from_pkg_optional(ryml ds::ryml HAVE_RYML)

# cubeb (audio backend)
ds_import_from_pkg_optional(cubeb ds::cubeb HAVE_CUBEB)

# SoundTouch (pkg-config: soundtouch)
ds_import_from_pkg(soundtouch ds::soundtouch)

# PlutoSVG (pkg-config: plutosvg>=0.0.6)
ds_import_from_pkg(plutosvg ds::plutosvg)

# WebP (CMake config sometimes available; fallback to pkg-config)
find_package(WebP QUIET)
if(WebP_FOUND)
  add_library(ds::webp ALIAS WebP::libwebp)
else()
  ds_import_from_pkg(libwebp ds::webp)
endif()

# FreeType
find_package(Freetype REQUIRED)
add_library(ds::freetype ALIAS Freetype::Freetype)

# Shaderc (CMake config sometimes available; fallback to pkg-config)
find_package(Shaderc QUIET)
if(Shaderc_FOUND)
  # Try to prefer shared if available
  if(TARGET Shaderc::shaderc_shared)
    add_library(ds::shaderc ALIAS Shaderc::shaderc_shared)
  else()
    add_library(ds::shaderc ALIAS Shaderc::shaderc)
  endif()
else()
  ds_import_from_pkg(shaderc ds::shaderc)
endif()

# SPIRV-Cross C API (pkg-config: spirv-cross-c)
ds_import_from_pkg(spirv-cross-c ds::spirv_cross_c)

# Alias expected upstream targets for compatibility
if(TARGET ds::shaderc)
  add_library(Shaderc::shaderc_shared ALIAS ds::shaderc)
endif()

if(TARGET ds::spirv_cross_c)
  add_library(spirv-cross-c-shared ALIAS ds::spirv_cross_c)
endif()

# Vulkan and OpenGL
if(ENABLE_VULKAN)
  find_package(Vulkan REQUIRED) # provides Vulkan::Vulkan
  add_library(ds::vulkan ALIAS Vulkan::Vulkan)
endif()
if(ENABLE_OPENGL)
  find_package(OpenGL REQUIRED) # provides OpenGL::GL
  add_library(ds::opengl ALIAS OpenGL::GL)
endif()

# FFmpeg (media-video/ffmpeg exports many .pc files)
if(ENABLE_FFMPEG)
  ds_import_from_pkg(libavcodec   ds::avcodec)
  ds_import_from_pkg(libavformat  ds::avformat)
  ds_import_from_pkg(libavutil    ds::avutil)
  ds_import_from_pkg(libswscale   ds::swscale)
  ds_import_from_pkg(libswresample ds::swresample)
endif()

# Discord RPC (no canonical CMake/pc everywhere -> roll our own find)
if(ENABLE_DISCORD_PRESENCE)
  find_path(DISCORDRPC_INCLUDE_DIR NAMES discord_rpc.h)
  find_library(DISCORDRPC_LIBRARY NAMES discord-rpc)
  if(NOT DISCORDRPC_INCLUDE_DIR OR NOT DISCORDRPC_LIBRARY)
    message(FATAL_ERROR "DiscordRPC requested but not found (discord_rpc.h / libdiscord-rpc).")
  endif()
  add_library(ds::discordrpc INTERFACE IMPORTED)
  set_target_properties(ds::discordrpc PROPERTIES
    INTERFACE_INCLUDE_DIRECTORIES "${DISCORDRPC_INCLUDE_DIR}"
    INTERFACE_LINK_LIBRARIES      "${DISCORDRPC_LIBRARY}"
  )
endif()

# curl
if(ENABLE_CURL)
  find_package(CURL REQUIRED)
  add_library(ds::curl ALIAS CURL::libcurl)
endif()

# zstd
if(ENABLE_ZSTD)
  find_package(ZSTD QUIET)
  if(ZSTD_FOUND)
    add_library(ds::zstd ALIAS ZSTD::ZSTD)
  else()
    ds_import_from_pkg(libzstd ds::zstd)
  endif()
endif()

# backtrace (libbacktrace rarely ships CMake config; use find_library)
if(ENABLE_BACKTRACE)
  find_library(BACKTRACE_LIB NAMES backtrace)
  if(NOT BACKTRACE_LIB)
    message(FATAL_ERROR "libbacktrace requested but not found.")
  endif()
  add_library(ds::backtrace INTERFACE IMPORTED)
  set_target_properties(ds::backtrace PROPERTIES
    INTERFACE_LINK_LIBRARIES "${BACKTRACE_LIB}"
  )
endif()

# ------------------------------------------------------------------------------
# Platform stacks (SDL, Wayland/X11, audio)
# ------------------------------------------------------------------------------

# SDL
if(ENABLE_SDL3)
  find_package(SDL3 REQUIRED)  # Provides SDL3::SDL3
  add_library(ds_sdl INTERFACE)
  target_link_libraries(ds_sdl INTERFACE SDL3::SDL3)
elseif(ENABLE_SDL2)
  find_package(SDL2 REQUIRED)  # Provides SDL2::SDL2
  add_library(ds_sdl ALIAS SDL2::SDL2)
endif()

# Wayland / X11 / EGL / GBM
if(ENABLE_WAYLAND)
  ds_import_from_pkg(wayland-client ds::wayland_client)
  ds_import_from_pkg_optional(wayland-egl ds::wayland_egl HAVE_WAYLAND_EGL)
endif()
if(ENABLE_X11)
  find_package(X11 REQUIRED)
  add_library(ds::x11 ALIAS X11::X11)
endif()
if(ENABLE_EGL)
  # Some systems offer FindEGL, others only pkg-config 'egl'
  find_package(EGL QUIET)
  if(TARGET EGL::EGL)
    add_library(ds::egl ALIAS EGL::EGL)
  else()
    ds_import_from_pkg(egl ds::egl)
  endif()
endif()
if(ENABLE_GBM)
  ds_import_from_pkg(gbm ds::gbm)
endif()
if(ENABLE_EVDEV)
  ds_import_from_pkg(libevdev ds::evdev)
endif()
if(ENABLE_UDEV)
  ds_import_from_pkg(libudev ds::udev)
endif()

# Audio stacks
if(ENABLE_ALSA)
  ds_import_from_pkg(alsa ds::alsa)
endif()
if(ENABLE_PULSEAUDIO)
  ds_import_from_pkg(libpulse ds::pulse)
endif()
if(ENABLE_PIPEWIRE)
  ds_import_from_pkg(libpipewire-0.3 ds::pipewire)
endif()

# ------------------------------------------------------------------------------
# Include your source tree (new CMakeLists you will author per component)
# These subdirectories should link to the ds::<name> imported targets defined above.
# ------------------------------------------------------------------------------

# Core emulator logic (always needed)
add_subdirectory(src/common)
add_subdirectory(src/util)
add_subdirectory(src/core)

# Optional frontends
if(BUILD_QT_FRONTEND)
  add_subdirectory(src/duckstation-qt)
endif()

if(BUILD_MINI_FRONTEND)
  add_subdirectory(src/duckstation-mini)
endif()

# Optional CLI / automation utilities
if(BUILD_REGTEST)
  add_subdirectory(src/duckstation-regtest)
endif()

# Optional test suite
if(BUILD_TESTS)
  add_subdirectory(tests)
endif()


# ------------------------------------------------------------------------------
# Summary
# ------------------------------------------------------------------------------

message(STATUS "==== DuckStation (Linux, 64-bit, system-libs-first) ====")
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "C++ standard: C++${CMAKE_CXX_STANDARD}")
message(STATUS "Frontends: QT=${BUILD_QT_FRONTEND} MINI=${BUILD_MINI_FRONTEND} REGTEST=${BUILD_REGTEST}")
message(STATUS "Renderers: OpenGL=${ENABLE_OPENGL} Vulkan=${ENABLE_VULKAN}")
message(STATUS "Media: FFmpeg=${ENABLE_FFMPEG} RetroAch=${ENABLE_RETROACHIEVEMENTS} Discord=${ENABLE_DISCORD_PRESENCE}")
message(STATUS "SDL: SDL3=${ENABLE_SDL3} SDL2=${ENABLE_SDL2}")
message(STATUS "Wayland=${ENABLE_WAYLAND} X11=${ENABLE_X11} EGL=${ENABLE_EGL} GBM=${ENABLE_GBM} EVDEV=${ENABLE_EVDEV} UDEV=${ENABLE_UDEV}")
message(STATUS "Audio: ALSA=${ENABLE_ALSA} Pulse=${ENABLE_PULSEAUDIO} PipeWire=${ENABLE_PIPEWIRE}")
message(STATUS "Other: CURL=${ENABLE_CURL} ZSTD=${ENABLE_ZSTD} BACKTRACE=${ENABLE_BACKTRACE}")
