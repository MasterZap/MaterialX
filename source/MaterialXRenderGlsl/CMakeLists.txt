include_directories(
    ${EXTERNAL_INCLUDE_DIRS}
    ${CMAKE_CURRENT_SOURCE_DIR}/../)

# Linux checks for X and OpenGL
if(UNIX AND NOT APPLE)
	find_package(PkgConfig REQUIRED)
	find_package(X11 REQUIRED)
	if(NOT X11_FOUND)
		message("Error in building MaterialXRender: X11 was not found")
	endif(NOT X11_FOUND)
	if(NOT X11_Xt_FOUND)
		message("Error in building MaterialXRender: Xt was not found")
	endif(NOT X11_Xt_FOUND)
	find_package(OpenGL REQUIRED)
	if(NOT OPENGL_FOUND)
		message("Error in building MaterialXRender: OpenGL was not found")
	endif(NOT OPENGL_FOUND)
	include_directories(${X11_INCLUDE_DIR})
endif(UNIX AND NOT APPLE)

file(GLOB_RECURSE materialx_source "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp")
file(GLOB_RECURSE materialx_headers "${CMAKE_CURRENT_SOURCE_DIR}/*.h*")

if(APPLE)
	find_library(COCOA_FRAMEWORK Cocoa)
	find_package(OpenGL REQUIRED)
	file(GLOB_RECURSE materialx_source_oc "${CMAKE_CURRENT_SOURCE_DIR}/*.m")
	message("Objective C files: " ${materialx_source_oc})
	set_source_files_properties(${materialx_source_oc} PROPERTIES
		COMPILE_FLAGS "-x objective-c++")
	set(materialx_source ${materialx_source} ${materialx_source_oc})
endif(APPLE)

assign_source_group("Source Files" ${materialx_source})
assign_source_group("Header Files" ${materialx_headers})

add_library(MaterialXRenderGlsl STATIC ${materialx_source} ${materialx_headers})

if(MSVC)
    target_link_libraries(
        MaterialXRenderGlsl
        MaterialXRenderHw
        Opengl32
        ${CMAKE_DL_LIBS})
elseif(APPLE)
    target_link_libraries(
        MaterialXRenderGlsl
        MaterialXRenderHw
        ${CMAKE_DL_LIBS}
        ${OPENGL_LIBRARIES}
	    "-framework Foundation"
      	"-framework Cocoa")
elseif(UNIX)
    target_link_libraries(
        MaterialXRenderGlsl
        MaterialXRenderHw
        ${CMAKE_DL_LIBS}
        ${OPENGL_LIBRARIES}
        ${X11_LIBRARIES}
        ${X11_Xt_LIB})
endif(MSVC)

set_target_properties(
    MaterialXRenderGlsl PROPERTIES
    OUTPUT_NAME MaterialXRenderGlsl
    COMPILE_FLAGS "${EXTERNAL_COMPILE_FLAGS}"
    LINK_FLAGS "${EXTERNAL_LINK_FLAGS}"
    VERSION "${MATERIALX_LIBRARY_VERSION}"
    SOVERSION "${MATERIALX_MAJOR_VERSION}")

install(TARGETS MaterialXRenderGlsl
    DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/)

install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/"
    DESTINATION ${CMAKE_INSTALL_PREFIX}/include/MaterialXRenderGlsl/ MESSAGE_NEVER
    FILES_MATCHING PATTERN "*.h*")

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_BUILD_TYPE}/MaterialXRenderGlsl.pdb"
    DESTINATION "${CMAKE_INSTALL_PREFIX}/lib/" OPTIONAL)
