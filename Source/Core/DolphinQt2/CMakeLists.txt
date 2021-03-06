find_package(Qt5 REQUIRED COMPONENTS Gui Widgets)

set_property(TARGET Qt5::Core PROPERTY INTERFACE_COMPILE_FEATURES "")
message(STATUS "Found Qt version ${Qt5Core_VERSION}")

set(CMAKE_AUTOMOC ON)

add_executable(dolphin-emu-qt2
  AboutDialog.cpp
  FIFOPlayerWindow.cpp
  HotkeyScheduler.cpp
  Host.cpp
  Main.cpp
  MainWindow.cpp
  MenuBar.cpp
  RenderWidget.cpp
  Resources.cpp
  SearchBar.cpp
  Settings.cpp
  ToolBar.cpp
  Translation.cpp
  WiiUpdate.cpp
  WiiUpdate.h
  Config/CheatCodeEditor.cpp
  Config/ARCodeWidget.cpp
  Config/CheatWarningWidget.cpp
  Config/ControllersWindow.cpp
  Config/FilesystemWidget.cpp
  Config/GameConfigWidget.cpp
  Config/GeckoCodeWidget.cpp
  Config/Graphics/AdvancedWidget.cpp
  Config/Graphics/EnhancementsWidget.cpp
  Config/Graphics/GeneralWidget.cpp
  Config/Graphics/HacksWidget.cpp
  Config/Graphics/GraphicsBool.cpp
  Config/Graphics/GraphicsChoice.cpp
  Config/Graphics/GraphicsRadio.cpp
  Config/Graphics/GraphicsSlider.cpp
  Config/Graphics/GraphicsWidget.cpp
  Config/Graphics/GraphicsWindow.cpp
  Config/Graphics/SoftwareRendererWidget.cpp
  Config/InfoWidget.cpp
  Config/LogConfigWidget.cpp
  Config/LogWidget.cpp
  Config/Mapping/GCKeyboardEmu.cpp
  Config/Mapping/GCMicrophone.cpp
  Config/Mapping/GCPadEmu.cpp
  Config/Mapping/GCPadWiiUConfigDialog.cpp
  Config/Mapping/Hotkey3D.cpp
  Config/Mapping/HotkeyDebugging.cpp
  Config/Mapping/HotkeyGeneral.cpp
  Config/Mapping/HotkeyGraphics.cpp
  Config/Mapping/HotkeyStates.cpp
  Config/Mapping/HotkeyTAS.cpp
  Config/Mapping/HotkeyWii.cpp
  Config/Mapping/IOWindow.cpp
  Config/Mapping/MappingBool.cpp
  Config/Mapping/MappingButton.cpp
  Config/Mapping/MappingCommon.cpp
  Config/Mapping/MappingIndicator.cpp
  Config/Mapping/MappingNumeric.cpp
  Config/Mapping/MappingWidget.cpp
  Config/Mapping/MappingWindow.cpp
  Config/Mapping/WiimoteEmuExtension.cpp
  Config/Mapping/WiimoteEmuGeneral.cpp
  Config/Mapping/WiimoteEmuMotionControl.cpp
  Config/NewPatchDialog.cpp
  Config/PatchesWidget.cpp
  Config/PropertiesDialog.cpp
  Config/SettingsWindow.cpp
  Debugger/BreakpointWidget.cpp
  Debugger/CodeViewWidget.cpp
  Debugger/CodeWidget.cpp
  Debugger/MemoryViewWidget.cpp
  Debugger/MemoryWidget.cpp
  Debugger/NewBreakpointDialog.cpp
  Debugger/RegisterColumn.cpp
  Debugger/RegisterWidget.cpp
  Debugger/WatchWidget.cpp
  GameList/GameList.cpp
  GameList/GameListModel.cpp
  GameList/GameTracker.cpp
  GameList/GridProxyModel.cpp
  GameList/ListProxyModel.cpp
  GCMemcardManager.cpp
  QtUtils/BlockUserInputFilter.cpp
  NetPlay/GameListDialog.cpp
  NetPlay/MD5Dialog.cpp
  NetPlay/NetPlayDialog.cpp
  NetPlay/NetPlaySetupDialog.cpp
  NetPlay/PadMappingDialog.cpp
  QtUtils/DoubleClickEventFilter.cpp
  QtUtils/ElidedButton.cpp
  QtUtils/ImageConverter.cpp
  QtUtils/ListTabWidget.cpp
  QtUtils/WindowActivationEventFilter.cpp
  QtUtils/WinIconHelper.cpp
  QtUtils/WrapInScrollArea.cpp
  QtUtils/AspectRatioWidget.cpp
  Settings/AdvancedPane.cpp
  Settings/AudioPane.cpp
  Settings/GameCubePane.cpp
  Settings/GeneralPane.cpp
  Settings/InterfacePane.cpp
  Settings/PathPane.cpp
  Settings/WiiPane.cpp
  Settings/USBDeviceAddToWhitelistDialog.cpp
  TAS/GCTASInputWindow.cpp
  TAS/WiiTASInputWindow.cpp
  TAS/Shared.cpp
  TAS/StickWidget.cpp
  TAS/IRWidget.cpp
  Updater.cpp
)

target_compile_definitions(dolphin-emu-qt2
PRIVATE
  -DQT_USE_QSTRINGBUILDER
  -DQT_NO_CAST_FROM_ASCII
  -DQT_NO_CAST_TO_ASCII
)

target_include_directories(dolphin-emu-qt2
PRIVATE
  ${CMAKE_CURRENT_BINARY_DIR}
  ${Qt5Gui_PRIVATE_INCLUDE_DIRS}
)

target_link_libraries(dolphin-emu-qt2
PRIVATE
  core
  Qt5::Widgets
  uicommon
)

if(WIN32)
  target_sources(dolphin-emu-qt2 PRIVATE DolphinQt2.manifest)
endif()

# Handle localization
find_package(Gettext)
if(GETTEXT_MSGMERGE_EXECUTABLE AND GETTEXT_MSGFMT_EXECUTABLE)
  set(pot_file "${CMAKE_SOURCE_DIR}/Languages/po/dolphin-emu.pot")
  file(GLOB LINGUAS ${CMAKE_SOURCE_DIR}/Languages/po/*.po)

  target_sources(dolphin-emu-qt2 PRIVATE ${pot_file} ${LINGUAS})
  source_group("Localization" FILES ${LINGUAS})
  source_group("Localization\\\\Generated" FILES ${pot_file})

  foreach(po ${LINGUAS})
    get_filename_component(lang ${po} NAME_WE)
    set(mo_dir ${CMAKE_CURRENT_BINARY_DIR}/${lang})
    set(mo ${mo_dir}/dolphin-emu.mo)

    target_sources(dolphin-emu-qt2 PRIVATE ${mo})
    source_group("Localization\\\\Generated" FILES ${mo})

    if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
      set_source_files_properties(${mo} PROPERTIES MACOSX_PACKAGE_LOCATION "Resources/${lang}.lproj")
    else()
      install(FILES ${mo} DESTINATION share/locale/${lang}/LC_MESSAGES)
    endif()

    add_custom_command(OUTPUT ${mo}
      COMMAND mkdir -p ${mo_dir}
      COMMAND ${GETTEXT_MSGMERGE_EXECUTABLE} --quiet --update --backup=none -s ${po} ${pot_file}
      COMMAND ${GETTEXT_MSGFMT_EXECUTABLE} -o ${mo} ${po}
      DEPENDS ${po}
    )
  endforeach()
endif()

if(APPLE)
  # Note: This is copied from DolphinQt, based on the DolphinWX version.

  include(BundleUtilities)
  set(BUNDLE_PATH ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/dolphin-emu-qt2.app)

  # Ask for an application bundle.
  set_target_properties(dolphin-emu-qt2 PROPERTIES
    MACOSX_BUNDLE true
    MACOSX_BUNDLE_INFO_PLIST ${CMAKE_CURRENT_SOURCE_DIR}/Info.plist.in
    )

  # Copy qt.conf into the bundle
  target_sources(dolphin-emu-qt2 PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/qt.conf")
  set_source_files_properties("${CMAKE_CURRENT_SOURCE_DIR}/qt.conf" PROPERTIES MACOSX_PACKAGE_LOCATION Resources)

  # Copy icon into the bundle
  target_sources(dolphin-emu-qt2 PRIVATE "${CMAKE_SOURCE_DIR}/Data/Dolphin.icns")
  set_source_files_properties("${CMAKE_SOURCE_DIR}/Data/Dolphin.icns" PROPERTIES MACOSX_PACKAGE_LOCATION Resources)

  # Copy Qt plugins into the bundle
  get_target_property(qtcocoa_location Qt5::QCocoaIntegrationPlugin LOCATION)
  target_sources(dolphin-emu-qt2 PRIVATE "${qtcocoa_location}")
  set_source_files_properties("${qtcocoa_location}" PROPERTIES MACOSX_PACKAGE_LOCATION MacOS/platforms)

  get_target_property(qtmacstyle_location Qt5::QMacStylePlugin LOCATION)
  target_sources(dolphin-emu-qt2 PRIVATE "${qtmacstyle_location}")
  set_source_files_properties("${qtmacstyle_location}" PROPERTIES MACOSX_PACKAGE_LOCATION MacOS/styles)

  # Copy resources into the bundle
  set_property(DIRECTORY APPEND PROPERTY CMAKE_CONFIGURE_DEPENDS "${CMAKE_SOURCE_DIR}/Data/Sys")
  file(GLOB_RECURSE resources RELATIVE "${CMAKE_SOURCE_DIR}/Data" "${CMAKE_SOURCE_DIR}/Data/Sys/*")
  foreach(res ${resources})
    target_sources(dolphin-emu-qt2 PRIVATE "${CMAKE_SOURCE_DIR}/Data/${res}")
    get_filename_component(resdir "${res}" DIRECTORY)
    set_source_files_properties("${CMAKE_SOURCE_DIR}/Data/${res}" PROPERTIES
      MACOSX_PACKAGE_LOCATION "Resources/${resdir}")
    source_group("Resources" FILES "${CMAKE_SOURCE_DIR}/Data/${res}")
  endforeach()

  # Update library references to make the bundle portable
  include(DolphinPostprocessBundle)
  dolphin_postprocess_bundle(dolphin-emu-qt2)
else()
  install(TARGETS dolphin-emu-qt2 RUNTIME DESTINATION ${bindir})
endif()
