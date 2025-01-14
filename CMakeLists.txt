cmake_minimum_required(VERSION 3.16.3)
project(openvr_headset_ros)

## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  message_generation
  geometry_msgs
  nodelet
  roscpp
  rospy
  roslib
  std_msgs
  std_srvs
  tf
  visualization_msgs
  nodelet
  rosconsole
  cv_bridge
  roslaunch
  sensor_msgs
  image_transport
  rviz
)
##  openvr_ros


find_package(gazebo REQUIRED)


link_directories(${catkin_LIBRARY_DIRS}) # ...typo before as catkin_LIBRARY_RIRS ??

## System dependencies are found with CMake's conventions
# find_package(Boost REQUIRED COMPONENTS system)


## Uncomment this if the package has a setup.py. This macro ensures
## modules and global scripts declared therein get installed
## See http://ros.org/doc/api/catkin/html/user_guide/setup_dot_py.html
# catkin_python_setup()

################################################
## Declare ROS messages, services and actions ##
################################################

## To declare and build messages, services or actions from within this
## package, follow these steps:
## * Let MSG_DEP_SET be the set of packages whose message types you use in
##   your messages/services/actions (e.g. std_msgs, actionlib_msgs, ...).
## * In the file package.xml:
##   * add a build_depend tag for "message_generation"
##   * add a build_depend and a run_depend tag for each package in MSG_DEP_SET
##   * If MSG_DEP_SET isn't empty the following dependency has been pulled in
##     but can be declared for certainty nonetheless:
##     * add a run_depend tag for "message_runtime"
## * In this file (CMakeLists.txt):
##   * add "message_generation" and every package in MSG_DEP_SET to
##     find_package(catkin REQUIRED COMPONENTS ...)
##   * add "message_runtime" and every package in MSG_DEP_SET to
##     catkin_package(CATKIN_DEPENDS ...)
##   * uncomment the add_*_files sections below as needed
##     and list every .msg/.srv/.action file to be processed
##   * uncomment the generate_messages entry below
##   * add every package in MSG_DEP_SET to generate_messages(DEPENDENCIES ...)
find_library(OPENVR_LIBRARY openvr_api HINTS /usr/local/lib/)
## Generate messages in the 'msg' folder
add_message_files(DIRECTORY msg
  FILES
  Vive.msg
  Vive_controller.msg
  Vive_buttons.msg
  Vive_trackpad.msg
  TrackedDeviceClass.msg
  TrackedDevicePose.msg
  TrackedDeviceResult.msg
  TrackedDeviceRole.msg
  TrackedDeviceHeader.msg
 )

## Generate services in the 'srv' folder
# add_service_files(
#   FILES
#   Service1.srv
#   Service2.srv
# )

## Generate actions in the 'action' folder
# add_action_files(
#   FILES
#   Action1.action
#   Action2.action
# )

## Generate added messages and services with any dependencies listed here
 generate_messages(
   DEPENDENCIES
   openvr_headset_ros # is this required??
   geometry_msgs
   std_msgs
 )

################################################
## Declare ROS dynamic reconfigure parameters ##
################################################

## To declare and build dynamic reconfigure parameters within this
## package, follow these steps:
## * In the file package.xml:
##   * add a build_depend and a run_depend tag for "dynamic_reconfigure"
## * In this file (CMakeLists.txt):
##   * add "dynamic_reconfigure" to
##     find_package(catkin REQUIRED COMPONENTS ...)
##   * uncomment the "generate_dynamic_reconfigure_options" section below
##     and list every .cfg file to be processed

## Generate dynamic reconfigure parameters in the 'cfg' folder
# generate_dynamic_reconfigure_options(
#   cfg/DynReconf1.cfg
#   cfg/DynReconf2.cfg
# )

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if your package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
#catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES vrui_mdf
#  CATKIN_DEPENDS geometry_msgs nodelet roscpp roslib std_msgs std_srvs tf
#  DEPENDS system_lib
#)


catkin_package(
# CATKIN_DEPENDS message_generation message_runtime roscpp std_msgs geometry_msgs # is message_generation req.d?
 CATKIN_DEPENDS message_runtime roscpp std_msgs geometry_msgs
 INCLUDE_DIRS include 
 LIBRARIES ${PROJECT_NAME}
)


###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations

##set(OpenVR "~/openvr") #This might be wrong
##message("${OpenVR}")
include_directories(
  ${PROJECT_NAME}
  include
##  "${OpenVR}/headers"
  ${OPENGL_INCLUDE_DIRS}
  ${GLEW_INCLUDE_DIRS}
  ${GLUT_INCLUDE_DIRS}
  ${SDL2_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${OPENVR_INCLUDE_DIR}
)

##link_directories("$${OPENVR}/lib/linux64")
find_package(SDL2 REQUIRED)
find_package(OpenGL REQUIRED)
find_package(OpenCV REQUIRED)
find_package(Boost REQUIRED COMPONENTS thread)
find_package(GLEW REQUIRED)
find_package(GLUT REQUIRED)
##find_package(OpenVR REQUIRED)
#find_package(catkin REQUIRED COMPONENTS thread)
#find_package(Qt<QTVERSION> COMPONENTS Core)
find_package(Qt5 COMPONENTS Core)

## Declare a C++ library
#add_library(${PROJECT_NAME}
  #src/${PROJECT_NAME}/vrui_mdf.cpp
#)

set(CMAKE_LINKER_FLAGS "-pthread" CACHE STRING "Linker Flags" FORCE)
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_LINKER_FLAGS}" CACHE STRING "" FORCE)
set(CMAKE_MODULE_LINKER_FLAGS "${CMAKE_LINKER_FLAGS}" CACHE STRING "" FORCE)
set(CMAKE_EXE_LINKER_FLAGS    "${CMAKE_LINKER_FLAGS}" CACHE STRING "" FORCE)

set(CMAKE_AUTOMOC ON)




add_executable(opentracking
  src/opentracking.cpp
)
add_dependencies(opentracking ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_link_libraries (opentracking LINK_PUBLIC
	${catkin_LIBRARIES}
	${Boost_LIBRARIES}
	${OPENVR_LIBRARY}
	${CMAKE_DL_LIBS}
)



add_executable(imagesub src/imagesub.cpp)
add_dependencies(imagesub ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_include_directories(imagesub PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
                           $<INSTALL_INTERFACE:include>)
target_link_libraries (imagesub OpenGL::GL ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${SDL2_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENVR_LIBRARY} ${CMAKE_DL_LIBS})

add_executable(statictest src/statictest.cpp)
add_dependencies(statictest ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_include_directories(statictest PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
                           $<INSTALL_INTERFACE:include>)
target_link_libraries (statictest OpenGL::GL ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${SDL2_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENVR_LIBRARY} ${CMAKE_DL_LIBS})

add_executable(controllers src/controllers.cpp)
add_dependencies(controllers ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_include_directories(controllers PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
                           $<INSTALL_INTERFACE:include>)
target_link_libraries (controllers OpenGL::GL ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${SDL2_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENVR_LIBRARY} ${CMAKE_DL_LIBS})

add_executable(controllers_hector src/controllers_hector.cpp)
add_dependencies(controllers_hector ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_include_directories(statictest PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
                           $<INSTALL_INTERFACE:include>)
target_link_libraries (controllers_hector OpenGL::GL ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${SDL2_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENVR_LIBRARY} ${CMAKE_DL_LIBS})

##-------------------LeftRight--------------------

add_executable(controllers_turtlebot_left src/controllers_turtlebot_left.cpp)
add_dependencies(controllers_turtlebot_left ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_include_directories(statictest PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
                           $<INSTALL_INTERFACE:include>)
target_link_libraries (controllers_turtlebot_left OpenGL::GL ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${SDL2_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENVR_LIBRARY} ${CMAKE_DL_LIBS})

add_executable(controllers_turtlebot_right src/controllers_turtlebot_right.cpp)
add_dependencies(controllers_turtlebot_right ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_include_directories(statictest PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
                           $<INSTALL_INTERFACE:include>)
target_link_libraries (controllers_turtlebot_right OpenGL::GL ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${SDL2_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENVR_LIBRARY} ${CMAKE_DL_LIBS})

##-------------------SwitchTest--------------------

add_executable(imagesub_switchtest src/switchtest/imagesub_switchtest.cpp)
add_dependencies(imagesub_switchtest ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_include_directories(imagesub_switchtest PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
                           $<INSTALL_INTERFACE:include>)
target_link_libraries (imagesub_switchtest OpenGL::GL ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${SDL2_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENVR_LIBRARY} ${CMAKE_DL_LIBS})

add_executable(opentracking_switchtest src/switchtest/opentracking_switchtest.cpp)
add_dependencies(opentracking_switchtest ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_link_libraries (opentracking_switchtest LINK_PUBLIC
	${catkin_LIBRARIES}
	${Boost_LIBRARIES}
	${OPENVR_LIBRARY}
	${CMAKE_DL_LIBS}
)

add_executable(controllers_turtlebot_1 src/switchtest/controllers_turtlebot_1.cpp)
add_dependencies(controllers_turtlebot_1 ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_include_directories(controllers_turtlebot_1 PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
                           $<INSTALL_INTERFACE:include>)
target_link_libraries (controllers_turtlebot_1 OpenGL::GL ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${SDL2_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENVR_LIBRARY} ${CMAKE_DL_LIBS})

add_executable(controllers_turtlebot_0 src/switchtest/controllers_turtlebot_0.cpp)
add_dependencies(controllers_turtlebot_0 ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_include_directories(controllers_turtlebot_0 PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
                           $<INSTALL_INTERFACE:include>)
target_link_libraries (controllers_turtlebot_0 OpenGL::GL ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${SDL2_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENVR_LIBRARY} ${CMAKE_DL_LIBS})

##-------------------ThreeRoverTest--------------------

add_executable(imagesub_threerover src/threerovertest/imagesub_threerover.cpp)
add_dependencies(imagesub_threerover ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_include_directories(imagesub_threerover PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
                           $<INSTALL_INTERFACE:include>)
target_link_libraries (imagesub_threerover OpenGL::GL ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${SDL2_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENVR_LIBRARY} ${CMAKE_DL_LIBS})

add_executable(opentracking_threerover src/threerovertest/opentracking_threerover.cpp)
add_dependencies(opentracking_threerover ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_link_libraries (opentracking_threerover LINK_PUBLIC
	${catkin_LIBRARIES}
	${Boost_LIBRARIES}
	${OPENVR_LIBRARY}
	${CMAKE_DL_LIBS}
)

add_executable(controllers_turtlebot_surv src/threerovertest/controllers_turtlebot_surv.cpp)
add_dependencies(controllers_turtlebot_surv ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_include_directories(controllers_turtlebot_surv PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
                           $<INSTALL_INTERFACE:include>)
target_link_libraries (controllers_turtlebot_surv OpenGL::GL ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${SDL2_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENVR_LIBRARY} ${CMAKE_DL_LIBS})

add_executable(controllers_turtlebot_tool src/threerovertest/controllers_turtlebot_tool.cpp)
add_dependencies(controllers_turtlebot_tool ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_include_directories(controllers_turtlebot_tool PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
                           $<INSTALL_INTERFACE:include>)
target_link_libraries (controllers_turtlebot_tool OpenGL::GL ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${SDL2_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENVR_LIBRARY} ${CMAKE_DL_LIBS})

add_executable(controllers_turtlebot_samp src/threerovertest/controllers_turtlebot_samp.cpp)
add_dependencies(controllers_turtlebot_samp ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_include_directories(controllers_turtlebot_samp PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
                           $<INSTALL_INTERFACE:include>)
target_link_libraries (controllers_turtlebot_samp OpenGL::GL ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GLEW_LIBRARIES} ${GLUT_LIBRARIES} ${SDL2_LIBRARIES} ${OpenCV_LIBRARIES} ${OPENVR_LIBRARY} ${CMAKE_DL_LIBS})

#target_link_libraries(${Boost_LIBRARIES})

## Add cmake target dependencies of the library
## as an example, code may need to be generated before libraries
## either from message generation or dynamic reconfigure
##add_dependencies(${openvr_ros} ${catkin_EXPORTED_TARGETS})
# see: https://answers.ros.org/question/188982/message-headers-wont-build-first/

##add_dependencies(opentracking vrui_mdf_generate_messages_cpp)

##add_dependencies(newcodes vrui_mdf_generate_messages_cpp)
## Declare a C++ executable
## With catkin_make all packages are built within a single CMake context
## The recommended prefix ensures that target names across packages don't collide
# add_executable(${PROJECT_NAME}_node src/vrui_mdf_node.cpp)

## Rename C++ executable without prefix
## The above recommended prefix causes long target names, the following renames the
## target back to the shorter version for ease of user use
## e.g. "rosrun someones_pkg node" instead of "rosrun someones_pkg someones_pkg_node"
# set_target_properties(${PROJECT_NAME}_node PROPERTIES OUTPUT_NAME node PREFIX "")

## Add cmake target dependencies of the executable
## same as for the library above
# add_dependencies(${PROJECT_NAME}_node ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

## Specify libraries to link a library or executable target against
# target_link_libraries(${PROJECT_NAME}_node
#   ${catkin_LIBRARIES}
# )

#############
## Install ##
#############

# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
# install(PROGRAMS
#   scripts/my_python_script
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark executables and/or libraries for installation
# install(TARGETS ${PROJECT_NAME} ${PROJECT_NAME}_node
#   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark cpp header files for installation
# install(DIRECTORY include/${PROJECT_NAME}/
#   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
#   FILES_MATCHING PATTERN "*.h"
#   PATTERN ".svn" EXCLUDE
# )

## Mark other files for installation (e.g. launch and bag files, etc.)
# install(FILES
#   # myfile1
#   # myfile2
#   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
# )

install(DIRECTORY 
launch
models DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})


#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
# catkin_add_gtest(${PROJECT_NAME}-test test/test_vrui_mdf.cpp)
# if(TARGET ${PROJECT_NAME}-test)
#   target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME})
# endif()

## Add folders to be run by python nosetests
# catkin_add_nosetests(test)

