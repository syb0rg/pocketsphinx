#
# Initial CMake and project setup
#

cmake_minimum_required(VERSION 3.0.2)

set(PROJECT_NAME pocketsphinx)
project(${PROJECT_NAME})

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")
set_directory_properties(PROPERTIES EP_PREFIX ${CMAKE_BINARY_DIR}/library-build)

include(ExternalProject)

#
# Setup GCC flags
#

set(CMAKE_C_FLAGS "-std=gnu11 -O3")

#
# Find/install dependencies
#

FIND_PACKAGE(sphinxbase)
if (${SPHINXBASE_FOUND})
    INCLUDE_DIRECTORIES(${SPHINXBASE_INCLUDE_DIR}/sphinxbase/)
else ()
    ExternalProject_Add(
        sphinxbase
        GIT_REPOSITORY      "https://github.com/syb0rg/sphinxbase.git"
        SOURCE_DIR          ${CMAKE_BINARY_DIR}/lib/sphinxbase
        UPDATE_COMMAND      ""
        INSTALL_COMMAND     ""
        BUILD_IN_SOURCE     ON
        LOG_DOWNLOAD        ON
        LOG_UPDATE          ON
        LOG_CONFIGURE       ON
        LOG_BUILD           ON
        LOG_TEST            ON
        LOG_INSTALL         ON
    )
    ExternalProject_Get_Property(sphinxbase SOURCE_DIR)
    ExternalProject_Get_Property(sphinxbase BINARY_DIR)
    SET(SPHINXBASE_SOURCE_DIR ${SOURCE_DIR})
    SET(SPHINXBASE_BINARY_DIR ${BINARY_DIR})
    find_path(SPHINXBASE_LIBRARIES libsphinxbase)
    if (APPLE)
        SET(SPHINXBASE_LIBRARIES ${SPHINXBASE_SOURCE_DIR}/libsphinxbase.dylib)
    elseif (UNIX)
        SET(SPHINXBASE_LIBRARIES ${SPHINXBASE_SOURCE_DIR}/libsphinxbase.so)
    endif ()
    SET(DEPENDENCIES ${DEPENDENCIES} sphinxbase)
    INCLUDE_DIRECTORIES(${SPHINXBASE_SOURCE_DIR}/include)
endif ()
SET(LIBS ${LIBS} ${SPHINXBASE_LIBRARIES})

if (WIN32)
	set(LIBS ${LIBS} winmm)
elseif (UNIX)
    set(LIBS ${LIBS} m pthread)
endif ()

#
# Final project settings
#

INCLUDE_DIRECTORIES(include)
ADD_SUBDIRECTORY(src)

ADD_LIBRARY(${PROJECT_NAME} SHARED
    $<TARGET_OBJECTS:libpocketsphinx>
)

ADD_LIBRARY(${PROJECT_NAME}_static STATIC
	$<TARGET_OBJECTS:libpocketsphinx>
)

set_property(TARGET ${PROJECT_NAME} PROPERTY POSITION_INDEPENDENT_CODE TRUE)
set_property(TARGET ${PROJECT_NAME}_static PROPERTY POSITION_INDEPENDENT_CODE TRUE)

target_compile_features(${PROJECT_NAME} PRIVATE c_restrict)
TARGET_LINK_LIBRARIES(${PROJECT_NAME} ${LIBS})
TARGET_LINK_LIBRARIES(${PROJECT_NAME}_static ${LIBS})

if (DEPENDENCIES)
	MESSAGE (STATUS "Any missing dependencies will be downloaded upon running \"make\"")
	ADD_DEPENDENCIES(${PROJECT_NAME} ${DEPENDENCIES})
endif ()