cmake_minimum_required(VERSION 3.23)

project(space CXX)

set(CMAKE_BUILD_TYPE Debug)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)

file (GLOB_RECURSE SOURCES "Src/*.cpp" "Src/*.c" "Src/*.mm" "Src/*.inl")
file (GLOB_RECURSE HEADERS "Src/*.hpp" "Src/*.h" "Src/*.inl")

add_definitions(
	-D_USE_MATH_DEFINES
	-DASSET_BASE_DIR="."
    # -DFX_DEBUG_STATIC_ARRAY
)

if (APPLE)
	add_compile_options(
		-O2
		# -fsanitize=address
	)

	add_link_options(
		# -fsanitize=address
	)
endif()

if(MSVC)
	add_compile_options("")
endif()

include_directories(
    ${CMAKE_SOURCE_DIR}/Src
    ${CMAKE_SOURCE_DIR}/Src/ThirdParty
    ${CMAKE_SOURCE_DIR}/Src/ThirdParty/imgui
    ${CMAKE_SOURCE_DIR}/Src/ThirdParty/imgui/backends
)
if(MSVC)
link_directories(${CMAKE_SOURCE_DIR}/Lib/Windows)
elseif(NOT (UNIX AND NOT APPLE))
link_directories(${CMAKE_SOURCE_DIR}/Lib)
endif()

if (UNIX AND NOT APPLE)
	# Objective-C++ backends (imgui_impl_osx.mm) only build on macOS.
	list(FILTER SOURCES EXCLUDE REGEX "\\.mm$")
	list(FILTER HEADERS EXCLUDE REGEX "\\.mm$")
endif()

add_executable(space ${SOURCES} ${HEADERS})

set (LIBS raylib )

if (MSVC)
    set (LIBS ${LIBS} winmm)

endif()

if (UNIX AND NOT APPLE)
	# Linux: never link the bundled macOS Lib/libraylib.a. Use a system raylib 5.5
	# if one is installed, otherwise build raylib 5.5 from source.
	enable_language(C)

	find_package(raylib 5.5 QUIET)
	if (NOT raylib_FOUND)
		message(STATUS "raylib 5.5 not found on the system, fetching it with FetchContent")
		include(FetchContent)
		set(BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
		set(BUILD_GAMES OFF CACHE BOOL "" FORCE)
		set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
		FetchContent_Declare(
			raylib
			URL https://github.com/raysan5/raylib/archive/refs/tags/5.5.tar.gz
			URL_HASH SHA256=aea98ecf5bc5c5e0b789a76de0083a21a70457050ea4cc2aec7566935f5e258e
			DOWNLOAD_EXTRACT_TIMESTAMP TRUE
		)
		FetchContent_MakeAvailable(raylib)
	else()
		message(STATUS "Using system raylib ${raylib_VERSION}")
	endif()

	set(OpenGL_GL_PREFERENCE GLVND)
	find_package(OpenGL REQUIRED)
	find_package(Threads REQUIRED)
	find_package(X11)

	set (LIBS ${LIBS} OpenGL::GL Threads::Threads ${CMAKE_DL_LIBS} m)
	if (X11_FOUND)
		set (LIBS ${LIBS} X11::X11)
	endif()
endif()

target_link_libraries(space ${LIBS})
