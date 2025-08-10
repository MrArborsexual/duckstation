# src/util/CMakeLists.txt

add_library(util STATIC
  audio_stream.cpp
  audio_stream.h
  cd_image.cpp
  cd_image.h
  cd_image_cue.cpp
  cd_image_chd.cpp
  cd_image_device.cpp
  cd_image_hasher.cpp
  cd_image_hasher.h
  cd_image_m3u.cpp
  cd_image_memory.cpp
  cd_image_mds.cpp
  cd_image_pbp.cpp
  cd_image_ppf.cpp
  compress_helpers.cpp
  compress_helpers.h
  cue_parser.cpp
  cue_parser.h
  elf_file.cpp
  elf_file.h
  gpu_device.cpp
  gpu_device.h
  gpu_framebuffer_manager.h
  gpu_shader_cache.cpp
  gpu_shader_cache.h
  gpu_texture.cpp
  gpu_texture.h
  host.cpp
  host.h
  http_downloader.cpp
  http_downloader.h
  image.cpp
  image.h
  imgui_fullscreen.cpp
  imgui_fullscreen.h
  imgui_manager.cpp
  imgui_manager.h
  ini_settings_interface.cpp
  ini_settings_interface.h
  input_manager.cpp
  input_manager.h
  input_source.cpp
  input_source.h
  iso_reader.cpp
  iso_reader.h
  media_capture.cpp
  media_capture.h
  page_fault_handler.cpp
  page_fault_handler.h
  platform_misc.h
  postprocessing.cpp
  postprocessing.h
  postprocessing_shader.cpp
  postprocessing_shader.h
  postprocessing_shader_fx.cpp
  postprocessing_shader_fx.h
  postprocessing_shader_glsl.cpp
  postprocessing_shader_glsl.h
  shadergen.cpp
  shadergen.h
  shiftjis.cpp
  shiftjis.h
  sockets.cpp
  sockets.h
  state_wrapper.cpp
  state_wrapper.h
  texture_decompress.cpp
  texture_decompress.h
  wav_reader_writer.cpp
  wav_reader_writer.h
  window_info.cpp
  window_info.h
)

target_precompile_headers(util PRIVATE "pch.h")

target_compile_features(util PRIVATE cxx_std_20)

target_include_directories(util
  PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/.."
  PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}"
)

target_link_libraries(util
  PUBLIC
    common
    simpleini
    imgui
  PRIVATE
    libchdr
    lzma
    JPEG::JPEG
    PNG::PNG
    WebP::libwebp
    plutosvg::plutosvg
    ZLIB::ZLIB
    SoundTouch::SoundTouchDLL
    xxhash
    zstd::libzstd_shared
    reshadefx
)

set_target_properties(util PROPERTIES
  CXX_VISIBILITY_PRESET hidden
  VISIBILITY_INLINES_HIDDEN ON
)

# ☑ Platform toggles
if(ENABLE_X11)
  target_sources(util PRIVATE
    x11_tools.cpp
    x11_tools.h
  )
  target_compile_definitions(util PRIVATE ENABLE_X11=1)
  target_include_directories(util PRIVATE
    "${X11_xcb_INCLUDE_PATH}"
    "${X11_xcb_randr_INCLUDE_PATH}"
    "${X11_X11_xcb_INCLUDE_PATH}"
  )
endif()

if(ENABLE_WAYLAND)
  target_compile_definitions(util PRIVATE ENABLE_WAYLAND=1)
endif()

if(ENABLE_OPENGL)
  target_compile_definitions(util PUBLIC ENABLE_OPENGL=1)
  target_sources(util PRIVATE
    opengl_context.cpp
    opengl_context.h
    opengl_device.cpp
    opengl_device.h
    opengl_loader.h
    opengl_pipeline.cpp
    opengl_pipeline.h
    opengl_stream_buffer.cpp
    opengl_stream_buffer.h
    opengl_texture.cpp
    opengl_texture.h
  )
  target_link_libraries(util PRIVATE glad)

  if(WIN32)
    target_sources(util PRIVATE
      opengl_context_wgl.cpp
      opengl_context_wgl.h
    )
  endif()

  if(LINUX OR BSD OR ANDROID)
    target_sources(util PRIVATE
      opengl_context_egl.cpp
      opengl_context_egl.h
    )
    target_compile_definitions(util PRIVATE ENABLE_EGL=1)

    if(ENABLE_X11)
      target_sources(util PRIVATE
        opengl_context_egl_xcb.cpp
        opengl_context_egl_xcb.h
        opengl_context_egl_xlib.cpp
        opengl_context_egl_xlib.h
      )
    endif()
    if(ENABLE_WAYLAND)
      target_sources(util PRIVATE
        opengl_context_egl_wayland.cpp
        opengl_context_egl_wayland.h
      )
    endif()
    if(ANDROID)
      target_include_directories(util PRIVATE "${CMAKE_SOURCE_DIR}/android")
    endif()
  endif()

  if(APPLE)
    target_sources(util PRIVATE
      opengl_context_agl.mm
      opengl_context_agl.h
    )
    set_source_files_properties(opengl_context_agl.mm PROPERTIES SKIP_PRECOMPILE_HEADERS TRUE)
  endif()
endif()

if(ENABLE_VULKAN)
  target_compile_definitions(util PUBLIC ENABLE_VULKAN=1)
  target_sources(util PRIVATE
    vulkan_builders.cpp
    vulkan_builders.h
    vulkan_device.cpp
    vulkan_device.h
    vulkan_entry_points.h
    vulkan_entry_points.inl
    vulkan_loader.cpp
    vulkan_loader.h
    vulkan_pipeline.cpp
    vulkan_pipeline.h
    vulkan_stream_buffer.cpp
    vulkan_stream_buffer.h
    vulkan_swap_chain.cpp
    vulkan_swap_chain.h
    vulkan_texture.cpp
    vulkan_texture.h
  )
  target_link_libraries(util PUBLIC vulkan-headers)
endif()

# 📦 Dynamic shaderc/spirv includes only
get_target_property(SHADERC_INCLUDE_DIR Shaderc::shaderc_shared INTERFACE_INCLUDE_DIRECTORIES)
get_target_property(SPIRV_CROSS_INCLUDE_DIR spirv-cross-c-shared INTERFACE_INCLUDE_DIRECTORIES)
target_include_directories(util PUBLIC ${SHADERC_INCLUDE_DIR} ${SPIRV_CROSS_INCLUDE_DIR})

if(NOT ANDROID)
  target_compile_definitions(util PUBLIC ENABLE_SDL)
  target_sources(util PRIVATE
    cubeb_audio_stream.cpp
    sdl_audio_stream.cpp
    sdl_input_source.cpp
    sdl_input_source.h
  )
  if(ENABLE_OPENGL)
    target_sources(util PRIVATE
      opengl_context_sdl.cpp
      opengl_context_sdl.h
    )
  endif()
  target_link_libraries(util PUBLIC
    cubeb
    SDL3::SDL3
  )
  target_include_directories(util PUBLIC ${FFMPEG_INCLUDE_DIRS})
endif()

# 🎮 Platform specifics
if(WIN32)
  target_sources(util PRIVATE
    d3d_common.cpp
    d3d_common.h
    d3d11_device.cpp
    d3d11_device.h
    d3d11_pipeline.cpp
    d3d11_pipeline.h
    d3d11_stream_buffer.cpp
    d3d11_stream_buffer.h
    d3d11_texture.cpp
    d3d11_texture.h
    d3d12_builders.cpp
    d3d12_builders.h
    d3d12_descriptor_heap_manager.cpp
    d3d12_descriptor_heap_manager.h
    d3d12_device.cpp
    d3d12_device.h
    d3d12_pipeline.cpp
    d3d12_pipeline.h
    d3d12_stream_buffer.cpp
    d3d12_stream_buffer.h
    d3d12_texture.cpp
    d3d12_texture.h
    dinput_source.cpp
    dinput_source.h
    http_downloader_winhttp.cpp
    platform_misc_win32.cpp
    win32_raw_input_source.cpp
    win32_raw_input_source.h
    xinput_source.cpp
    xinput_source.h
  )
  target_link_libraries(util PRIVATE
    d3d12ma
    winmm
    Dwmapi
    winhttp
  )
  target_link_libraries(util PRIVATE
    $<$<CONFIG:Debug,Devel>:WinPixEventRuntime::WinPixEventRuntime>
  )
elseif(APPLE)
  include(AddMetalSources)
  set(MAC_SOURCES
    metal_device.h
    metal_device.mm
    metal_layer.h
    metal_stream_buffer.h
    metal_stream_buffer.mm
    platform_misc_mac.mm
  )
  set(METAL_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/metal_shaders.metal")
  set_property(GLOBAL PROPERTY UTIL_METAL_SOURCES ${METAL_SOURCES})
  target_sources(util PRIVATE ${MAC_SOURCES})
  set_source_files_properties(${MAC_SOURCES} PROPERTIES SKIP_PRECOMPILE_HEADERS TRUE)

  find_library(IOK_LIBRARY IOKit REQUIRED)
  find_library(METAL_LIBRARY Metal REQUIRED)
  find_library(QUARTZCORE_LIBRARY QuartzCore REQUIRED)
  target_link_libraries(util PRIVATE ${METAL_LIBRARY} ${QUARTZCORE_LIBRARY} ${IOK_LIBRARY})
elseif(NOT ANDROID)
  target_sources(util PRIVATE platform_misc_unix.cpp)
  find_package(PkgConfig REQUIRED)
  pkg_check_modules(DBUS REQUIRED dbus-1)
  target_include_directories(util PRIVATE ${DBUS_INCLUDE_DIRS})

  if(LINUX)
  target_sources(util PRIVATE platform_misc_linux.cpp)
  target_link_libraries(util PRIVATE UDEV::UDEV)
endif()
