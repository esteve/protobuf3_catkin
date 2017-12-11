cmake_minimum_required(VERSION 2.8.3)
project(protobuf3_catkin)

find_package(catkin_simple REQUIRED)
catkin_simple()

include(ExternalProject)

file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)

ExternalProject_Add(protobuf3_src
  URL https://github.com/google/protobuf/releases/download/v3.5.0/protobuf-all-3.5.0.tar.gz
  UPDATE_COMMAND ""
  PATCH_COMMAND patch -p0 < ${CMAKE_SOURCE_DIR}/unused_parameter_warnings.patch &&
                patch -p0 < ${CMAKE_SOURCE_DIR}/unused_parameter_warnings_wire_format.patch
  CONFIGURE_COMMAND cd ../protobuf3_src && ./autogen.sh && ./configure --with-pic --prefix=${CATKIN_DEVEL_PREFIX}
  BUILD_COMMAND cd ../protobuf3_src && make -j8
  INSTALL_COMMAND cd ../protobuf3_src && make install -j8
)

cs_add_library(${PROJECT_NAME} src/dependency_tracker.cc)
add_dependencies(${PROJECT_NAME} protobuf3_src)
target_link_libraries(${PROJECT_NAME}
  ${CATKIN_DEVEL_PREFIX}/lib/libprotobuf-lite${CMAKE_SHARED_LIBRARY_SUFFIX}
  ${CATKIN_DEVEL_PREFIX}/lib/libprotoc${CMAKE_SHARED_LIBRARY_SUFFIX}
  ${CATKIN_DEVEL_PREFIX}/lib/libprotobuf${CMAKE_SHARED_LIBRARY_SUFFIX})

cs_install()

cs_export(
  INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
  CFG_EXTRAS protobuf3-generate-cpp.cmake protobuf3-extras.cmake)
