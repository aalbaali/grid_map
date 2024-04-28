cmake_minimum_required(VERSION 3.5.1)
project(grid_map_cv)

set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
add_compile_options(-Wall -Wextra -Wpedantic)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

find_package(GridMapCore REQUIRED)
find_package(Eigen3 QUIET)

if(NOT EIGEN3_FOUND)
  find_package(PkgConfig REQUIRED)
  pkg_check_modules(EIGEN3 REQUIRED eigen3)
  set(EIGEN3_INCLUDE_DIR ${EIGEN3_INCLUDE_DIRS})
endif()

find_package(Eigen3 REQUIRED)
find_package(OpenCV REQUIRED)

###########
## Build ##
###########

## Declare a cpp library
add_library(GridMapCv
  src/GridMapCvProcessing.cpp
  #src/InpaintFilter.cpp
)
target_link_libraries(GridMapCv
  GridMap::GridMapCore
  Eigen3::Eigen
  ${OpenCV_LIBRARIES}
)
target_include_directories(
  GridMapCv
  PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>  
  $<INSTALL_INTERFACE:include>
)

#############
## Install ##
#############

# Mark executables and/or libraries for installation
install(
  TARGETS GridMapCv
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

#############
## Testing ##
#############

if(CATKIN_ENABLE_TESTING)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")

  ## Add gtest based cpp test target and link libraries
  catkin_add_gtest(GridMapCv-test
    test/test_grid_map_cv.cpp
    test/GridMapCvTest.cpp
    test/GridMapCvProcessingTest.cpp
  )
  target_include_directories(GridMapCv-test PRIVATE
    include
  )
  target_include_directories(GridMapCv-test SYSTEM PUBLIC
    ${catkin_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIR}
  )
  target_link_libraries(GridMapCv-test
    GridMapCv
  )

  ###################
  ## Code_coverage ##
  ###################
  find_package(cmake_code_coverage QUIET)
  if(cmake_code_coverage_FOUND)
    add_gtest_coverage(
      TEST_BUILD_TARGETS
        GridMapCv-test
    )
  endif()
endif()

#################
## Clang_tools ##
#################
find_package(cmake_clang_tools QUIET)
if(cmake_clang_tools_FOUND)
  add_default_clang_tooling(
    DISABLE_CLANG_FORMAT
  )
endif(cmake_clang_tools_FOUND)

########################
## Installing targets ##
########################
# Export package for use from the build tree
set(CMAKE_EXPORT_DIR lib/cmake/GridMapCv)
set(TARGETS_NAME "GridMapCv")
set(TARGETS_NAMESPACE "GridMap")
set(TARGETS_CONFIG_FILENAME "${TARGETS_NAME}Config.cmake.in")

install(
  TARGETS "${TARGETS_NAME}"
  EXPORT  "${TARGETS_NAME}Targets"
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib 
)
install(EXPORT "${TARGETS_NAME}Targets" 
  NAMESPACE "${TARGETS_NAMESPACE}::"
  DESTINATION ${CMAKE_EXPORT_DIR}
)

# Install the include directory
install(
  DIRECTORY ${PROJECT_SOURCE_DIR}/include/
  DESTINATION include
)
include(CMakePackageConfigHelpers)
configure_package_config_file(
  "${TARGETS_CONFIG_FILENAME}" "${CMAKE_CURRENT_BINARY_DIR}/${TARGETS_NAME}Config.cmake"
  INSTALL_DESTINATION "${CMAKE_EXPORT_DIR}"
)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${TARGETS_NAME}Config.cmake" DESTINATION "${CMAKE_EXPORT_DIR}")

# Export without installing
export(
  EXPORT "${TARGETS_NAME}Targets"
  FILE "${CMAKE_CURRENT_BINARY_DIR}/${TARGETS_NAME}Targets.cmake"
  NAMESPACE "${TARGETS_NAMESPACE}::"
)

