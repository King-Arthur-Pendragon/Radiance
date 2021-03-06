if(X11_FOUND)
  get_filename_component(X11_LIBRARY_DIR ${X11_X11_LIB} PATH)
endif()
if(TIFF_LIBRARY)
  get_filename_component(TIFF_LIBRARY_DIR ${TIFF_LIBRARY} PATH)
  if(WIN32)
    #also need the location of libtiff3.dll on Windows.
    #By default this is ../bin relative to TIFF_LIBRARY_DIR.
    set(TIFF_BINARY_DIR "${TIFF_LIBRARY_DIR}/../bin")
    get_filename_component(TIFF_BINARY_DIR "${TIFF_BINARY_DIR}" ABSOLUTE)
    set(TIFF_LIBRARY_DIRS "${TIFF_LIBRARY_DIR};${TIFF_BINARY_DIR}")
  else()
    set(TIFF_LIBRARY_DIRS "${TIFF_LIBRARY_DIR}")
  endif()
endif()

if(NOT BUILD_HEADLESS)
  #plugins loaded by rvu at runtime
  get_target_property(GIF_PLUGIN Qt5::QGifPlugin LOCATION)
  get_target_property(ICO_PLUGIN Qt5::QICOPlugin LOCATION)
  get_target_property(JPEG_PLUGIN Qt5::QJpegPlugin LOCATION)
  get_target_property(MNG_PLUGIN Qt5::QMngPlugin LOCATION)
  get_target_property(TIFF_PLUGIN Qt5::QTiffPlugin LOCATION)
  get_filename_component(GIF_PLUGIN_NAME "${GIF_PLUGIN}" NAME)
  get_filename_component(ICO_PLUGIN_NAME "${ICO_PLUGIN}" NAME)
  get_filename_component(JPEG_PLUGIN_NAME "${JPEG_PLUGIN}" NAME)
  get_filename_component(MNG_PLUGIN_NAME "${MNG_PLUGIN}" NAME)
  get_filename_component(TIFF_PLUGIN_NAME "${TIFF_PLUGIN}" NAME)

  if(APPLE OR WIN32)
    if(WIN32)
      get_target_property(QPA_PLUGIN Qt5::QWindowsIntegrationPlugin LOCATION)
    elseif(APPLE)
      get_target_property(QPA_PLUGIN Qt5::QCocoaIntegrationPlugin LOCATION)
    endif()
    get_filename_component(QPA_PLUGIN_NAME "${QPA_PLUGIN}" NAME)
  endif()

  configure_file(
     "${CMAKE_CURRENT_SOURCE_DIR}/dependencies.cmake.in"
     "${CMAKE_CURRENT_BINARY_DIR}/dependencies.cmake"
     @ONLY
     )
  install(SCRIPT "${CMAKE_CURRENT_BINARY_DIR}/dependencies.cmake")
endif()
