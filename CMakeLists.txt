cmake_minimum_required(VERSION 3.7.2)

set(CMAKE_USE_RELATIVE_PATHS TRUE)
project("optionparser")
set("PROJECT_DESCRIPTION" "The Lean Mean C++ Option Parser")
set("PROJECT_HOMEPAGE_URL" "https://sourceforge.net/projects/optionparser/") 

set(CPACK_PACKAGE_VENDOR "Matthias S. Benkmann")
set(CPACK_PACKAGE_VERSION_MAJOR "1")
set(CPACK_PACKAGE_VERSION_MINOR "7")
set(CPACK_PACKAGE_VERSION_PATCH "0")
set(CPACK_PACKAGE_VERSION "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")

set(OUR_CMAKE_MODULES_DIR "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
set(OUR_CMAKE_3PARTY_MODULES_DIR "${OUR_CMAKE_MODULES_DIR}/thirdParty")
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${OUR_CMAKE_MODULES_DIR}" "${OUR_CMAKE_3PARTY_MODULES_DIR}" "${OUR_CMAKE_3PARTY_MODULES_DIR}/Boilerplate" "${OUR_CMAKE_3PARTY_MODULES_DIR}/DoxygenUtils")


include(Boilerplate)

set(Include_dir "${CMAKE_CURRENT_SOURCE_DIR}/include")
set(Examples_Dir "${CMAKE_CURRENT_SOURCE_DIR}/examples")

buildAndPackageLib(${PROJECT_NAME}
	TYPE INTERFACE
	COMPONENT "lib"
	DESCRIPTION "${PROJECT_DESCRIPTION}"
	CMAKE_EXPORT_NAMESPACE "mbenkmann"
	PUBLIC_INCLUDES ${Include_dir}
	DO_NOT_PASSTHROUGH
)

option(WITH_EXAMPLES "Build examples" OFF)
if(WITH_EXAMPLES)
	file(GLOB_RECURSE EXAMPLES "${Examples_Dir}/*.cpp")
	foreach(example ${EXAMPLES})
		get_filename_component(exampleName "${example}" NAME_WE)
		add_executable("${exampleName}" "${example}")
		target_include_directories("${exampleName}" PRIVATE "${Include_dir}")
	endforeach()
endif()

option(WITH_DOCS "Build docs" ON)
if(WITH_DOCS)
	find_package(Doxygen REQUIRED dot)

	set(DOXYGEN_PROJECT_BRIEF "${CPACK_PACKAGE_DESCRIPTION}")
	set(DOXYGEN_USE_MDFILE_AS_MAINPAGE "${CPACK_RESOURCE_FILE_README}")

	set("DOXYGEN_GENERATE_HTML" YES)
	set("DOXYGEN_GENERATE_MAN" YES)

	if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
		#message(STATUS "Compiler is CLang, enabling CLang-assisted parsing in Doxygen.")
		#set(DOXYGEN_CLANG_ASSISTED_PARSING YES)
		#set(DOXYGEN_CLANG_OPTIONS "-I${Include_dir}")
	endif()

	set(DOXYGEN_EXAMPLE_PATH "${Examples_Dir}")
	set(DOXYGEN_STRIP_FROM_PATH "${Include_dir}")
	set(DOXYGEN_STRIP_FROM_INC_PATH "${Include_dir}")


	file(GLOB HEADERS "${Include_dir}/*.h" "${Include_dir}/*.hxx" "${Include_dir}/*.hpp")
	doxygen_add_docs(docs
		"${HEADERS}"
		ALL
		USE_STAMP_FILE
	)
endif()

#pass_through_cpack_vars()

include(CPack)
