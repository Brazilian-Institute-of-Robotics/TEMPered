cmake_minimum_required(VERSION 2.8)
set(CMAKE_USER_MAKE_RULES_OVERRIDE
	"${CMAKE_CURRENT_SOURCE_DIR}/default_c_flags.cmake"
)
project(TEMPered C)

include(CMakeDependentOption)
include(GNUInstallDirs OPTIONAL)

if (NOT DEFINED CMAKE_INSTALL_LIBDIR)
	message(WARNING
		"The GNUInstallDirs module is missing; 'make install' will not work."
		"\nHint: GNUInstallDirs comes standard with CMake 2.8.5 and above."
	)
	# If you need to use an earlier version of CMake, simply copy the
	# GNUInstallDirs.cmake file from a later version, and place it in this
	# directory with the name GNUInstallDirs (without the extension).
endif()

option(BUILD_HIDAPI_SHARED "Build with shared version of HIDAPI" ON)

option(BUILD_SHARED_LIB "Build shared version of tempered library" ON)
option(BUILD_STATIC_LIB "Build static version of tempered library" OFF)

cmake_dependent_option(
	BUILD_UTILITIES "Build the utilities" ON
	"( BUILD_SHARED_LIB OR BUILD_STATIC_LIB )" OFF
)
cmake_dependent_option(
	UTILS_USE_SHARED_LIB "Utilities use the shared tempered library" ON
	"BUILD_UTILITIES;BUILD_SHARED_LIB" OFF
)

find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBHIDAPI REQUIRED hidapi-hidraw hidapi-libusb)

if (BUILD_HIDAPI_SHARED)
    set(HIDAPI_LIB ${LIBHIDAPI_LDFLAGS})
	set(HIDAPI_STATIC_OBJECT)
	set(HIDAPI_LINK_LIBS ${HIDAPI_LIB})
else()
	set(HIDAPI_LIB)
	find_file(HIDAPI_OBJECT NAMES hid.o hid-libusb.o
		PATHS ../hidapi ../hidapi.git
		PATH_SUFFIXES linux/.libs libusb/.libs linux libusb mac
		DOC "The location of the HIDAPI static object file"
	)
	set(HIDAPI_STATIC_OBJECT ${HIDAPI_OBJECT})
	if (HIDAPI_OBJECT MATCHES \(-libusb|/libusb/(.libs/)?hid\)\\.o\$)
		pkg_check_modules(LIBUSB REQUIRED libusb-1.0)
		set(HIDAPI_LINK_LIBS ${LIBUSB_LIBRARIES} rt pthread)
	else()
		pkg_check_modules(LIBUDEV REQUIRED libudev)
		set(HIDAPI_LINK_LIBS ${LIBUDEV_LIBRARIES})
	endif()
endif()

include_directories(${LIBHIDAPI_INCLUDE_DIRS})

add_subdirectory(libtempered)
add_subdirectory(libtempered-util)
add_subdirectory(utils)
add_subdirectory(examples)
