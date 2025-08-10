set(_saved_CMAKE_MESSAGE_LOG_LEVEL ${CMAKE_MESSAGE_LOG_LEVEL})
set(CMAKE_MESSAGE_LOG_LEVEL NOTICE)

# Dependency subdirs (only included if NOT using system libs)
foreach(dep IN ITEMS fmt minizip lzma libchdr xxhash imgui simpleini vulkan googletest fast_float reshadefx rcheevos rapidyaml cubeb glad xbyak zydis vixl biscuit riscv-disas d3d12ma winpixeventruntime)
  if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${dep}")
    add_subdirectory(${dep} EXCLUDE_FROM_ALL)
    if(TARGET ${dep})
      disable_compiler_warnings_for_target(${dep})
    endif()
  endif()
endforeach()

set(CMAKE_MESSAGE_LOG_LEVEL ${_saved_CMAKE_MESSAGE_LOG_LEVEL})
