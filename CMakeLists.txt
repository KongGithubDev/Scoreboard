cmake_minimum_required (VERSION 3.15)

project ("Scoreboard"
	VERSION 1.2
)
# Output to 'output' folder relative to project root (works on any machine)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_SOURCE_DIR}/output")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE "${CMAKE_SOURCE_DIR}/output")
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)

# Suppress FetchContent_Populate deprecation warning (CMP0169)
cmake_policy(SET CMP0169 OLD)

# Suppress deprecation warnings from third-party dependencies
set(CMAKE_WARN_DEPRECATED OFF CACHE BOOL "" FORCE)

set(IMGUI_EXAMPLES OFF)
set(IMGUI_DEMO OFF)
set(IMGUI_IMPL_SDL OFF)
set(IMGUI_IMPL_GLFW OFF)
set(IMGUI_IMPL_GLUT OFF)
set(IMGUI_IMPL_OPENGL OFF)
set(IMGUI_IMPL_OPENGL2 OFF)
set(IMGUI_IMPL_DX9 ON)
set(IMGUI_IMPL_DX10 OFF)
set(IMGUI_IMPL_DX11 OFF)
set(IMGUI_IMPL_DX12 OFF)

include(FetchContent)

# Fix for CMake 4.x - use shallow clone and longer timeout
set(FETCHCONTENT_QUIET OFF)
set(FETCHCONTENT_FULLY_DISCONNECTED_TIMEOUT 300)

FetchContent_Declare(
	sampapi
	GIT_REPOSITORY https://github.com/BlastHackNet/SAMP-API.git
	GIT_TAG f89b35edb7eaf5db374e32a63f2366c4bb714418
	GIT_PROGRESS TRUE
)
FetchContent_Declare(
	xbyak
	GIT_REPOSITORY https://github.com/herumi/xbyak.git
	GIT_TAG 2a85bba3fe304c387d652537b715b09005e747f9
	PATCH_COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/cmake/xbyak_CMakeLists.txt CMakeLists.txt
	GIT_PROGRESS TRUE
)
FetchContent_Declare(
	polyhook
	GIT_REPOSITORY https://github.com/stevemk14ebr/PolyHook_2_0.git
	GIT_TAG master
	GIT_PROGRESS TRUE
)
FetchContent_Declare(
	imgui
	GIT_REPOSITORY https://github.com/ocornut/imgui.git
	GIT_TAG docking
	GIT_PROGRESS TRUE
)
FetchContent_Declare(
	ktsignal
	GIT_REPOSITORY https://github.com/kin4stat/ktsignal.git
	GIT_TAG f08a19d9a00ad5284871bf44178d835da7b26ebf
	GIT_PROGRESS TRUE
)
FetchContent_Declare(
	kthook
	GIT_REPOSITORY https://github.com/kin4stat/kthook.git
	GIT_TAG 21ca70481273d1efcc98ca49063178e46277009b
	GIT_PROGRESS TRUE
)
FetchContent_Declare(
	rakhook
	GIT_REPOSITORY https://github.com/imring/RakHook.git
	GIT_TAG 625fd17897395997feb56ae293c7c07dbe49d26b
	GIT_PROGRESS TRUE
)
FetchContent_MakeAvailable(sampapi kthook ktsignal rakhook)

# Suppress PolyHook compiler warnings
if(TARGET PolyHook_2)
	target_compile_options(PolyHook_2 PRIVATE /wd4100 /wd4244)
endif()

# Manual xbyak setup (header only)
FetchContent_GetProperties(xbyak)
if(NOT xbyak_POPULATED)
	FetchContent_Populate(xbyak)
endif()
if(NOT TARGET xbyak)
	add_library(xbyak INTERFACE)
	target_include_directories(xbyak INTERFACE ${xbyak_SOURCE_DIR})
	add_library(xbyak::xbyak ALIAS xbyak)
endif()

# Manual imgui setup
FetchContent_GetProperties(imgui)
if(NOT imgui_POPULATED)
	FetchContent_Populate(imgui)
endif()

if(NOT TARGET imgui)
	add_library(imgui STATIC
		${imgui_SOURCE_DIR}/imgui.cpp
		${imgui_SOURCE_DIR}/imgui_demo.cpp
		${imgui_SOURCE_DIR}/imgui_draw.cpp
		${imgui_SOURCE_DIR}/imgui_widgets.cpp
		${imgui_SOURCE_DIR}/imgui_tables.cpp
		${imgui_SOURCE_DIR}/backends/imgui_impl_dx9.cpp
		${imgui_SOURCE_DIR}/backends/imgui_impl_win32.cpp
	)
	target_include_directories(imgui PUBLIC ${imgui_SOURCE_DIR} ${imgui_SOURCE_DIR}/backends)
endif()

add_library(${PROJECT_NAME} SHARED)
target_link_libraries(${PROJECT_NAME} PUBLIC sampapi kthook rakhook imgui d3d9)
target_sources(${PROJECT_NAME} PRIVATE
	src/main.cpp
	src/Plugin.cpp
	src/PluginRender.cpp
	src/PluginGUI.cpp
	src/PluginRPC.cpp
	src/PlayerList.cpp
)

set_target_properties(${PROJECT_NAME}
	PROPERTIES
	OUTPUT_NAME ${PROJECT_NAME}
	SUFFIX ".asi"
)
target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_20)
add_compile_options("$<$<C_COMPILER_ID:MSVC>:/utf-8>")
add_compile_options("$<$<CXX_COMPILER_ID:MSVC>:/utf-8>")