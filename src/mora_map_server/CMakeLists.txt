cmake_minimum_required(VERSION 2.8.3)
project(mora_map_server)

find_package(catkin REQUIRED
        COMPONENTS
            roscpp
            tf
            nav_msgs
            std_msgs
        )

find_package(PkgConfig)
pkg_check_modules(NEW_YAMLCPP yaml-cpp>=0.5)
if(NEW_YAMLCPP_FOUND)
add_definitions(-DHAVE_NEW_YAMLCPP)
endif(NEW_YAMLCPP_FOUND)

catkin_package(
    INCLUDE_DIRS
        include
    LIBRARIES
        image_loader
    CATKIN_DEPENDS
        roscpp
        tf
        nav_msgs
    DEPENDS
        system_lib
)

include_directories( 
  include
    ${catkin_INCLUDE_DIRS}
)

add_library(image_loader src/image_loader.cpp)
target_link_libraries(
  image_loader
  SDL
  SDL_image)

add_executable(mora_map_server src/mora_map_server.cpp)
target_link_libraries(mora_map_server
    image_loader
    yaml-cpp
    ${catkin_LIBRARIES}
)

# copy test data to same place as tests are run
function(copy_test_data)
    cmake_parse_arguments(PROJECT "" "" "FILES" ${ARGN})
    foreach(datafile ${PROJECT_FILES})
        file(COPY ${datafile} DESTINATION ${PROJECT_BINARY_DIR}/test)
    endforeach()
endfunction()

## Tests
if(CATKIN_ENABLE_TESTING)
  copy_test_data( FILES
      test/testmap.bmp
      test/testmap.png )
  catkin_add_gtest(${PROJECT_NAME}_utest test/utest.cpp test/test_constants.cpp)
  target_link_libraries(${PROJECT_NAME}_utest image_loader SDL SDL_image)

  add_executable(rtest test/rtest.cpp test/test_constants.cpp)
  target_link_libraries( rtest
      gtest
      ${catkin_LIBRARIES}
  )
  add_dependencies(
    rtest
    nav_msgs_gencpp
  )

  # This has to be done after we've already built targets, or catkin variables get borked
  find_package(rostest)
  add_rostest(test/rtest.xml)
endif()

## Install executables and/or libraries
install(TARGETS mora_map_server image_loader
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

## Install project namespaced headers
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
  PATTERN ".svn" EXCLUDE)

## Install excutable python script
install( 
    PROGRAMS
      scripts/crop_map
    DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

## Enable C++11 compiler
add_definitions(-std=c++11)
