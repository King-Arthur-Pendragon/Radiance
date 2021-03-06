cmake_minimum_required(VERSION 2.8.4)
project("radiance")

if(NOT ${CMAKE_VERSION} VERSION_LESS "2.8.11")
  cmake_policy(SET CMP0020 NEW)
  if(NOT ${CMAKE_VERSION} VERSION_LESS "3.0")
    cmake_policy(SET CMP0042 NEW)
    if(NOT ${CMAKE_VERSION} VERSION_LESS "3.1")
      cmake_policy(SET CMP0054 NEW)
    endif()
  endif()
endif()

enable_testing()
include(CTest)

option(BUILD_SHARED_LIBS "Build radiance using shared libraries?" OFF)
option(BUILD_HEADLESS "Build radiance without any GUI components?" OFF)

option(CPACK_NREL_SUPPORT "Include OpenStudio ToolFinder Help?" OFF)

if( CPACK_NREL_SUPPORT )
  # find git so we can query for the current git tag
  find_program(git git)
  if (NOT git)
    message(ERROR "Please specify the path to the git executable")
  endif()

  # call git log to get output containing the tag
  execute_process(
    COMMAND ${git} describe --tag --exact-match
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    OUTPUT_VARIABLE git_tag_output)

  # create & install the source version .txt file
  file(WRITE "NREL_ver.txt" "Radiance version ${git_tag_output}")
  install(FILES NREL_ver.txt DESTINATION bin/..)
endif()
 
if(NOT WIN32)
  set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY 1)
  install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/doc/man/ DESTINATION man)
  set(qt_plugin_term "PLUGIN")
endif()

if (WIN32)
  set(CPACK_NSIS_MODIFY_PATH 1)
  set(qt_plugin_term "LIBRARY")

  # Set RAYPATH on install
  set(CPACK_NSIS_EXTRA_INSTALL_COMMANDS
    "WriteRegExpandStr HKLM \\\"SYSTEM\\\\CurrentControlSet\\\\Control\\\\Session Manager\\\\Environment\\\" RAYPATH '\\\$INSTDIR\\\\lib\\\;.'"
  )

  # Unset RAYPATH on uninstall
  set(CPACK_NSIS_EXTRA_UNINSTALL_COMMANDS
    "DeleteRegValue HKLM \\\"SYSTEM\\\\CurrentControlSet\\\\Control\\\\Session Manager\\\\Environment\\\" RAYPATH"
  )
endif()

if(UNIX)
  set(CPACK_PACKAGING_INSTALL_PREFIX "/usr/local/radiance")
  add_definitions(-Dlinux -D_FILE_OFFSET_BITS=64 -DNOSTEREO)
else()
  set(CPACK_PACKAGING_INSTALL_PREFIX "/")
endif()

if(APPLE)
  add_definitions(-DBSD -DNOSTEREO -Dfreebsd)
  include_directories(/usr/X11R6/include)
  SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mmacosx-version-min=10.6")
endif()

IF( BUILD_PACKAGE )
  INCLUDE(radianceCPack.cmake )
ENDIF( BUILD_PACKAGE )

set(CPACK_PACKAGE_VERSION_MAJOR 5)
set(CPACK_PACKAGE_VERSION_MINOR 0)
set(CPACK_PACKAGE_VERSION_PATCH a.8)
set(CPACK_PACKAGE_INSTALL_DIRECTORY "Radiance")
SET(CPACK_RESOURCE_FILE_WELCOME "${radiance_SOURCE_DIR}/Welcome.txt")
SET(CPACK_RESOURCE_FILE_README "${radiance_SOURCE_DIR}/README.txt")
SET(CPACK_RESOURCE_FILE_LICENSE "${radiance_SOURCE_DIR}/License.txt") 


include(CPack)

if(NOT BUILD_HEADLESS)
  find_package(Qt5Widgets)
  find_package(X11)
endif()

SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)

if(WIN32 AND "${CMAKE_C_COMPILER_ID}" MATCHES "^(Intel)$")
  set(_INTEL_WINDOWS 1)
endif()
if(MSVC OR _INTEL_WINDOWS)
  add_definitions(-D_CRT_SECURE_NO_DEPRECATE -D_CRT_NONSTDC_NO_DEPRECATE)
endif()
if(WIN32)
  add_definitions(-DEZXML_NOMMAP -Dfseeko=fseek -Dstrcasecmp=_stricmp)
endif()

add_subdirectory("src")

set(test_dir ${CMAKE_CURRENT_SOURCE_DIR}/test/test\ data)
option(BUILD_TESTING "Download support tarball & enable testing?" ON)

if(BUILD_TESTING)
  #download and extract radiance support files
  include(ExternalProject)
  set(base "${CMAKE_BINARY_DIR}/Downloads")
  set_property(DIRECTORY PROPERTY EP_BASE ${base})
  ExternalProject_Add(radiance_support
    URL http://www.radiance-online.org/software/non-cvs/rad4R1supp.tar.gz
    URL_MD5 efd82970e1d0747eede8eb64c01048f4
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    INSTALL_COMMAND ""
  )

  #install support files.
  set(lib_dir ${base}/Source/radiance_support/lib)
  install(
    FILES
      ${lib_dir}/Earth.cal ${lib_dir}/He.cal ${lib_dir}/He3.cal
      ${lib_dir}/adobe.cal ${lib_dir}/ball.cal ${lib_dir}/brick.cal
      ${lib_dir}/brickweave.cal ${lib_dir}/carpet.cal ${lib_dir}/climit.cal
      ${lib_dir}/corrug.cal ${lib_dir}/covelight.cal ${lib_dir}/cyl.cal
      ${lib_dir}/dichroic.cal ${lib_dir}/dirt.cal ${lib_dir}/fisheye.cal
      ${lib_dir}/fishgeom.cal ${lib_dir}/fprism.cal ${lib_dir}/globe.cal
      ${lib_dir}/gloss.cal ${lib_dir}/grating.cal ${lib_dir}/jigglepic.cal
      ${lib_dir}/lcp0.cal ${lib_dir}/limit.cal ${lib_dir}/marble.cal
      ${lib_dir}/orange.cal ${lib_dir}/picture.cal ${lib_dir}/plink.cal
      ${lib_dir}/prism.cal ${lib_dir}/retrorefl.cal ${lib_dir}/serraglaze.cal
      ${lib_dir}/slate.cal ${lib_dir}/surf.cal ${lib_dir}/tim1.cal
      ${lib_dir}/weave.cal ${lib_dir}/winxmit.cal ${lib_dir}/woodpat.cal
      ${lib_dir}/woodtex.cal ${lib_dir}/wrinkle.cal
      ${lib_dir}/page1.txt ${lib_dir}/page2.txt
      ${lib_dir}/Pix.fnt
      ${lib_dir}/PixOblique.fnt
      ${lib_dir}/Veerdana.fnt
      ${lib_dir}/VeerdanaBold.fnt
      ${lib_dir}/VeerdanaBoldItalic.fnt
      ${lib_dir}/VeerdanaItalic.fnt
      ${lib_dir}/helvet.fnt
      ${lib_dir}/hexbit4x1.fnt
      ${test_dir}/model.oct
      ${test_dir}/model.vp
    DESTINATION lib
  )

  install(FILES ${lib_dir}/Earth.hdr DESTINATION lib RENAME Earth.pic)
  install(FILES ${lib_dir}/EarthBig.hdr DESTINATION lib RENAME EarthBig.pic)
  install(FILES ${lib_dir}/alberta.hdr DESTINATION lib RENAME alberta.pic)
  install(FILES ${lib_dir}/brick.hdr DESTINATION lib RENAME brick.pic)
  install(FILES ${lib_dir}/brickweave.hdr DESTINATION lib RENAME brickweave.pic)
  install(FILES ${lib_dir}/cabin.hdr DESTINATION lib RENAME cabin.pic)
  install(FILES ${lib_dir}/forestfl.hdr DESTINATION lib RENAME forestfl.pic)
  install(FILES ${lib_dir}/oakfloor.hdr DESTINATION lib RENAME oakfloor.pic)
  install(FILES ${lib_dir}/pine.hdr DESTINATION lib RENAME pine.pic)
  install(FILES ${lib_dir}/pinebark.hdr DESTINATION lib RENAME pinebark.pic)
  install(FILES ${lib_dir}/podlife.hdr DESTINATION lib RENAME podlife.pic)
  install(FILES ${lib_dir}/richgrove.hdr DESTINATION lib RENAME richgrove.pic)
  install(FILES ${lib_dir}/shingle.hdr DESTINATION lib RENAME shingle.pic)
  install(FILES ${lib_dir}/summercab.hdr DESTINATION lib RENAME summercab.pic)
  install(FILES ${lib_dir}/tulips.hdr DESTINATION lib RENAME tulips.pic)

  add_subdirectory(cmake_tests)
endif()
  
install(
  FILES
    ${test_dir}/model.oct
    ${test_dir}/model.vp
  DESTINATION lib
)

include(InstallRequiredSystemLibraries)
add_subdirectory(InstallRules)
