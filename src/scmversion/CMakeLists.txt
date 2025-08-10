# scmversion/CMakeLists.txt

# SCM version generator script — detects Git revision and emits scmversion.cpp
set(SCMVERSION_CPP "${CMAKE_CURRENT_BINARY_DIR}/scmversion.cpp")

if(WIN32)
  add_custom_command(
    OUTPUT ${SCMVERSION_CPP}
    COMMAND cmd /c "${CMAKE_CURRENT_SOURCE_DIR}/gen_scmversion.bat"
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
    VERBATIM
  )
else()
  add_custom_command(
    OUTPUT ${SCMVERSION_CPP}
    COMMAND "${CMAKE_CURRENT_SOURCE_DIR}/gen_scmversion.sh"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/gen_scmversion.sh"
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
    VERBATIM
  )
endif()

# Generate scmversion.cpp before build
add_custom_target(generate_scmversion DEPENDS ${SCMVERSION_CPP})

add_library(scmversion STATIC
  ${SCMVERSION_CPP}
  scmversion.h
)

add_dependencies(scmversion generate_scmversion)

set_target_properties(scmversion PROPERTIES
  CXX_VISIBILITY_PRESET hidden
  VISIBILITY_INLINES_HIDDEN ON
)

target_compile_features(scmversion PRIVATE cxx_std_20)

target_include_directories(scmversion
  PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/.."
  PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}"
)
