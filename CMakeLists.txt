# ----------------------------------------------------------------------------
#   Basic Configuration
# ----------------------------------------------------------------------------
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

PROJECT(aruco_ogre_test)
set(PROJECT_VERSION "0.0.3")

set(HEADERS
   src/Server.h
   src/vr.h
   src/PulseDetector.h
)
set(SOURCES 
   aruco_test_ogre.cpp
   src/Server.cpp
   src/vr.cpp
   src/PulseDetector.cpp
)


#------------------------------------------------------
# Build type
#------------------------------------------------------

IF(NOT CMAKE_BUILD_TYPE )
   SET( CMAKE_BUILD_TYPE "Release" )
ENDIF()

#------------------------------------------------------
# Lib Names and Dirs
#------------------------------------------------------

if(WIN32)
    # Postfix of DLLs:
	set(CMAKE_MODULE_PATH "$ENV{OGRE_HOME}/CMake/;${CMAKE_MODULE_PATH}")
    set(PROJECT_DLLVERSION "${PROJECT_VERSION_MAJOR}${PROJECT_VERSION_MINOR}${PROJECT_VERSION_PATCH}")
    set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}")
	SET(CMAKE_INCLUDE_PATH ${CMAKE_INCLUDE_PATH} "C:/boost_1_49_0")
	SET(CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH} "C:/boost_1_49_0/lib")
else()
    # Postfix of so's:
    set(PROJECT_DLLVERSION)
    SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_INSTALL_PREFIX}/lib/cmake/ /usr/lib/cmake /usr/local/lib/cmake /usr/share/OGRE/cmake/modules)
endif()


# ----------------------------------------------------------------------------
#   Find Dependencies
# ----------------------------------------------------------------------------

set(BOOST_ROOT "/usr/local")
FIND_PACKAGE(Boost REQUIRED system thread timer chrono)
IF (Boost_FOUND)
    INCLUDE_DIRECTORIES(${Boost_INCLUDE_DIR})
    ADD_DEFINITIONS( "-DHAS_BOOST" )
ENDIF()
find_package(OpenCV REQUIRED)
find_package(aruco 	REQUIRED)
find_package(OGRE REQUIRED)
if(NOT OGRE_FOUND)
  message(SEND_ERROR "Failed to find OGRE")
endif()
find_package(OIS REQUIRED)
if(NOT OIS_FOUND)
	message(SEND_ERROR "Failed to find OIS.")
endif()

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake) 
FIND_PACKAGE(GSL REQUIRED)
include_directories(${GSL_INCLUDE_DIRS} ${GSLCBLAS_INCLUDE_DIRS})

#collect all libraries
set (REQUIRED_LIBRARIES ${OpenCV_LIBS}  ${aruco_LIBS} ${OGRE_LIBRARY} ${OIS_LIBRARY} ${Boost_LIBRARIES} ${GSL_LIBRARIES} ${GSLCBLAS_LIBRARIES})

include_directories (${OGRE_INCLUDE_DIRS})


# ----------------------------------------------------------------------------
#   Compilation targets
# ----------------------------------------------------------------------------

#add_executable (aruco_test_ogre aruco_test_ogre.cpp)
add_executable (aruco_test_ogre ${SOURCES} ${HEADERS})
#add_executable (aruco_test_board_ogre aruco_test_board_ogre.cpp)
#add_executable (aruco_test_board_ogre_mask aruco_test_board_ogre_mask.cpp)
target_link_libraries(aruco_test_ogre ${REQUIRED_LIBRARIES})
#target_link_libraries(aruco_test_board_ogre ${REQUIRED_LIBRARIES})
#target_link_libraries(aruco_test_board_ogre_mask ${REQUIRED_LIBRARIES})

configure_file(data/plugins.cfg plugins.cfg COPYONLY)
configure_file(data/ogre.cfg ogre.cfg COPYONLY)
#configure_file(data/Sinbad.zip Sinbad.zip COPYONLY)
configure_file(data/male_body.zip male_body.zip COPYONLY)
configure_file(data/haarcascade_frontalface_alt.xml haarcascade_frontalface_alt.xml COPYONLY)




# ----------------------------------------------------------------------------
#   Program Optimization and debug (Extracted from OpenCV)
# ----------------------------------------------------------------------------
set(WARNINGS_ARE_ERRORS 		OFF CACHE BOOL "Treat warnings as errors")
set(WHOLE_PROGRAM_OPTIMIZATION 	OFF CACHE BOOL "Flags for whole program optimization.")

set(EXTRA_C_FLAGS "")
set(EXTRA_C_FLAGS_RELEASE "")
set(EXTRA_C_FLAGS_DEBUG "")
set(EXTRA_EXE_LINKER_FLAGS "")
set(EXTRA_EXE_LINKER_FLAGS_RELEASE "")
set(EXTRA_EXE_LINKER_FLAGS_DEBUG "")


IF(CMAKE_COMPILER_IS_GNUCXX OR MINGW)
    set(ENABLE_PROFILING 		OFF CACHE BOOL "Enable profiling in the GCC compiler (Add flags: -g -pg)")
    set(USE_OMIT_FRAME_POINTER 	ON CACHE BOOL "Enable -fomit-frame-pointer for GCC")
    if(${CMAKE_SYSTEM_PROCESSOR} MATCHES arm*) # We can use only -O2 because the -O3 causes gcc crash
        set(USE_O2 ON CACHE BOOL "Enable -O2 for GCC")
        set(USE_FAST_MATH ON CACHE BOOL "Enable -ffast-math for GCC")
    endif()
    if(${CMAKE_SYSTEM_PROCESSOR} MATCHES powerpc*)
        set(USE_O3 ON CACHE BOOL "Enable -O3 for GCC")
        set(USE_POWERPC ON CACHE BOOL "Enable PowerPC for GCC")
    endif ()
    if(${CMAKE_SYSTEM_PROCESSOR} MATCHES amd64* OR ${CMAKE_SYSTEM_PROCESSOR} MATCHES x86_64*)
        set(USE_O3 ON CACHE BOOL "Enable -O3 for GCC")
        set(USE_FAST_MATH ON CACHE BOOL "Enable -ffast-math for GCC")
        set(USE_MMX ON CACHE BOOL "Enable MMX for GCC")
        set(USE_SSE ON CACHE BOOL "Enable SSE for GCC")
        set(USE_SSE2 ON CACHE BOOL "Enable SSE2 for GCC")
        set(USE_SSE3 ON CACHE BOOL "Enable SSE3 for GCC")
    endif()
    if(${CMAKE_SYSTEM_PROCESSOR} MATCHES i686* OR ${CMAKE_SYSTEM_PROCESSOR} MATCHES x86)
        set(USE_O3 ON CACHE BOOL "Enable -O3 for GCC")
        set(USE_FAST_MATH ON CACHE BOOL "Enable -ffast-math for GCC")
        set(USE_MMX ON CACHE BOOL "Enable MMX for GCC")
        set(USE_SSE OFF CACHE BOOL "Enable SSE for GCC")
        set(USE_SSE2 OFF CACHE BOOL "Enable SSE2 for GCC")
        set(USE_SSE3 OFF CACHE BOOL "Enable SSE3 for GCC")
    endif ()

    set(EXTRA_C_FLAGS "${EXTRA_C_FLAGS} -Wall")

    if(WARNINGS_ARE_ERRORS)
        set(EXTRA_C_FLAGS "${EXTRA_C_FLAGS} -Werror")
    endif()

    # The -Wno-long-long is required in 64bit systems when including sytem headers.
    if(${CMAKE_SYSTEM_PROCESSOR} MATCHES x86_64* OR ${CMAKE_SYSTEM_PROCESSOR} MATCHES amd64*)
		set(EXTRA_C_FLAGS "${EXTRA_C_FLAGS} -Wno-long-long")
    endif()

    # Whole program optimization
    if(WHOLE_PROGRAM_OPTIMIZATION)
        set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -fwhole-program --combine")
    endif()

    # Other optimizations
    if(USE_OMIT_FRAME_POINTER)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -fomit-frame-pointer")
    endif()
    if(USE_O2)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -O2")
    endif()
    if(USE_O3)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -O3")
    endif()
    if(USE_FAST_MATH)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -ffast-math")
    endif()
    if(USE_POWERPC)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -mcpu=G3 -mtune=G5")
    endif()
    if(USE_MMX)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -mmmx")
    endif()
    if(USE_SSE)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -msse")
    endif()
    if(USE_SSE2)
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -msse2")
    endif()
    if(USE_SSE3 AND NOT MINGW) # SSE3 should be disabled under MingW because it generates compiler errors
       set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -msse3")
    endif()

    if(ENABLE_PROFILING)
        set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -pg -g")
    else()
        if(NOT APPLE)
            set(EXTRA_C_FLAGS "${EXTRA_C_FLAGS} -ffunction-sections")
        endif()
    endif()

    # Parallel mode
    if(ENABLE_OPENMP)
        set(EXTRA_C_FLAGS "${EXTRA_C_FLAGS}  -fopenmp")
        set(LINKER_LIBS ${LINKER_LIBS} gomp)
    endif()

    set(EXTRA_C_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE} -DNDEBUG")
    set(EXTRA_C_FLAGS_DEBUG "-g3 -O0 -DDEBUG -D_DEBUG -W -Wextra -Wno-return-type")
ENDIF	()

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${EXTRA_C_FLAGS}")
SET(CMAKE_CXX_FLAGS_RELEASE "${EXTRA_C_FLAGS_RELEASE}")
SET(CMAKE_CXX_FLAGS_DEBUG "${EXTRA_C_FLAGS_DEBUG}")
SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${EXTRA_EXE_LINKER_FLAGS}")
SET(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} ${EXTRA_EXE_LINKER_FLAGS_RELEASE}")
SET(CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} ${EXTRA_EXE_LINKER_FLAGS_DEBUG}")








