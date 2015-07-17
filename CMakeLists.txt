cmake_minimum_required (VERSION 2.6) 
project (Plist)

option(PLIST_BUILD_TESTS "Build and run tests" OFF)

# Disable exceptions in pugixml
set(BUILD_DEFINES "-DPUGIXML_NO_EXCEPTIONS=1")

add_subdirectory(third-party/pugixml/scripts)
add_subdirectory(third-party/NSPlist)

include_directories(third-party third-party/pugixml/src third-party/NSPlist/src)
IF(MSVC)
	include_directories(${CMAKE_SOURCE_DIR}/Windows/include)
ENDIF()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${EXTRA_CXX_FLAGS} -Wall")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${EXTRA_CXX_FLAGS} -Wall")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${EXTRA_CXX_FLAGS} -Wall -DTEST_VERBOSE")

add_library(PlistCpp src/Plist.cpp src/PlistDate.cpp)

IF(PLIST_BUILD_TESTS)
	add_subdirectory(third-party/unittest-cpp)
	include_directories(third-party/unittest-cpp)

	IF(MSVC)
		set (SCRIPT_EXT bat)
		set (MY_BUILD_TYPE "\$(Configuration)")
	ELSE()
		set (SCRIPT_EXT sh)
		set (MY_BUILD_TYPE ${CMAKE_BUILD_TYPE})
	ENDIF()

	add_executable(runTests src/runTests.cpp src/plistTests.cpp)
	target_link_libraries(runTests PlistCpp pugixml UnitTest++ NSPlist)

	ADD_CUSTOM_COMMAND(
		WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
		TARGET runTests
		POST_BUILD
		COMMAND ./runTests.${SCRIPT_EXT} ${CMAKE_CURRENT_BINARY_DIR} ${MY_BUILD_TYPE}
	)
ENDIF()
