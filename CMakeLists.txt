cmake_minimum_required(VERSION 3.7)
project(libime VERSION 1.0.17)
set(LibIME_VERSION ${PROJECT_VERSION})

if(ANDROID)
	set(OS_TYPE "android")
	set(ARCH_TYPE "${ANDROID_ABI}")

	add_definitions(-D__ANDROID__)

	message(STATUS "ANDROID_PLATFORM: [${ANDROID_PLATFORM}]")
	message(STATUS "ANDROID_STL: [${ANDROID_STL}]")
	message(STATUS "ANDROID_ABI: [${ANDROID_ABI}]")
	message(STATUS "CMAKE_TOOLCHAIN_FILE: [${CMAKE_TOOLCHAIN_FILE}]")
	message(STATUS "CMAKE_CXX_FLAGS: [${CMAKE_CXX_FLAGS}]")
elseif(APPLE)
	set(OS_TYPE "darwin")
	set(ARCH_TYPE "${CMAKE_SYSTEM_PROCESSOR}")

	add_definitions(-D__DARWIN__)
elseif(WIN32)
	set(OS_TYPE "win32")
	set(ARCH_TYPE "${CMAKE_SYSTEM_PROCESSOR}")

	add_definitions(-D__WIN32__)

	if ("${ARCH_TYPE}" STREQUAL "AMD64")
		set(ARCH_TYPE "x86_64")
	endif()

	if (("${CMAKE_GENERATOR_PLATFORM}" STREQUAL "Win32") OR (( NOT CMAKE_GENERATOR_PLATFORM) AND ("${CMAKE_VS_PLATFORM_NAME_DEFAULT}" STREQUAL "Win32")))
		set(ARCH_TYPE "x86")
	endif()

	if (("${CMAKE_GENERATOR_PLATFORM}" STREQUAL "x64") OR ((NOT CMAKE_GENERATOR_PLATFORM) AND ("${CMAKE_VS_PLATFORM_NAME_DEFAULT}" STREQUAL "x64")))
		set(ARCH_TYPE "x86_64")
	endif()
elseif(UNIX)
	set(OS_TYPE "linux")
	set(ARCH_TYPE "${CMAKE_SYSTEM_PROCESSOR}")

	add_definitions(-D__LINUX__)
else()
	message(FATAL_ERROR "Unknown OS type")
endif()

find_package(ECM 1.0 REQUIRED)
set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})

#########################################
# Options
#########################################
option(ENABLE_TEST "Build Test" On)
option(ENABLE_COVERAGE "Build the project with gcov support (Need ENABLE_TEST=On)" Off)
set(GCOV_TOOL "gcov" CACHE STRING "Path to gcov tool used by coverage.")
option(ENABLE_DOC "Build doxygen" Off)

#########################################
# Dependency
#########################################

include(GNUInstallDirs)
include(FeatureSummary)
include(GenerateExportHeader)
include(CMakePackageConfigHelpers)
include(ECMSetupVersion)
include(ECMGenerateHeaders)
include(ECMGeneratePkgConfigFile)
include(ECMUninstallTarget)
include(CheckLibraryExists)

# find_package(Fcitx5Utils REQUIRED)
# include("${FCITX_INSTALL_CMAKECONFIG_DIR}/Fcitx5Utils/Fcitx5CompilerSettings.cmake")

# find_package(Boost 1.61 REQUIRED COMPONENTS iostreams filesystem regex)
set(LIBIME_INSTALL_PKGDATADIR "${CMAKE_INSTALL_FULL_DATADIR}/libime")
set(LIBIME_INSTALL_LIBDATADIR "${CMAKE_INSTALL_FULL_LIBDIR}/libime")

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.in ${CMAKE_CURRENT_BINARY_DIR}/config.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

if(ENABLE_TEST)
    enable_testing()
    add_subdirectory(test)

    if (ENABLE_COVERAGE)
        add_custom_target(coverage
                        COMMAND "${CMAKE_CTEST_COMMAND}"
                        COMMAND lcov --gcov-tool "${GCOV_TOOL}" --no-external --capture --directory ./ -b "${CMAKE_CURRENT_SOURCE_DIR}" --output-file coverage.info
                        COMMAND lcov --remove coverage.info "*/kenlm/*" -o coverage.info
                        COMMAND genhtml coverage.info --output-directory "coverage_pages"
                        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
    endif()
endif()

add_subdirectory(src)
# add_subdirectory(tools)
# add_subdirectory(data)

if (ENABLE_DOC)
  find_package(Doxygen REQUIRED)
  file(READ "${CMAKE_CURRENT_SOURCE_DIR}/.codedocs" FCITX_DOXYGEN_CONFIGURATION)
  configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in
    ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
    @ONLY)
  add_custom_target(doc
      COMMAND ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
      WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
      COMMENT "Generating API documentation with Doxygen"
      VERBATIM)
endif()

feature_summary(WHAT ALL FATAL_ON_MISSING_REQUIRED_PACKAGES)
