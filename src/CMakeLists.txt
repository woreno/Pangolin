#######################################################
## Library sources

SET( INCDIR "../include/pangolin")

SET(
  HEADERS
  ${INCDIR}/pangolin.h
  ${INCDIR}/platform.h
  ${INCDIR}/simple_math.h
  ${INCDIR}/plotter.h 
  ${INCDIR}/display.h ${INCDIR}/display_internal.h 
  ${INCDIR}/view.h ${INCDIR}/viewport.h
  ${INCDIR}/handler.h ${INCDIR}/opengl_render_state.h
  ${INCDIR}/gl.h ${INCDIR}/gldraw.h ${INCDIR}/glvbo.h
  ${INCDIR}/glcuda.h ${INCDIR}/cg.h ${INCDIR}/glsl.h
  ${INCDIR}/threadedfilebuf.h ${INCDIR}/timer.h
)

SET(
  SOURCES
  display.cpp view.cpp viewport.cpp
  handler.cpp opengl_render_state.cpp
  plotter.cpp
  threadedfilebuf.cpp
)

#######################################################
## User build options

OPTION(BUILD_PANGOLIN_VARS "Build support for Pangolin Vars" ON)
IF(BUILD_PANGOLIN_VARS)
  LIST(APPEND HEADERS ${INCDIR}/vars.h ${INCDIR}/vars_internal.h ${INCDIR}/input_record_repeat.h ${INCDIR}/widgets.h )
  LIST(APPEND SOURCES vars.cpp input_record_repeat.cpp widgets.cpp )
ENDIF()

OPTION(BUILD_PANGOLIN_VIDEO "Build support for Pangolin Video Utilities" ON)
IF(BUILD_PANGOLIN_VIDEO)
  LIST(APPEND HEADERS ${INCDIR}/video.h ${INCDIR}/video_recorder.h ${INCDIR}/video_record_repeat.h ${INCDIR}/video/pvn_video.h )
  LIST(APPEND SOURCES video.cpp video_recorder.cpp video_record_repeat.cpp video/pvn_video.cpp )
ENDIF()

#######################################################
## Find required dependencies

FIND_PACKAGE(OpenGL REQUIRED QUIET)
FIND_PACKAGE(GLEW REQUIRED)
FIND_PACKAGE(Boost COMPONENTS thread filesystem system REQUIRED QUIET)

## Apply required dependencies settings
LIST(APPEND USER_INC  "${PROJECT_SOURCE_DIR}/include;${CMAKE_CURRENT_SOURCE_DIR}/include" )
LIST(APPEND USER_INC  "${OPENGL_INCLUDE_DIR};${GLEW_INCLUDE_DIR};${Boost_INCLUDE_DIRS}" )
LIST(APPEND LINK_LIBS "${OPENGL_LIBRARIES};${GLEW_LIBRARY};${Boost_LIBRARIES}" )
LIST(APPEND LIB_DIRS  "${OPENGL_LIBRARY_DIR};${GLEW_LIBRARY_DIR};${Boost_LIBRARY_DIRS}" )

#######################################################
## Find optional dependencies

FIND_PACKAGE(CVARS QUIET)
IF(CVARS_FOUND)
  SET(HAVE_CVARS 1)
  LIST(APPEND USER_INC  ${CVARS_INCLUDE_DIR} )
  LIST(APPEND LINK_LIBS ${CVARS_LIBRARIES} )
  LIST(APPEND LIB_DIRS ${CVARS_DIR} )
ENDIF()

FIND_PACKAGE(CUDA QUIET)
IF(CUDA_FOUND)
  SET(HAVE_CUDA 1)
  CUDA_INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})
  SET(CUDA_ATTACH_VS_BUILD_RULE_TO_CUDA_FILE OFF)
  SET(CUDA_NVCC_FLAGS "-O3" )
  ADD_DEFINITIONS(-DCUDA_BUILD)
  MESSAGE(STATUS "CUDA Found and Enabled")
ENDIF()

FIND_PACKAGE(FREEGLUT)
FIND_PACKAGE(GLUT)

IF(_OSX_)
  INCLUDE(CheckCXXSourceRuns)
  SET(CMAKE_REQUIRED_LIBRARIES ${GLUT_LIBRARY})
  CHECK_CXX_SOURCE_RUNS("#include <GLUT/glut.h>\n int main () {return glutGetProcAddress(\"glutScrollFunc\") ? 0 : -1;};" HAVE_MODIFIED_OSXGLUT)
  IF(NOT HAVE_MODIFIED_OSXGLUT)
        MESSAGE(STATUS "Install modified osxglut for smooth scroll support / pinch / zoom.")
        MESSAGE(STATUS "(https://github.com/stevenlovegrove/osxglut)")
  ENDIF()
ENDIF()

# Prefer OSX_MODIFIED_GLUT > FREEGLUT > GLUT
IF(FREEGLUT_FOUND AND NOT HAVE_MODIFIED_OSXGLUT)
  SET(HAVE_FREEGLUT 1)
  LIST(APPEND USER_INC  ${FREEGLUT_INCLUDE_DIR} )
  LIST(APPEND LINK_LIBS ${FREEGLUT_LIBRARY} )
ELSEIF(GLUT_FOUND)
  LIST(APPEND USER_INC  ${GLUT_INCLUDE_DIR} )
  LIST(APPEND LINK_LIBS ${GLUT_LIBRARY} )
  IF(_OSX_)
    SET(HAVE_APPLE_OPENGL_FRAMEWORK 1)
    IF(NOT HAVE_MODIFIED_OSXGLUT)
        MESSAGE(STATUS "Using Apple Framework GLUT.")
        MESSAGE(STATUS "Install Freeglut or modified osxglut for scroll support.")
    ENDIF()
  ENDIF()
ENDIF()

IF(FREEGLUT_FOUND OR GLUT_FOUND)
    SET(HAVE_GLUT 1)
    MESSAGE(STATUS "Glut Found and Enabled")
ENDIF()

FIND_PACKAGE(Eigen3 QUIET)
IF(EIGEN3_FOUND)
  SET(HAVE_EIGEN 1)
  LIST(APPEND USER_INC  ${EIGEN3_INCLUDE_DIR} )
  MESSAGE(STATUS "Eigen Found and Enabled")
ENDIF()

FIND_PACKAGE(TooN QUIET)
IF(TooN_FOUND)
  SET(HAVE_TOON 1)
  LIST(APPEND USER_INC  ${TooN_INCLUDE_DIR} )
  MESSAGE(STATUS "TooN Found and Enabled")
ENDIF()

FIND_PACKAGE(DC1394 QUIET)
IF(BUILD_PANGOLIN_VIDEO AND DC1394_FOUND)
  SET(HAVE_DC1394 1)
  LIST(APPEND INTERNAL_INC  ${DC1394_INCLUDE_DIR} )
  LIST(APPEND LINK_LIBS  ${DC1394_LIBRARY} )
  LIST(APPEND HEADERS ${INCDIR}/video/firewire.h )
  LIST(APPEND SOURCES video/firewire.cpp)
  MESSAGE(STATUS "libdc1394 Found and Enabled")
ENDIF()

IF(BUILD_PANGOLIN_VIDEO AND _LINUX_)
  SET(HAVE_V4L 1)
  LIST(APPEND HEADERS ${INCDIR}/video/v4l.h)
  LIST(APPEND SOURCES video/v4l.cpp)
ENDIF()

FIND_PACKAGE(FFMPEG QUIET)
IF(BUILD_PANGOLIN_VIDEO AND FFMPEG_FOUND)
  SET(HAVE_FFMPEG 1)
  LIST(APPEND INTERNAL_INC  ${FFMPEG_INCLUDE_DIRS} )
  LIST(APPEND LINK_LIBS ${FFMPEG_LIBRARIES} )
  LIST(APPEND HEADERS ${INCDIR}/video/ffmpeg.h)
  LIST(APPEND SOURCES video/ffmpeg.cpp)
  MESSAGE(STATUS "ffmpeg Found and Enabled")
ENDIF()

FIND_PACKAGE(OpenNI QUIET)
IF(BUILD_PANGOLIN_VIDEO AND OPENNI_FOUND)
  SET(HAVE_OPENNI 1)
  LIST(APPEND INTERNAL_INC  ${OPENNI_INCLUDE_DIRS} )
  LIST(APPEND LINK_LIBS ${OPENNI_LIBRARIES} )
  LIST(APPEND HEADERS ${INCDIR}/video/openni.h )
  LIST(APPEND SOURCES video/openni.cpp)
  MESSAGE(STATUS "OpenNI Found and Enabled")
ENDIF()

# Generic Image Library (GIL) should come with boost.
# If you have a version of boost without it, maybe figure out how to test here.
SET(HAVE_BOOST_GIL 1)

find_package(PNG QUIET)
if(PNG_FOUND)
  set(HAVE_PNG 1)
  list(APPEND INTERNAL_INC ${PNG_INCLUDE_DIR} )
  list(APPEND LINK_LIBS ${PNG_LIBRARY} )
endif()

find_package(JPEG QUIET)
if(JPEG_FOUND)
  set(HAVE_JPEG 1)
  list(APPEND INTERNAL_INC ${JPEG_INCLUDE_DIR} )
  list(APPEND LINK_LIBS ${JPEG_LIBRARY} )
endif()

find_package(TIFF QUIET)
if(TIFF_FOUND)
  set(HAVE_TIFF 1)
  list(APPEND INTERNAL_INC ${TIFF_INCLUDE_DIR} )
  list(APPEND LINK_LIBS ${TIFF_LIBRARY} )
endif()


#######################################################
## Add Libraries / Include Directories / Link directories

INCLUDE_DIRECTORIES( ${INTERNAL_INC} )
INCLUDE_DIRECTORIES( ${USER_INC} )
LINK_LIBRARIES(${LINK_LIBS})
LINK_DIRECTORIES(${LIB_DIRS})

ADD_LIBRARY(${LIBRARY_NAME} ${SOURCES} ${HEADERS})

## Set Special Compiler flags
IF(MSVC)
  SET(CMAKE_CXX_FLAGS "/EHs")
ELSEIF(CMAKE_COMPILER_IS_GNUCXX)
  SET(CMAKE_CXX_FLAGS "-Wall -Wno-error=deprecated-declarations")
ENDIF()

#######################################################
## Create configure file for inclusion in library

CONFIGURE_FILE(
  "${CMAKE_CURRENT_SOURCE_DIR}/config.h.in"
  "${CMAKE_CURRENT_SOURCE_DIR}/include/${LIBRARY_NAME}/config.h"
)

#######################################################
## Install headers / targets

INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/config.h
  DESTINATION ${CMAKE_INSTALL_PREFIX}/include/${LIBRARY_NAME}
)
INSTALL(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/${LIBRARY_NAME}
  DESTINATION ${CMAKE_INSTALL_PREFIX}/include/${LIBRARY_NAME}
)

INSTALL(TARGETS ${LIBRARY_NAME}
  RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
  LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
  ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
)

#######################################################
## Export config file for easy inclusion into other projects: FIND_PACKAGE( Pangolin )
get_target_property( LIB_LOC ${LIBRARY_NAME} LOCATION )
CONFIGURE_FILE( ${CMAKE_CURRENT_SOURCE_DIR}/PangolinConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/PangolinConfig.cmake @ONLY IMMEDIATE )
EXPORT( PACKAGE Pangolin )