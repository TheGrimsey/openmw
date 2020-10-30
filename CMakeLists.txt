# Apps and tools
option(BUILD_OPENMW             "Build OpenMW" ON)
option(BUILD_LAUNCHER           "Build Launcher" ON)
option(BUILD_WIZARD             "Build Installation Wizard" ON)
option(BUILD_MWINIIMPORTER      "Build MWiniImporter" ON)
option(BUILD_OPENCS             "Build OpenMW Construction Set" ON)
option(BUILD_ESSIMPORTER        "Build ESS (Morrowind save game) importer" ON)
option(BUILD_BSATOOL            "Build BSA extractor" ON)
option(BUILD_ESMTOOL            "Build ESM inspector" ON)
option(BUILD_NIFTEST            "Build nif file tester" ON)
option(BUILD_DOCS               "Build documentation." OFF )
option(BUILD_WITH_CODE_COVERAGE "Enable code coverage with gconv" OFF)
option(BUILD_UNITTESTS          "Enable Unittests with Google C++ Unittest" OFF)
option(BULLET_USE_DOUBLES       "Use double precision for Bullet" OFF)

if (POLICY CMP0048)
    # cmake warns if loaded from a min-3.0-required parent dir, so silence the warning:
    cmake_policy(SET CMP0048 NEW)
endif()

project(OpenMW VERSION 1.0.0)
cmake_minimum_required(VERSION 3.12)

# Set up configuration.
set(CMAKE_CONFIGURATION_TYPES Debug Release RelWithDebInfo CACHE STRING "" FORCE)
set(CMAKE_CXX_STANDARD 17)
set(ROOT_FOLDER ${PROJECT_NAME})

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# Set up source & build directories.
set(SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR})

# Includes.
include(OpenMWMacros)

# Dependencies.

# OpenGL
find_package(OpenGL REQUIRED)

# LZ4
find_package(LZ4 REQUIRED)

# Qt
find_package(Qt5Core 5.12 REQUIRED)
find_package(Qt5Widgets REQUIRED)
find_package(Qt5Network REQUIRED)
find_package(Qt5OpenGL REQUIRED)

# FFmpeg
find_package(FFmpeg REQUIRED COMPONENTS AVCODEC AVFORMAT AVUTIL SWSCALE SWRESAMPLE)
# Required for building the FFmpeg headers.
add_definitions(-D__STDC_CONSTANT_MACROS)

# OpenSceneGraph
find_package(OpenSceneGraph 3.3.4 REQUIRED osgDB osgViewer osgText osgGA osgParticle osgUtil osgFX osgShadow)
include_directories(SYSTEM ${OPENSCENEGRAPH_INCLUDE_DIRS})
set(USED_OSG_PLUGINS osgdb_bmp osgdb_dds osgdb_freetype osgdb_jpeg osgdb_osg osgdb_png osgdb_serializers_osg osgdb_tga )
set(OSGPlugins_LIB_DIR "")
foreach(OSGDB_LIB ${OSGDB_LIBRARY})
    # Skip library type names
	  if(EXISTS ${OSGDB_LIB} AND NOT IS_DIRECTORY ${OSGDB_LIB})
		    get_filename_component(OSG_LIB_DIR ${OSGDB_LIB} DIRECTORY)
		    list(APPEND OSGPlugins_LIB_DIR "${OSG_LIB_DIR}/osgPlugins-${OPENSCENEGRAPH_VERSION}")
	  endif()
endforeach(OSGDB_LIB)

# Boost
set(BOOST_COMPONENTS system filesystem program_options iostreams)
if(WIN32)
    set(BOOST_COMPONENTS ${BOOST_COMPONENTS} locale)
    if(MSVC)
        set(BOOST_COMPONENTS ${BOOST_COMPONENTS} zlib)
    endif(MSVC)
endif(WIN32)
find_package(Boost REQUIRED COMPONENTS ${BOOST_COMPONENTS})

find_package(MyGUI 3.2.2 REQUIRED)
find_package(SDL2 2.0.9 REQUIRED)
find_package(OpenAL REQUIRED)
find_package(Bullet 286 REQUIRED COMPONENTS BulletCollision LinearMath)

include_directories("."
    SYSTEM
    ${Boost_INCLUDE_DIR}
    ${MyGUI_INCLUDE_DIRS}
    ${SDL2_INCLUDE_DIR}
    ${OPENAL_INCLUDE_DIR}
    ${Bullet_INCLUDE_DIR}
)
link_directories(${SDL2_LIBRARY_DIRS} ${Boost_LIBRARY_DIRS})

add_subdirectory(files/)

# Build Paths.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${BINARY_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${BINARY_DIR}")

# Other files
configure_resource_file(${SOURCE_DIR}/files/settings-default.cfg "${BINARY_DIR}" "settings-default.cfg")
configure_resource_file(${SOURCE_DIR}/files/openmw.appdata.xml "${BINARY_DIR}" "openmw.appdata.xml")
configure_resource_file(${SOURCE_DIR}/files/openmw.cfg.local "${BINARY_DIR}" "openmw.cfg")
configure_resource_file(${SOURCE_DIR}/files/openmw.cfg "${BINARY_DIR}" "openmw.cfg.install")
configure_resource_file(${SOURCE_DIR}/files/openmw-cs.cfg "${BINARY_DIR}" "openmw-cs.cfg")
configure_resource_file(${SOURCE_DIR}/files/gamecontrollerdb.txt "${BINARY_DIR}" "gamecontrollerdb.txt")

# Needs the copy version because the configure version assumes the end of the file has been reached when a null character is reached and there are no CMake expressions to evaluate.
copy_resource_file(${SOURCE_DIR}/files/opencs/defaultfilters "${BINARY_DIR}" "resources/defaultfilters")

# Extern
set(RECASTNAVIGATION_STATIC ON CACHE BOOL "Build recastnavigation static libraries")

add_subdirectory (extern/recastnavigation EXCLUDE_FROM_ALL)
add_subdirectory (extern/osg-ffmpeg-videoplayer)
add_subdirectory (extern/oics)
if (BUILD_OPENCS)
    add_subdirectory (extern/osgQt)
endif()

# Components
add_subdirectory (components)
target_compile_definitions(components PRIVATE OPENMW_DOC_BASEURL="${OPENMW_DOC_BASEURL}")

# Apps and tools
if (BUILD_OPENMW)
    add_subdirectory( apps/openmw )
endif()

if (BUILD_BSATOOL)
  add_subdirectory( apps/bsatool )
endif()

if (BUILD_ESMTOOL)
  add_subdirectory( apps/esmtool )
endif()

if (BUILD_LAUNCHER)
   add_subdirectory( apps/launcher )
endif()

if (BUILD_MWINIIMPORTER)
   add_subdirectory( apps/mwiniimporter )
endif()

if (BUILD_ESSIMPORTER)
   add_subdirectory (apps/essimporter )
endif()

if (BUILD_OPENCS)
   add_subdirectory (apps/opencs)
endif()

if (BUILD_WIZARD)
   add_subdirectory(apps/wizard)
endif()

if (BUILD_NIFTEST)
    add_subdirectory(apps/niftest)
endif(BUILD_NIFTEST)

# UnitTests
if (BUILD_UNITTESTS)
  add_subdirectory( apps/openmw_test_suite )
endif()

# Installation
get_generator_is_multi_config(multi_config)
if (multi_config)
    SET(INSTALL_SOURCE "${BINARY_DIR}/$<CONFIG>")
else (multi_config)
    SET(INSTALL_SOURCE "${BINARY_DIR}")
endif (multi_config)

if(WIN32)
    INSTALL(DIRECTORY "${INSTALL_SOURCE}/" DESTINATION "." FILES_MATCHING PATTERN "*.dll"
        PATTERN "deps" EXCLUDE
        PATTERN "apps" EXCLUDE
        PATTERN "CMakeFiles" EXCLUDE
        PATTERN "components" EXCLUDE
        PATTERN "docs" EXCLUDE
        PATTERN "extern" EXCLUDE
        PATTERN "files" EXCLUDE
        PATTERN "Testing" EXCLUDE)

    INSTALL(DIRECTORY "${INSTALL_SOURCE}/" DESTINATION "." CONFIGURATIONS Debug;RelWithDebInfo FILES_MATCHING PATTERN "*.pdb"
        PATTERN "deps" EXCLUDE
        PATTERN "apps" EXCLUDE
        PATTERN "CMakeFiles" EXCLUDE
        PATTERN "components" EXCLUDE
        PATTERN "docs" EXCLUDE
        PATTERN "extern" EXCLUDE
        PATTERN "files" EXCLUDE
        PATTERN "Testing" EXCLUDE)

    INSTALL(FILES "${INSTALL_SOURCE}/openmw.cfg.install" DESTINATION "." RENAME "openmw.cfg")
    INSTALL(FILES "${SOURCE_DIR}/CHANGELOG.md" DESTINATION "." RENAME "CHANGELOG.txt")
    INSTALL(FILES "${SOURCE_DIR}/README.md" DESTINATION "." RENAME "README.txt")
    INSTALL(FILES "${SOURCE_DIR}/LICENSE" DESTINATION "." RENAME "LICENSE.txt")
    INSTALL(FILES "${SOURCE_DIR}/files/mygui/DejaVuFontLicense.txt" DESTINATION ".")
    INSTALL(FILES "${INSTALL_SOURCE}/settings-default.cfg" DESTINATION ".")
    INSTALL(FILES "${INSTALL_SOURCE}/gamecontrollerdb.txt" DESTINATION ".")

    INSTALL(DIRECTORY "${INSTALL_SOURCE}/resources" DESTINATION ".")

    SET(CPACK_GENERATOR "NSIS")
    SET(CPACK_PACKAGE_NAME "OpenMW")
    SET(CPACK_PACKAGE_VENDOR "OpenMW.org")
    SET(CPACK_PACKAGE_VERSION ${OPENMW_VERSION})
    SET(CPACK_PACKAGE_VERSION_MAJOR ${OPENMW_VERSION_MAJOR})
    SET(CPACK_PACKAGE_VERSION_MINOR ${OPENMW_VERSION_MINOR})
    SET(CPACK_PACKAGE_VERSION_PATCH ${OPENMW_VERSION_RELEASE})
    SET(CPACK_PACKAGE_EXECUTABLES "openmw;OpenMW")
    IF(BUILD_LAUNCHER)
        SET(CPACK_PACKAGE_EXECUTABLES "${CPACK_PACKAGE_EXECUTABLES};openmw-launcher;OpenMW Launcher")
    ENDIF(BUILD_LAUNCHER)
    IF(BUILD_OPENCS)
        SET(CPACK_PACKAGE_EXECUTABLES "${CPACK_PACKAGE_EXECUTABLES};openmw-cs;OpenMW Construction Set")
    ENDIF(BUILD_OPENCS)
    IF(BUILD_WIZARD)
        SET(CPACK_PACKAGE_EXECUTABLES "${CPACK_PACKAGE_EXECUTABLES};openmw-wizard;OpenMW Wizard")
    ENDIF(BUILD_WIZARD)
    SET(CPACK_NSIS_CREATE_ICONS_EXTRA "CreateShortCut '\$SMPROGRAMS\\\\$STARTMENU_FOLDER\\\\Readme.lnk' '\$INSTDIR\\\\README.txt'")
    SET(CPACK_NSIS_DELETE_ICONS_EXTRA "!insertmacro MUI_STARTMENU_GETFOLDER Application $MUI_TEMP Delete \\\"$SMPROGRAMS\\\\$MUI_TEMP\\\\Readme.lnk\\\"")
    SET(CPACK_RESOURCE_FILE_README "${SOURCE_DIR}/README.md")
    SET(CPACK_PACKAGE_DESCRIPTION_FILE "${SOURCE_DIR}/README.md")
    SET(CPACK_NSIS_EXECUTABLES_DIRECTORY ".")
    SET(CPACK_NSIS_DISPLAY_NAME "OpenMW ${OPENMW_VERSION}")
    SET(CPACK_NSIS_HELP_LINK "https:\\\\\\\\www.openmw.org")
    SET(CPACK_NSIS_URL_INFO_ABOUT "https:\\\\\\\\www.openmw.org")
    SET(CPACK_NSIS_INSTALLED_ICON_NAME "openmw-launcher.exe")
    SET(CPACK_NSIS_MUI_FINISHPAGE_RUN "openmw-launcher.exe")
    SET(CPACK_NSIS_MUI_ICON "${SOURCE_DIR}/files/windows/openmw.ico")
    SET(CPACK_NSIS_MUI_UNIICON "${SOURCE_DIR}/files/windows/openmw.ico")
    SET(CPACK_PACKAGE_ICON "${SOURCE_DIR}\\\\files\\\\openmw.bmp")

    SET(VCREDIST32 "${BINARY_DIR}/vcredist_x86.exe")
    if(EXISTS ${VCREDIST32})
        INSTALL(FILES ${VCREDIST32} DESTINATION "redist")
        SET(CPACK_NSIS_EXTRA_INSTALL_COMMANDS "ExecWait '\\\"$INSTDIR\\\\redist\\\\vcredist_x86.exe\\\" /q'" )
    endif(EXISTS ${VCREDIST32})

    SET(VCREDIST64 "${BINARY_DIR}/vcredist_x64.exe")
    if(EXISTS ${VCREDIST64})
        INSTALL(FILES ${VCREDIST64} DESTINATION "redist")
        SET(CPACK_NSIS_EXTRA_INSTALL_COMMANDS "ExecWait '\\\"$INSTDIR\\\\redist\\\\vcredist_x64.exe\\\" /q'" )
    endif(EXISTS ${VCREDIST64})

    SET(OALREDIST "${BINARY_DIR}/oalinst.exe")
    if(EXISTS ${OALREDIST})
        INSTALL(FILES ${OALREDIST} DESTINATION "redist")
        SET(CPACK_NSIS_EXTRA_INSTALL_COMMANDS "${CPACK_NSIS_EXTRA_INSTALL_COMMANDS} ExecWait '\\\"$INSTDIR\\\\redist\\\\oalinst.exe\\\" /s'" )
    endif(EXISTS ${OALREDIST})

    if(CMAKE_CL_64)
        SET(CPACK_NSIS_INSTALL_ROOT "$PROGRAMFILES64")
    endif()

    include(CPack)
endif(WIN32)
# TODO LINUX INSTALLATION