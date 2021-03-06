cmake_minimum_required( VERSION 3.12 )

project(imgui_addons
	VERSION 1.7.8)

include(CMakeDependentOption)

option(IMGUI_BUILD_EXAMPLES "Build examples." OFF)

# ---- Add dependencies via CPM ----
# see https://github.com/TheLartians/CPM.cmake for more info

include(ExternalProject)

####

if (NOT DEFINED cpmpckg_SOURCE_DIR)
	include(cmake/CPM.cmake)

	# One frustrating part of this project setup is the circular dependency between cpmpckg and this repo.
	# GIT_TAG will always lag cpmpckg @ HEAD when this project is updated there.
	CPMAddPackage(
		NAME cpmpckg
		GITHUB_REPOSITORY loopunit/cpmpckg
		GIT_TAG d92021769936a770f8eccc41b19e24f0bf606ff6
		DOWNLOAD_ONLY true)

	include(${cpmpckg_SOURCE_DIR}/cmake/add_cpm_module.cmake)
else()
	set(CPM_SCRIPTS ${cpmpckg_SOURCE_DIR}/cmake)
	include(${cpmpckg_SOURCE_DIR}/cmake/CPM.cmake)
	include(${cpmpckg_SOURCE_DIR}/cmake/add_cpm_module.cmake)
endif()

####

CPMAddPackage(
  NAME PackageProject.cmake
  GITHUB_REPOSITORY loopunit/PackageProject.cmake
  GIT_TAG e5ec20069766f4f078f9f01a86e250e20da0817c)

####

CPMAddBaseModule(imgui)

####

CPMAddPackage(
	NAME implot
	GITHUB_REPOSITORY epezent/implot
	GIT_TAG 862b8c44ea6d62d736e320b7a11be8438ecbac5d
	DOWNLOAD_ONLY true)

if (NOT implot_ADDED)
	message( FATAL_ERROR "Could not find implot" )
endif()

file(GLOB implot_public_headers 
	"${implot_SOURCE_DIR}/implot.h"
	"${implot_SOURCE_DIR}/implot_internal.h"
)

file(GLOB implot_source
	"${implot_SOURCE_DIR}/*.cpp"
)

#############

CPMAddPackage(
	NAME ImGuiColorTextEdit
	GITHUB_REPOSITORY BalazsJako/ImGuiColorTextEdit
	GIT_TAG 0a88824f7de8d0bd11d8419066caa7d3469395c4
	DOWNLOAD_ONLY true)

if (NOT ImGuiColorTextEdit_ADDED)
	message( FATAL_ERROR "Could not find ImGuiColorTextEdit" )
endif()

file(GLOB ImGuiColorTextEdit_public_headers 
	"${ImGuiColorTextEdit_SOURCE_DIR}/TextEditor.h"
)

file(GLOB ImGuiColorTextEdit_source
	"${ImGuiColorTextEdit_SOURCE_DIR}/*.cpp"
)

#############

CPMAddPackage(
	NAME imgui_markdown
	GITHUB_REPOSITORY juliettef/imgui_markdown
	GIT_TAG 2b33c435b4463999438a5532f1dc201c142a2aab
	DOWNLOAD_ONLY true)

if (NOT imgui_markdown_ADDED)
	message( FATAL_ERROR "Could not find imgui_markdown" )
endif()

file(GLOB imgui_markdown_public_headers 
	"${imgui_markdown_SOURCE_DIR}/imgui_markdown.h"
)

#############

CPMAddPackage(
	NAME imgui_node_editor
	GITHUB_REPOSITORY thedmd/imgui-node-editor
	GIT_TAG 687a72f940c76cf5064e13fe55fa0408c18fcbe4
	DOWNLOAD_ONLY true)

if (NOT imgui_node_editor_ADDED)
	message( FATAL_ERROR "Could not find imgui_node_editor" )
endif()

file(GLOB imgui_node_editor_public_headers 
	"${imgui_node_editor_SOURCE_DIR}/*.h"
	"${imgui_node_editor_SOURCE_DIR}/*.inl"
)

file(GLOB imgui_node_editor_source
	"${imgui_node_editor_SOURCE_DIR}/*.cpp"
)

#############

CPMAddPackage(
	NAME imgui_console
	GITHUB_REPOSITORY rmxbalanque/imgui-console
	GIT_TAG a9a1f83b016c3b455f2b88361aa2b79f25f3f961
	DOWNLOAD_ONLY true)


if (NOT imgui_console_ADDED)
	message( FATAL_ERROR "Could not find imgui_console" )
endif()

file(GLOB imgui_console_public_headers 
	"${imgui_console_SOURCE_DIR}/include/imgui_console/imgui_console.h"
)

file(GLOB imgui_console_public_headers_csys
	"${imgui_console_SOURCE_DIR}/include/csys/*.*"
)

file(GLOB imgui_console_source
	"${imgui_console_SOURCE_DIR}/src/imgui_console.cpp"
)

#############

CPMAddPackage(
	NAME ImGuiFileDialog
	GITHUB_REPOSITORY aiekick/ImGuiFileDialog
	GIT_TAG 27cf103c3be5fe11084a07c902c4c180d4e0fff6
	DOWNLOAD_ONLY true)

if (NOT ImGuiFileDialog_ADDED)
	message( FATAL_ERROR "Could not find ImGuiFileDialog" )
endif()

file(GLOB ImGuiFileDialog_public_headers 
	"${ImGuiFileDialog_SOURCE_DIR}/ImGuiFileDialog/ImGuiFileDialog.h"
	"${ImGuiFileDialog_SOURCE_DIR}/ImGuiFileDialog/ImGuiFileDialogConfig.h"
	"${ImGuiFileDialog_SOURCE_DIR}/3rdparty/dirent/include/dirent.h")

file(GLOB ImGuiFileDialog_source
	"${ImGuiFileDialog_SOURCE_DIR}/ImGuiFileDialog/*.cpp")

#############

set(STAGING_ROOT ${CMAKE_CURRENT_BINARY_DIR}/imgui_addons_staging)

file(COPY 
	${implot_public_headers}
	${ImGuiColorTextEdit_public_headers}
	${imgui_markdown_public_headers}
	${imgui_node_editor_public_headers}
	${ImGuiFileDialog_public_headers} 
	${imgui_console_public_headers}
	DESTINATION ${STAGING_ROOT}/include)

file(COPY 
	${ImTerm_public_headers}
	DESTINATION ${STAGING_ROOT}/include/ImTerm)

file(COPY 
	${imgui_console_public_headers_csys}
	DESTINATION ${STAGING_ROOT}/include/csys)


file(COPY 
	${implot_source}
	${ImGuiColorTextEdit_source}
	${imgui_node_editor_source}
	${ImGuiFileDialog_source}
	${imgui_console_source}
	DESTINATION ${STAGING_ROOT}/src)

file(GLOB sources ${STAGING_ROOT}/src/*.*)

add_library(imgui_addons STATIC ${sources})

set_target_properties(imgui_addons PROPERTIES CXX_STANDARD 20)

target_link_libraries(imgui_addons PRIVATE cpm_install::imgui)

target_include_directories(imgui_addons 
	PUBLIC 
		$<BUILD_INTERFACE:${STAGING_ROOT}/include>
		$<INSTALL_INTERFACE:include>)

packageProject(
	# the name of the target to export
	NAME imgui_addons
	# the version of the target to export
	VERSION ${PROJECT_VERSION}
	# a temporary directory to create the config files
	BINARY_DIR ${PROJECT_BINARY_DIR}
	# location of the target's public headers
	INCLUDE_DIR ${STAGING_ROOT}/include
	# should match the target's INSTALL_INTERFACE include directory
	INCLUDE_DESTINATION include
	# semicolon separated list of the project's dependencies
	DEPENDENCIES ""
)

if(MSVC)
  if("${CMAKE_BUILD_TYPE}" STREQUAL "Debug")
    set_target_properties(imgui_addons PROPERTIES IMPORTED_CONFIGURATIONS "Debug")
    set_target_properties(imgui_addons PROPERTIES COMPILE_PDB_NAME imgui_addons COMPILE_PDB_OUTPUT_DIRECTORY "${CMAKE_INSTALL_PREFIX}/lib" )
  elseif("${CMAKE_BUILD_TYPE}" STREQUAL "RelWithDebInfo")
    set_target_properties(imgui_addons PROPERTIES IMPORTED_CONFIGURATIONS "RelWithDebInfo")
    set_target_properties(imgui_addons PROPERTIES COMPILE_PDB_NAME imgui_addons COMPILE_PDB_OUTPUT_DIRECTORY "${CMAKE_INSTALL_PREFIX}/lib" )
  endif()
endif()

################

if(CPM_BUILD_TEST)
	CPMAddBaseModule(imgui_app_fw)

	file(GLOB addon_example_sources 
		${CMAKE_CURRENT_LIST_DIR}/examples/addon_main.cpp)

	add_executable(imgui_addons_example
		${addon_example_sources})

	set_target_properties(imgui_addons_example PROPERTIES CXX_STANDARD 20)

	target_include_directories(imgui_addons_example
		PRIVATE 
			${imgui_SOURCE_DIR}/examples)

	target_link_libraries(imgui_addons_example
		PUBLIC
			cpm_install::imgui_app_fw imgui_addons)
endif()