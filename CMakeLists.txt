cmake_minimum_required(VERSION 3.15)

project(MirrorEngine-ThirdParty)

set(THIRD_PARTY_INSTALL_DIR "${PROJECT_BINARY_DIR}/ThirdParty")

# Set install prefix if not provided by the user; it is suggested to set cmake
# install prefix *after* project() call, 
# see https://stackoverflow.com/questions/39481958
# (HACK: note however subdirectories can have their own project() calls... 
# what's the effect on them?)
if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set(CMAKE_INSTALL_PREFIX ${THIRD_PARTY_INSTALL_DIR} CACHE PATH
        "Folder for storing built third-party libraries." FORCE)
endif()

# Compile with C++17 standard enabled
set(CMAKE_CXX_STANDARD          17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# TODO: add g++/clang MT MD equivalent flags?

#------------------------------------------------------------------------------
# Set suitable CMAKE_CXX_FLAGS
#------------------------------------------------------------------------------

# Using MSVC
if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")

    # use C++17 (VS usually do not pick up previous requests)
    set(CMAKE_CXX_FLAGS     "${CMAKE_CXX_FLAGS} /std:c++17")

    # use PDB files for debugging, which also provides potential optimizations
    set(CMAKE_CXX_FLAGS     "${CMAKE_CXX_FLAGS} /Zi")

    if("${CMAKE_BUILD_TYPE}" STREQUAL "Debug")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MDd")
    else()
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MD")
    endif()

    # use multithreaded compilation
    set(CMAKE_CXX_FLAGS     "${CMAKE_CXX_FLAGS} /MP")

# Using GCC
elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")

    # as a fallback when CMake version < 3.8.0 or for whatever reasons
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")

# Using regular Clang or AppleClang
elseif(CMAKE_CXX_COMPILER_ID MATCHES "Clang")

    # TODO: enforce C++17 if previous requests have no use

endif()

include(ExternalProject)
include(GNUInstallDirs)

set(SPDLOG_LIBRARY_NAME "spdlog")

#------------------------------------------------------------------------------
# spdlog
#------------------------------------------------------------------------------

ExternalProject_Add(${SPDLOG_LIBRARY_NAME}
    PREFIX         "${PROJECT_BINARY_DIR}/${SPDLOG_LIBRARY_NAME}"
    BUILD_ALWAYS   ON
    SOURCE_DIR     "${PROJECT_SOURCE_DIR}/${SPDLOG_LIBRARY_NAME}"
    BINARY_DIR     "${PROJECT_BINARY_DIR}/${SPDLOG_LIBRARY_NAME}"
    INSTALL_DIR    "${THIRD_PARTY_INSTALL_DIR}/${SPDLOG_LIBRARY_NAME}"
    
    CMAKE_CACHE_ARGS
        -DBUILD_TESTING:BOOL=OFF
        -DBUILD_SHARED_LIBS:BOOL=ON
        -DCMAKE_CXX_STANDARD:STRING=${CMAKE_CXX_STANDARD}
        -DCMAKE_CXX_STANDARD_REQUIRED:BOOL=ON
        -DCMAKE_CXX_FLAGS:STRING=${CMAKE_CXX_FLAGS}
        -DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
        -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
        
    INSTALL_COMMAND ${CMAKE_COMMAND} --install <BINARY_DIR> --config $<CONFIG> --prefix <INSTALL_DIR>)

