project(FooBar VERSION 1.0.0 LANGUAGES CXX)

if(MSVC)
# Second, for multi-config builds (e.g. msvc)
foreach(OUTPUTCONFIG ${CMAKE_CONFIGURATION_TYPES})
    string(TOUPPER ${OUTPUTCONFIG} OUTPUTCONFIG)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${CMAKE_CURRENT_BINARY_DIR})
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${CMAKE_CURRENT_BINARY_DIR})
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_${OUTPUTCONFIG} ${CMAKE_CURRENT_BINARY_DIR})
endforeach()
endif()

find_target(CMakeSwig::Foo CMakeSwig REQUIRED)
find_target(CMakeSwig::Bar CMakeSwig REQUIRED)

file(GLOB _HDRS "include/foobar/*.hpp" "include/foobar/*.hxx")
file(GLOB _SRCS "src/*.[hc]pp")

#add_executable(${PROJECT_NAME} ${_HDRS} ${_SRCS})
add_library(${PROJECT_NAME} ${_HDRS} src/FooBar.cpp)
target_include_directories(${PROJECT_NAME} PRIVATE
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	$<INSTALL_INTERFACE:include>)
set_target_properties(FooBar PROPERTIES PUBLIC_HEADER ${_HDRS})
if(APPLE)
	set_target_properties(${PROJECT_NAME} PROPERTIES
		INSTALL_RPATH "@loader_path/../Foo;@loader_path/../Bar;@loader_path")
elseif(UNIX AND NOT APPLE)
	set_target_properties(${PROJECT_NAME} PROPERTIES
		INSTALL_RPATH "$ORIGIN/../Foo:$ORIGIN/../Bar:$ORIGIN")
endif()
target_link_libraries(${PROJECT_NAME} PUBLIC CMakeSwig::Foo PRIVATE CMakeSwig::Bar)
add_library(CMakeSwig::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

# Add Python Wrapper
add_subdirectory(python)

# Install
include(GNUInstallDirs)
install(TARGETS ${PROJECT_NAME}
	EXPORT CMakeSwigTargets
	PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/foobar
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	#RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	)
