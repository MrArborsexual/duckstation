# src/duckstation-qt/CMakeLists.txt
find_package(Qt6 REQUIRED COMPONENTS LinguistTools)

set(CMAKE_AUTOUIC OFF)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(SRCS
  aboutdialog.cpp
  aboutdialog.h
  aboutdialog.ui
  achievementlogindialog.cpp
  achievementlogindialog.h
  achievementlogindialog.ui
  achievementsettingswidget.cpp
  achievementsettingswidget.h
  achievementsettingswidget.ui
  advancedsettingswidget.cpp
  advancedsettingswidget.h
  advancedsettingswidget.ui
  audiosettingswidget.cpp
  audiosettingswidget.h
  audiosettingswidget.ui
  audiostretchsettingsdialog.ui
  autoupdaterwindow.cpp
  autoupdaterwindow.h
  autoupdaterwindow.ui
  biossettingswidget.cpp
  biossettingswidget.h
  biossettingswidget.ui
  colorpickerbutton.cpp
  colorpickerbutton.h
  consolesettingswidget.cpp
  consolesettingswidget.h
  consolesettingswidget.ui
  controllerbindingwidget_analog_controller.ui
  controllerbindingwidget_analog_joystick.ui
  controllerbindingwidget_digital_controller.ui
  controllerbindingwidget_guncon.ui
  controllerbindingwidget_justifier.ui
  controllerbindingwidget_mouse.ui
  controllerbindingwidget_negcon.ui
  controllerbindingwidget_negconrumble.ui
  controllerbindingwidgets.cpp
  controllerbindingwidgets.h
  controllerbindingwidget.ui
  controllerglobalsettingswidget.cpp
  controllerglobalsettingswidget.h
  controllerglobalsettingswidget.ui
  controllerledsettingsdialog.ui
  controllermacroeditwidget.ui
  controllermacrowidget.ui
  controllersettingswindow.cpp
  controllersettingswindow.h
  controllersettingswindow.ui
  controllersettingwidgetbinder.h
  coverdownloadwindow.cpp
  coverdownloadwindow.h
  coverdownloadwindow.ui
  debuggeraddbreakpointdialog.ui
  debuggermodels.cpp
  debuggermodels.h
  debuggerwindow.cpp
  debuggerwindow.h
  debuggerwindow.ui
  displaywidget.cpp
  displaywidget.h
  emptygamelistwidget.ui
  emulationsettingswidget.cpp
  emulationsettingswidget.h
  emulationsettingswidget.ui
  foldersettingswidget.cpp
  foldersettingswidget.h
  foldersettingswidget.ui
  gamecheatcodechoiceeditordialog.ui
  gamecheatcodeeditordialog.ui
  gamecheatsettingswidget.cpp
  gamecheatsettingswidget.h
  gamecheatsettingswidget.ui
  gamepatchdetailswidget.ui
  gamepatchsettingswidget.cpp
  gamepatchsettingswidget.h
  gamepatchsettingswidget.ui
  gamelistrefreshthread.cpp
  gamelistrefreshthread.h
  gamelistsettingswidget.cpp
  gamelistsettingswidget.h
  gamelistsettingswidget.ui
  gamelistwidget.cpp
  gamelistwidget.h
  gamelistwidget.ui
  gamesummarywidget.cpp
  gamesummarywidget.h
  gamesummarywidget.ui
  graphicssettingswidget.cpp
  graphicssettingswidget.h
  graphicssettingswidget.ui
  hotkeysettingswidget.cpp
  hotkeysettingswidget.h
  inputbindingdialog.cpp
  inputbindingdialog.h
  inputbindingdialog.ui
  inputbindingwidgets.cpp
  inputbindingwidgets.h
  interfacesettingswidget.cpp
  interfacesettingswidget.h
  interfacesettingswidget.ui
  isobrowserwindow.cpp
  isobrowserwindow.h
  isobrowserwindow.ui
  logwindow.cpp
  logwindow.h
  mainwindow.cpp
  mainwindow.h
  mainwindow.ui
  memorycardeditorwindow.cpp
  memorycardeditorwindow.h
  memorycardeditorwindow.ui
  memorycardrenamefiledialog.ui
  memorycardsettingswidget.cpp
  memorycardsettingswidget.h
  memoryscannerwindow.cpp
  memoryscannerwindow.h
  memoryscannerwindow.ui
  memoryviewwidget.cpp
  memoryviewwidget.h
  postprocessingsettingswidget.cpp
  postprocessingsettingswidget.h
  postprocessingchainconfigwidget.ui
  postprocessingoverlayconfigwidget.ui
  qthost.cpp
  qthost.h
  qtkeycodes.cpp
  qtprogresscallback.cpp
  qtprogresscallback.h
  qtthemes.cpp
  qttranslations.inl
  qtutils.cpp
  qtutils.h
  resource.h
  selectdiscdialog.cpp
  selectdiscdialog.h
  selectdiscdialog.ui
  settingswindow.cpp
  settingswindow.h
  settingswindow.ui
  settingwidgetbinder.h
  setupwizarddialog.cpp
  setupwizarddialog.h
  setupwizarddialog.ui
  texturereplacementsettingsdialog.ui
)

set(TS_FILES
  translations/duckstation-qt_de.ts
  translations/duckstation-qt_en.ts
  translations/duckstation-qt_es.ts
  translations/duckstation-qt_es-ES.ts
  translations/duckstation-qt_fr.ts
  translations/duckstation-qt_he.ts
  translations/duckstation-qt_id.ts
  translations/duckstation-qt_it.ts
  translations/duckstation-qt_ja.ts
  translations/duckstation-qt_ko.ts
  translations/duckstation-qt_nl.ts
  translations/duckstation-qt_pl.ts
  translations/duckstation-qt_pt-BR.ts
  translations/duckstation-qt_pt-PT.ts
  translations/duckstation-qt_ru.ts
  translations/duckstation-qt_sv.ts
  translations/duckstation-qt_tr.ts
  translations/duckstation-qt_zh-CN.ts
)

qt_add_lrelease(duckstation-qt TS_FILES ${TS_FILES} QM_FILES_OUTPUT_VARIABLE QM_FILES)

add_executable(duckstation-qt ${SRCS} ${QM_FILES})

target_compile_features(duckstation-qt PRIVATE cxx_std_20)

target_precompile_headers(duckstation-qt PRIVATE "pch.h")

target_include_directories(duckstation-qt
  PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/.."
  PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}"
          "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>"
)

target_compile_definitions(duckstation-qt PRIVATE QT_NO_EXCEPTIONS)

target_link_libraries(duckstation-qt
  PRIVATE
    core
    common
    imgui
    minizip
    scmversion
    Qt6::Core
    Qt6::Gui
    Qt6::Widgets
)

set_target_properties(duckstation-qt PROPERTIES
  CXX_VISIBILITY_PRESET hidden
  VISIBILITY_INLINES_HIDDEN ON
)

add_core_resources(duckstation-qt)

# UI wrapping (manual to avoid full rebuilds)
set(UI_FILES "")
foreach(src ${SRCS})
  if(src MATCHES "\\.ui$")
    list(APPEND UI_FILES ${src})
  endif()
endforeach()
qt_wrap_ui(UI_SRCS SOURCES ${UI_FILES})
target_sources(duckstation-qt PRIVATE ${UI_SRCS})

# Qt resource compilation
set(RCC_FILE "${CMAKE_CURRENT_BINARY_DIR}/duckstation-qt.rcc")
qt_add_binary_resources(duckstation-qt-rcc resources/duckstation-qt.qrc DESTINATION ${RCC_FILE} OPTIONS -no-compress)
add_dependencies(duckstation-qt duckstation-qt-rcc)
target_sources(duckstation-qt PRIVATE ${RCC_FILE})

# Translation install/copy
set(QM_OUTPUT_DIR "$<TARGET_FILE_DIR:duckstation-qt>/translations")
add_custom_command(TARGET duckstation-qt POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E make_directory "${QM_OUTPUT_DIR}"
)
foreach(QM_FILE IN LISTS QM_FILES)
  get_filename_component(QM_FILE_NAME ${QM_FILE} NAME)
  add_custom_command(TARGET duckstation-qt POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different "${QM_FILE}" "${QM_OUTPUT_DIR}/${QM_FILE_NAME}"
  )
  if(ALLOW_INSTALL)
    install(FILES "${QM_FILE}" DESTINATION "${CMAKE_INSTALL_BINDIR}/translations")
  endif()
endforeach()

# Platform-specific tweaks
if(WIN32)
  target_sources(duckstation-qt PRIVATE duckstation-qt.rc vcruntimecheck.cpp)
  set_target_properties(duckstation-qt PROPERTIES WIN32_EXECUTABLE TRUE DEBUG_POSTFIX "-debug")
elseif(APPLE)
  set_target_properties(duckstation-qt PROPERTIES
    MACOSX_BUNDLE TRUE
    MACOSX_BUNDLE_INFO_PLIST "${CMAKE_CURRENT_SOURCE_DIR}/Info.plist.in"
    OUTPUT_NAME DuckStation
  )
  target_sources(duckstation-qt PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/DuckStation.icns")
  set_source_files_properties("${CMAKE_CURRENT_SOURCE_DIR}/DuckStation.icns" PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
endif()

if(ALLOW_INSTALL)
  install(TARGETS duckstation-qt RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")
endif()

copy_base_translations(duckstation-qt)
