cmake_minimum_required(VERSION 2.8.3)
project(msf_core)

# Set the build type.  Options are:
#  Coverage       : w/ debug symbols, w/o optimization, w/ code-coverage
#  Debug          : w/ debug symbols, w/o optimization
#  Release        : w/o debug symbols, w/ optimization
#  RelWithDebInfo : w/ debug symbols, w/ optimization
#  MinSizeRel     : w/o debug symbols, w/ optimization, stripped binaries
set(CMAKE_BUILD_TYPE Release)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake/)

find_package(catkin REQUIRED COMPONENTS sensor_msgs dynamic_reconfigure msf_timing tf)
find_package(Eigen REQUIRED)
find_package(GLog REQUIRED)

find_package(Doxygen)
set(MSF_DOCUMENTATION msf_documentation)
if(DOXYGEN_FOUND)
  add_custom_target(${MSF_DOCUMENTATION}
                    COMMAND doxygen msf_dox.dox
                    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/../)
else()
  add_custom_target(${MSF_DOCUMENTATION})
endif(DOXYGEN_FOUND)

include_directories(include ${Eigen_INCLUDE_DIRS} ${catkin_INCLUDE_DIRS})

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wno-unused-parameter")
if (CMAKE_COMPILER_IS_GNUCC)
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif()

# Enable timing the core
add_definitions (-DENABLE_MSF_TIMING)

generate_dynamic_reconfigure_options(cfg/MSF_Core.cfg)

catkin_package(
    DEPENDS eigen GLog
    CATKIN_DEPENDS roscpp sensor_msgs dynamic_reconfigure msf_timing tf
    INCLUDE_DIRS include ${Eigen_INCLUDE_DIRS}
    LIBRARIES msf_core similaritytransform ${GLOG_LIBRARY}
    CFG_EXTRAS export_flags.cmake
)

install(DIRECTORY cmake
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})

add_library(${PROJECT_NAME} src/lib/msf_tools.cc src/lib/falsecolor.cc 
                     src/lib/gps_conversion.cc)
target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES})
add_dependencies(${PROJECT_NAME} ${PROJECT_NAME}_gencfg ${MSF_DOCUMENTATION}) 

add_library(similaritytransform src/similaritytransform.cc)
target_link_libraries(similaritytransform ${catkin_LIBRARIES})

catkin_add_gtest(test_similaritytransform src/test/test_similaritytransform.cc)
target_link_libraries(test_similaritytransform similaritytransform ${GLOG_LIBRARY})

catkin_add_gtest(test_static_statelist src/test/test_staticstatelist.cc)
target_link_libraries(test_static_statelist pthread ${PROJECT_NAME} ${GLOG_LIBRARY})

