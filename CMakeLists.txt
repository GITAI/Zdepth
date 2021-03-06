cmake_minimum_required(VERSION 3.5)
project(zdepth LANGUAGES CXX)
include("cmake/zdepth-config-version.cmake")
set(CMAKE_CXX_STANDARD 11)


################################################################################
# Build Options

set(CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "" FORCE)

# In debug mode, add -DDEBUG
add_compile_options("$<$<CONFIG:DEBUG>:-DDEBUG>")

if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
else()
    # Warnings
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -pedantic")

    # Remove Asio warning
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-local-typedefs")

    # Static library: -fPIC
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -march=native")

    set(CMAKE_CXX_FLAGS_DEBUG "-g -O0 -fstack-protector")
    set(CMAKE_CXX_FLAGS_RELEASE "-O3")
endif()


################################################################################
# Subprojects

if (NOT TARGET zstd)
    add_subdirectory(zstd)
endif()


################################################################################
# Source

set(INCLUDE_FILES
    include/zdepth.hpp
)

set(SOURCE_FILES
    ${INCLUDE_FILES}
    src/zdepth.cpp
)

include_directories(include)


################################################################################
# Targets

if (WIN32 AND BUILD_SHARED_LIBS)
    set(CMAKE_POSITION_INDEPENDENT_CODE ON)
    if (WIN32)
        set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
        include(GenerateExportHeader)
    endif ()
endif ()

# zdepth library
add_library(zdepth ${SOURCE_FILES})
add_library(zdepth::zdepth ALIAS zdepth)

target_link_libraries(zdepth PUBLIC zstd)
set_target_properties(zdepth PROPERTIES PUBLIC_HEADER include/zdepth.hpp)
target_include_directories(zdepth PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<INSTALL_INTERFACE:include>"
)

# zdepth test application

add_executable(zdepth_test tests/zdepth_test.cpp)
target_link_libraries(zdepth_test PRIVATE zdepth)


################################################################################
# Install

include(GNUInstallDirs)

install(FILES LICENSE DESTINATION ${CMAKE_INSTALL_PREFIX})
install(FILES README.md DESTINATION ${CMAKE_INSTALL_PREFIX})
install(TARGETS zdepth
    EXPORT zdepth-target
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(EXPORT zdepth-target DESTINATION ${CMAKE_INSTALL_PREFIX}/share/Zdepth/cmake
  EXPORT_LINK_INTERFACE_LIBRARIES)
install(EXPORT zstd-target DESTINATION ${CMAKE_INSTALL_PREFIX}/share/Zdepth/cmake
  EXPORT_LINK_INTERFACE_LIBRARIES)

include(CMakePackageConfigHelpers)
configure_package_config_file(cmake/ZdepthConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/ZdepthConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_PREFIX}/share/Zdepth/cmake)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/ZdepthConfig.cmake
  DESTINATION ${CMAKE_INSTALL_PREFIX}/share/Zdepth/cmake)
