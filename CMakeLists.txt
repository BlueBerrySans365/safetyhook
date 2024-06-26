# This file is automatically generated from cmake.toml - DO NOT EDIT
# See https://github.com/build-cpp/cmkr for more information

cmake_minimum_required(VERSION 3.15)
set(CMAKE_CXX_STANDARD 23)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_BINARY_DIR)
	message(FATAL_ERROR "In-tree builds are not supported. Run CMake from a separate directory: cmake -B build")
endif()

set(CMKR_ROOT_PROJECT OFF)
if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
	set(CMKR_ROOT_PROJECT ON)

	# Bootstrap cmkr and automatically regenerate CMakeLists.txt
	include(cmkr.cmake OPTIONAL RESULT_VARIABLE CMKR_INCLUDE_RESULT)
	if(CMKR_INCLUDE_RESULT)
		cmkr()
	endif()

	# Enable folder support
	set_property(GLOBAL PROPERTY USE_FOLDERS ON)

	# Create a configure-time dependency on cmake.toml to improve IDE support
	configure_file(cmake.toml cmake.toml COPYONLY)
endif()

# Options
option(SAFETYHOOK_BUILD_DOCS "" OFF)
option(SAFETYHOOK_BUILD_TESTS "" OFF)
option(SAFETYHOOK_AMALGAMATE "" OFF)
option(SAFETYHOOK_FETCH_ZYDIS "" OFF)

project(safetyhook)

include(FetchContent)

if(SAFETYHOOK_BUILD_TESTS) # build-tests
	message(STATUS "Fetching Catch2 (v3.3.2)...")
	FetchContent_Declare(Catch2
		GIT_REPOSITORY
			"https://github.com/catchorg/Catch2.git"
		GIT_TAG
			v3.3.2
		GIT_SHALLOW
			ON
	)
	FetchContent_MakeAvailable(Catch2)

endif()
if(SAFETYHOOK_BUILD_TESTS) # build-tests
	message(STATUS "Fetching xbyak (v6.69)...")
	FetchContent_Declare(xbyak
		GIT_REPOSITORY
			"https://github.com/herumi/xbyak.git"
		GIT_TAG
			v6.69
		GIT_SHALLOW
			ON
	)
	FetchContent_MakeAvailable(xbyak)

endif()
if(SAFETYHOOK_FETCH_ZYDIS) # fetch-zydis
	option(ZYDIS_BUILD_EXAMPLES "" OFF)
	option(ZYDIS_BUILD_TOOLS "" OFF)
	option(ZYDIS_BUILD_DOXYGEN "" OFF)

	message(STATUS "Fetching Zydis (v4.0.0)...")
	FetchContent_Declare(Zydis
		GIT_REPOSITORY
			"https://github.com/zyantific/zydis.git"
		GIT_TAG
			v4.0.0
		GIT_SHALLOW
			ON
	)
	FetchContent_MakeAvailable(Zydis)

endif()
# Packages
if(SAFETYHOOK_BUILD_DOCS) # build-docs
	find_package(Doxygen REQUIRED)

endif()
if(SAFETYHOOK_AMALGAMATE) # amalgamate
	find_package(Python3 REQUIRED)

endif()
# Target: safetyhook
set(safetyhook_SOURCES
	"src/allocator.cpp"
	"src/easy.cpp"
	"src/inline_hook.cpp"
	"src/mid_hook.cpp"
	"src/thread_freezer.cpp"
	"src/utility.cpp"
	"src/vmt_hook.cpp"
	cmake.toml
)

add_library(safetyhook STATIC)

target_sources(safetyhook PRIVATE ${safetyhook_SOURCES})
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${safetyhook_SOURCES})

add_library(safetyhook::safetyhook ALIAS safetyhook)
target_compile_features(safetyhook PUBLIC
	cxx_std_23
)

if(MSVC) # msvc
	target_compile_options(safetyhook PRIVATE
		"/permissive-"
		"/W4"
		"/w14640"
	)
endif()

if((CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_CXX_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$") OR (CMAKE_C_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_C_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$")) # clang
	target_compile_options(safetyhook PRIVATE
		-Wall
		-Wextra
		-Wshadow
		-Wnon-virtual-dtor
		-pedantic
	)
endif()

if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_C_COMPILER_ID STREQUAL "GNU") # gcc
	target_compile_options(safetyhook PRIVATE
		-Wall
		-Wextra
		-Wshadow
		-Wnon-virtual-dtor
		-pedantic
	)
endif()

target_include_directories(safetyhook PUBLIC
	"include/"
)

target_link_libraries(safetyhook PUBLIC
	Zydis
	ntdll
)

# Target: docs
if(SAFETYHOOK_BUILD_DOCS) # build-docs
	set(CMKR_TARGET docs)
	file(GLOB_RECURSE HEADER_FILES include/*.hpp)
	set(DOXYGEN_INDEX_FILE ${CMAKE_CURRENT_SOURCE_DIR}/docs/html/index.html)
	set(DOXYFILE_IN ${CMAKE_CURRENT_SOURCE_DIR}/docs/Doxyfile.in)
	set(DOXYFILE_OUT ${CMAKE_CURRENT_SOURCE_DIR}/docs/Doxyfile)
	configure_file(${DOXYFILE_IN} ${DOXYFILE_OUT})
	add_custom_command(
	    OUTPUT ${DOXYGEN_INDEX_FILE}
	    DEPENDS ${HEADER_FILES}
	    COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYFILE_OUT}
	    MAIN_DEPENDENCY ${DOXYFILE_OUT} ${DOXYFILE_IN}
	    COMMENT "Generating docs"
	)
	add_custom_target(Doxygen ALL DEPENDS ${DOXYGEN_INDEX_FILE})

	set(docs_SOURCES
		cmake.toml
	)

	add_custom_target(docs SOURCES)

	target_sources(docs PRIVATE ${docs_SOURCES})
	source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${docs_SOURCES})

endif()
# Target: amalgamation
if(SAFETYHOOK_AMALGAMATE) # amalgamate
	set(CMKR_TARGET amalgamation)
	file(GLOB_RECURSE HEADER_FILES include/*.hpp)
	file(GLOB_RECURSE SOURCE_FILES src/*.cpp)
	set(AMALGAMATED_FILE ${CMAKE_CURRENT_SOURCE_DIR}/amalgamated-dist/safetyhook.cpp)
	set(AMALGAMATED_HEADER ${CMAKE_CURRENT_SOURCE_DIR}/amalgamated-dist/safetyhook.hpp)
	set(AMALGAMATE_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/amalgamate.py)
	add_custom_command(
	    OUTPUT ${AMALGAMATED_FILE} ${AMALGAMATED_HEADER}
	    DEPENDS ${HEADER_FILES} ${SOURCE_FILES} ${AMALGAMATE_SCRIPT}
	    COMMAND ${Python3_EXECUTABLE} ${AMALGAMATE_SCRIPT} ${AMALGAMATED_FILE} ${AMALGAMATED_HEADER}
	    MAIN_DEPENDENCY ${AMALGAMATE_SCRIPT}
	    COMMENT "Amalgamating"
	)
	add_custom_target(Amalgamate ALL DEPENDS ${AMALGAMATED_FILE} ${AMALGAMATED_HEADER})

	set(amalgamation_SOURCES
		cmake.toml
	)

	add_custom_target(amalgamation SOURCES)

	target_sources(amalgamation PRIVATE ${amalgamation_SOURCES})
	source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${amalgamation_SOURCES})

endif()
# Target: test0
if(SAFETYHOOK_BUILD_TESTS) # build-tests
	set(test0_SOURCES
		"tests/test0.cpp"
		cmake.toml
	)

	add_executable(test0)

	target_sources(test0 PRIVATE ${test0_SOURCES})
	source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${test0_SOURCES})

	target_compile_features(test0 PRIVATE
		cxx_std_23
	)

	if(MSVC) # msvc
		target_compile_options(test0 PRIVATE
			"/WX"
			"/permissive-"
			"/W4"
			"/w14640"
			"/EHsc"
		)
	endif()

	if((CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_CXX_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$") OR (CMAKE_C_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_C_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$")) # clang
		target_compile_options(test0 PRIVATE
			-Werror
			-Wall
			-Wextra
			-Wshadow
			-Wnon-virtual-dtor
			-pedantic
		)
	endif()

	if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_C_COMPILER_ID STREQUAL "GNU") # gcc
		target_compile_options(test0 PRIVATE
			-Werror
			-Wall
			-Wextra
			-Wshadow
			-Wnon-virtual-dtor
			-pedantic
		)
	endif()

	target_link_libraries(test0 PRIVATE
		safetyhook::safetyhook
	)

	get_directory_property(CMKR_VS_STARTUP_PROJECT DIRECTORY ${PROJECT_SOURCE_DIR} DEFINITION VS_STARTUP_PROJECT)
	if(NOT CMKR_VS_STARTUP_PROJECT)
		set_property(DIRECTORY ${PROJECT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT test0)
	endif()

endif()
# Target: test1
if(SAFETYHOOK_BUILD_TESTS) # build-tests
	set(test1_SOURCES
		"tests/test1.cpp"
		cmake.toml
	)

	add_executable(test1)

	target_sources(test1 PRIVATE ${test1_SOURCES})
	source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${test1_SOURCES})

	target_compile_features(test1 PRIVATE
		cxx_std_23
	)

	if(MSVC) # msvc
		target_compile_options(test1 PRIVATE
			"/WX"
			"/permissive-"
			"/W4"
			"/w14640"
			"/EHsc"
		)
	endif()

	if((CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_CXX_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$") OR (CMAKE_C_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_C_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$")) # clang
		target_compile_options(test1 PRIVATE
			-Werror
			-Wall
			-Wextra
			-Wshadow
			-Wnon-virtual-dtor
			-pedantic
		)
	endif()

	if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_C_COMPILER_ID STREQUAL "GNU") # gcc
		target_compile_options(test1 PRIVATE
			-Werror
			-Wall
			-Wextra
			-Wshadow
			-Wnon-virtual-dtor
			-pedantic
		)
	endif()

	target_link_libraries(test1 PRIVATE
		safetyhook::safetyhook
	)

	get_directory_property(CMKR_VS_STARTUP_PROJECT DIRECTORY ${PROJECT_SOURCE_DIR} DEFINITION VS_STARTUP_PROJECT)
	if(NOT CMKR_VS_STARTUP_PROJECT)
		set_property(DIRECTORY ${PROJECT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT test1)
	endif()

endif()
# Target: test2
if(SAFETYHOOK_BUILD_TESTS) # build-tests
	set(test2_SOURCES
		"tests/test2.cpp"
		cmake.toml
	)

	add_executable(test2)

	target_sources(test2 PRIVATE ${test2_SOURCES})
	source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${test2_SOURCES})

	target_compile_features(test2 PRIVATE
		cxx_std_23
	)

	if(MSVC) # msvc
		target_compile_options(test2 PRIVATE
			"/WX"
			"/permissive-"
			"/W4"
			"/w14640"
			"/EHsc"
		)
	endif()

	if((CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_CXX_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$") OR (CMAKE_C_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_C_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$")) # clang
		target_compile_options(test2 PRIVATE
			-Werror
			-Wall
			-Wextra
			-Wshadow
			-Wnon-virtual-dtor
			-pedantic
		)
	endif()

	if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_C_COMPILER_ID STREQUAL "GNU") # gcc
		target_compile_options(test2 PRIVATE
			-Werror
			-Wall
			-Wextra
			-Wshadow
			-Wnon-virtual-dtor
			-pedantic
		)
	endif()

	target_link_libraries(test2 PRIVATE
		safetyhook::safetyhook
	)

	get_directory_property(CMKR_VS_STARTUP_PROJECT DIRECTORY ${PROJECT_SOURCE_DIR} DEFINITION VS_STARTUP_PROJECT)
	if(NOT CMKR_VS_STARTUP_PROJECT)
		set_property(DIRECTORY ${PROJECT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT test2)
	endif()

endif()
# Target: test3
if(SAFETYHOOK_BUILD_TESTS) # build-tests
	set(test3_SOURCES
		"tests/test3.cpp"
		cmake.toml
	)

	add_executable(test3)

	target_sources(test3 PRIVATE ${test3_SOURCES})
	source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${test3_SOURCES})

	target_compile_features(test3 PRIVATE
		cxx_std_23
	)

	if(MSVC) # msvc
		target_compile_options(test3 PRIVATE
			"/WX"
			"/permissive-"
			"/W4"
			"/w14640"
			"/EHsc"
		)
	endif()

	if((CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_CXX_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$") OR (CMAKE_C_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_C_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$")) # clang
		target_compile_options(test3 PRIVATE
			-Werror
			-Wall
			-Wextra
			-Wshadow
			-Wnon-virtual-dtor
			-pedantic
		)
	endif()

	if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_C_COMPILER_ID STREQUAL "GNU") # gcc
		target_compile_options(test3 PRIVATE
			-Werror
			-Wall
			-Wextra
			-Wshadow
			-Wnon-virtual-dtor
			-pedantic
		)
	endif()

	target_link_libraries(test3 PRIVATE
		safetyhook::safetyhook
	)

	get_directory_property(CMKR_VS_STARTUP_PROJECT DIRECTORY ${PROJECT_SOURCE_DIR} DEFINITION VS_STARTUP_PROJECT)
	if(NOT CMKR_VS_STARTUP_PROJECT)
		set_property(DIRECTORY ${PROJECT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT test3)
	endif()

endif()
# Target: test4
if(SAFETYHOOK_BUILD_TESTS) # build-tests
	set(test4_SOURCES
		"tests/test4.cpp"
		cmake.toml
	)

	add_library(test4 SHARED)

	target_sources(test4 PRIVATE ${test4_SOURCES})
	source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${test4_SOURCES})

	target_compile_features(test4 PUBLIC
		cxx_std_23
	)

	if(MSVC) # msvc
		target_compile_options(test4 PRIVATE
			"/WX"
			"/permissive-"
			"/W4"
			"/w14640"
			"/EHsc"
		)
	endif()

	if((CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_CXX_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$") OR (CMAKE_C_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_C_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$")) # clang
		target_compile_options(test4 PRIVATE
			-Werror
			-Wall
			-Wextra
			-Wshadow
			-Wnon-virtual-dtor
			-pedantic
		)
	endif()

	if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_C_COMPILER_ID STREQUAL "GNU") # gcc
		target_compile_options(test4 PRIVATE
			-Werror
			-Wall
			-Wextra
			-Wshadow
			-Wnon-virtual-dtor
			-pedantic
		)
	endif()

	target_link_libraries(test4 PUBLIC
		safetyhook::safetyhook
	)

endif()
# Target: unittest
if(SAFETYHOOK_BUILD_TESTS) # build-tests
	set(unittest_SOURCES
		"unittest/allocator.cpp"
		"unittest/inline_hook.cpp"
		"unittest/inline_hook.x86_64.cpp"
		"unittest/mid_hook.cpp"
		"unittest/vmt_hook.cpp"
		cmake.toml
	)

	add_executable(unittest)

	target_sources(unittest PRIVATE ${unittest_SOURCES})
	source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${unittest_SOURCES})

	target_compile_features(unittest PRIVATE
		cxx_std_23
	)

	if(MSVC) # msvc
		target_compile_options(unittest PRIVATE
			"/WX"
			"/permissive-"
			"/W4"
			"/w14640"
			"/EHsc"
		)
	endif()

	if((CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_CXX_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$") OR (CMAKE_C_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_C_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$")) # clang
		target_compile_options(unittest PRIVATE
			-Werror
			-Wall
			-Wextra
			-Wshadow
			-Wnon-virtual-dtor
			-pedantic
		)
	endif()

	if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_C_COMPILER_ID STREQUAL "GNU") # gcc
		target_compile_options(unittest PRIVATE
			-Werror
			-Wall
			-Wextra
			-Wshadow
			-Wnon-virtual-dtor
			-pedantic
		)
	endif()

	target_link_libraries(unittest PRIVATE
		safetyhook::safetyhook
	)

	target_link_libraries(unittest PRIVATE
		Catch2::Catch2WithMain
		safetyhook::safetyhook
		xbyak::xbyak
	)

	get_directory_property(CMKR_VS_STARTUP_PROJECT DIRECTORY ${PROJECT_SOURCE_DIR} DEFINITION VS_STARTUP_PROJECT)
	if(NOT CMKR_VS_STARTUP_PROJECT)
		set_property(DIRECTORY ${PROJECT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT unittest)
	endif()

endif()
# Target: unittest-amalgamated
if(SAFETYHOOK_BUILD_TESTS) # build-tests
	if(SAFETYHOOK_BUILD_TESTS AND SAFETYHOOK_AMALGAMATE) # build-amalgamate-tests
		set(unittest-amalgamated_SOURCES
			"unittest/allocator.cpp"
			"unittest/inline_hook.cpp"
			"unittest/inline_hook.x86_64.cpp"
			"unittest/mid_hook.cpp"
			"unittest/vmt_hook.cpp"
			"amalgamated-dist/safetyhook.cpp"
			cmake.toml
		)

		add_executable(unittest-amalgamated)

		target_sources(unittest-amalgamated PRIVATE ${unittest-amalgamated_SOURCES})
		source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${unittest-amalgamated_SOURCES})

		target_compile_features(unittest-amalgamated PRIVATE
			cxx_std_23
		)

		if(MSVC) # msvc
			target_compile_options(unittest-amalgamated PRIVATE
				"/WX"
				"/permissive-"
				"/W4"
				"/w14640"
				"/EHsc"
			)
		endif()

		if((CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_CXX_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$") OR (CMAKE_C_COMPILER_ID MATCHES "Clang" AND NOT CMAKE_C_COMPILER_FRONTEND_VARIANT MATCHES "^MSVC$")) # clang
			target_compile_options(unittest-amalgamated PRIVATE
				-Werror
				-Wall
				-Wextra
				-Wshadow
				-Wnon-virtual-dtor
				-pedantic
			)
		endif()

		if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_C_COMPILER_ID STREQUAL "GNU") # gcc
			target_compile_options(unittest-amalgamated PRIVATE
				-Werror
				-Wall
				-Wextra
				-Wshadow
				-Wnon-virtual-dtor
				-pedantic
			)
		endif()

		target_link_libraries(unittest-amalgamated PRIVATE
			safetyhook::safetyhook
		)

		target_include_directories(unittest-amalgamated PRIVATE
			"amalgamated-dist/"
		)

		target_link_libraries(unittest-amalgamated PRIVATE
			Zydis
			Catch2::Catch2WithMain
			xbyak::xbyak
		)

		get_directory_property(CMKR_VS_STARTUP_PROJECT DIRECTORY ${PROJECT_SOURCE_DIR} DEFINITION VS_STARTUP_PROJECT)
		if(NOT CMKR_VS_STARTUP_PROJECT)
			set_property(DIRECTORY ${PROJECT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT unittest-amalgamated)
		endif()

		set(CMKR_TARGET unittest-amalgamated)
		add_dependencies(unittest-amalgamated Amalgamate)

	endif()
endif()
