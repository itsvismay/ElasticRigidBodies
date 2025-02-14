cmake_minimum_required(VERSION 2.8.6)

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

option(LIBIGL_VIEWER_WITH_NANOGUI "Enable nanogui " ON)

# We need C++11. Put this directive after CGAL's include.
if(NOT WIN32)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -g -Wall -Wno-reorder -Wno-unknown-pragmas -Wno-deprecated-declarations")
endif()

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
endif()

find_package(EIGEN REQUIRED)
include_directories( "${EIGEN_INCLUDE_DIR}" )
# Must be found before trying to find libraries in libigl/external
find_package(LIBIGL REQUIRED)
include_directories( "${LIBIGL_INCLUDE_DIR}" )
find_package(OpenGL REQUIRED)
find_package(GLFWH REQUIRED)
include_directories( ${GLFW_INCLUDE_DIR})
find_package(TETGEN REQUIRED)
include_directories( ${TETGEN_INCLUDE_DIR})

find_package(SUITESPARSE QUIET)
include_directories(${SUITESPARSE_INCLUDE_DIRS})

find_package(LIBLBFGS QUIET)
include_directories(${LIBLBFGS_INCLUDE_DIRS})


if(LIBIGL_VIEWER_WITH_NANOGUI)
  add_definitions(-DIGL_VIEWER_WITH_NANOGUI)
  set(NANOGUI_BUILD_PYTHON OFF CACHE BOOL " " FORCE)
  set(NANOGUI_BUILD_EXAMPLE OFF CACHE BOOL " " FORCE)
  set(NANOGUI_BUILD_SHARED OFF CACHE BOOL " " FORCE)
  find_package(NANOGUI REQUIRED)
  add_subdirectory(   "${NANOGUI_INCLUDE_DIR}/../" "nanogui")
  include_directories("${NANOGUI_INCLUDE_DIR}/../include")
  include_directories("${NANOGUI_INCLUDE_DIR}/../ext/nanovg/src")
  include_directories(${OPENGL_INCLUDE_DIR})
  list(APPEND LIBS "nanogui")
else()
  # still need to build GLFW
  set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL " " FORCE)
  set(GLFW_BUILD_TESTS OFF CACHE BOOL " " FORCE)
  set(GLFW_BUILD_DOCS OFF CACHE BOOL " " FORCE)
  set(GLFW_BUILD_INSTALL OFF CACHE BOOL " " FORCE)
  add_subdirectory("${GLFW_INCLUDE_DIR}/../" "glfw")
  list(APPEND SHARED_LIBRARIES "glfw")
endif()

if(APPLE)
  set(CMAKE_CXX_LINK_FLAGS "-framework OpenGL -framework Cocoa -framework IOKit -framework CoreVideo")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native")
ELSE()
  find_package(GLEW REQUIRED)
  include_directories( ${GLEW_INCLUDE_DIR})
endif()

link_directories(
  /usr/local/lib
  /opt/local/lib
  ${EIGEN_DIRS}
)

set(LIBS 
  ${LIBS}
  ${LIBIGL_LIBRARIES}
  ${OPENGL_LIBRARIES}
  ${GLEW_LIBRARIES}
  ${NANOGUI_LIBRARIES}
  ${TETGEN_LIBRARIES}
  ${SUITESPARSE_LIBRARIES}
  ${LIBLBFGS_LIBRARIES}
)

if(UNIX AND NOT APPLE)
  set(LIBS ${LIBS} ${LIBS})
  set(CMAKE_CXX_LINK_EXECUTABLE "${CMAKE_CXX_LINK_EXECUTABLE} -lGL -lGLU -lrt -lX11 -lXxf86vm -lXrandr -lpthread -lXi  -lXcursor -lXinerama -ldl")
endif(UNIX AND NOT APPLE)

FILE(GLOB HEADFILES *.h)
FILE(GLOB SRCFILES *.cpp)
add_executable(elastic ${HEADFILES} ${SRCFILES} ${GLEW_SOURCES} ${TETGEN_SOURCES})
target_link_libraries(elastic ${LIBS})
