function(find_assign_files)
    foreach(_source IN ITEMS ${ARGN})
        if (IS_ABSOLUTE "${_source}")
            file(RELATIVE_PATH _source_rel "${CMAKE_CURRENT_SOURCE_DIR}" "${_source}")
        else()
            set(_source_rel "${_source}")
        endif()
        get_filename_component(_source_path "${_source_rel}" PATH)
        string(REPLACE "/" "\\" _source_path_msvc "${_source_path}")
        source_group("${_source_path_msvc}" FILES "${_source}")
    endforeach()
endfunction(find_assign_files)

# local files
set(GAME
    main.cpp
    engine.cpp

    ${CMAKE_SOURCE_DIR}/files/windows/openmw.rc
    ${CMAKE_SOURCE_DIR}/files/windows/openmw.exe.manifest
)

if (ANDROID)
    set(GAME ${GAME} android_main.cpp)
endif()

set(GAME_HEADER
    engine.hpp
)

project(openmw-client VERSION 1.0.0)

file(GLOB_RECURSE files "*.[ch]pp")

add_executable(${PROJECT_NAME} ${files})
set_target_properties(${PROJECT_NAME} PROPERTIES FOLDER ${ROOT_FOLDER})

find_assign_files(${files})

# Sound stuff - here so CMake doesn't stupidly recompile EVERYTHING
# when we change the backend.
include_directories(
    ${FFmpeg_INCLUDE_DIRS}
)

target_link_libraries(${PROJECT_NAME}
    ${OSG_LIBRARIES}
    ${OSGPARTICLE_LIBRARIES}
    ${OSGUTIL_LIBRARIES}
    ${OSGDB_LIBRARIES}
    ${OSGVIEWER_LIBRARIES}
    ${OSGGA_LIBRARIES}
    ${OSGSHADOW_LIBRARIES}
    ${Boost_SYSTEM_LIBRARY}
    ${Boost_THREAD_LIBRARY}
    ${Boost_FILESYSTEM_LIBRARY}
    ${Boost_PROGRAM_OPTIONS_LIBRARY}
    ${OPENAL_LIBRARY}
    ${FFmpeg_LIBRARIES}
    ${MyGUI_LIBRARIES}
    ${SDL2_LIBRARY}
    ${RecastNavigation_LIBRARIES}
    "osg-ffmpeg-videoplayer"
    "oics"
    components
)

if (ANDROID)
    set (OSG_PLUGINS -Wl,--whole-archive)
    foreach(PLUGIN_NAME ${USED_OSG_PLUGINS})
        set(OSG_PLUGINS ${OSG_PLUGINS} ${OSG_PLUGINS_DIR}/lib${PLUGIN_NAME}.a)
    endforeach()

    set (OSG_PLUGINS ${OSG_PLUGINS} -Wl,--no-whole-archive )

    target_link_libraries(${PROJECT_NAME} EGL android log dl z ${OPENSCENEGRAPH_LIBRARIES} freetype jpeg png )
endif (ANDROID)

if (NOT UNIX)
target_link_libraries(${PROJECT_NAME} ${SDL2MAIN_LIBRARY})
endif()

if (BUILD_WITH_CODE_COVERAGE)
  add_definitions (--coverage)
  target_link_libraries(${PROJECT_NAME} gcov)
endif()

if (MSVC)
    # Debug version needs increased number of sections beyond 2^16
    if (CMAKE_CL_64)
        set (CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /bigobj")
    endif (CMAKE_CL_64)
endif (MSVC)

target_precompile_headers(${PROJECT_NAME} PRIVATE "pch.h")

if (WIN32)
    INSTALL(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION ".")
endif (WIN32)
