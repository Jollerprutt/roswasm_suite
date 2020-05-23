# roswasm_suite
Libraries for compiling C++ ROS nodes to Webassembly using Emscripten

## Building an Emscripten node

The `roswasm` library uses `catkin` to build an emscripten project.
All you have to do in your package that is using `roswasm` is to add
it as a dependency in your `cmake` file and link against
`${roswasm_LIBRARIES}`. It will automatically set the Emscripten
`em++` compiler as the default for building and linking nodes.
Note that you have to install and source the emscripten SDK before
building your workspace. The following minimal `cmake` file includes
a guard to check that Emscripten is present.
```cmake
cmake_minimum_required(VERSION 2.8.3)
project(talker)

find_package(catkin REQUIRED COMPONENTS roscpp roswasm std_msgs)
catkin_package()

if (DEFINED ENV{EMSDK})

include_directories(
  ${catkin_INCLUDE_DIRS}
)
add_executable(talker src/talker.cpp)
set_target_properties(talker PROPERTIES OUTPUT_NAME "talker.js")
target_link_libraries(talker ${roswasm_LIBRARIES})
configure_file(www/talker.html ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_BIN_DESTINATION}/talker.html COPYONLY)

endif()
```

## Running an Emscripten node

After building your node, you can run it similar to the following command:
```
rosrun roswasm run.py _pkg:=roswasm_tutorials _node:=talker.html _display_port:=8080
```
This will start a webserver and allow you to view the page at `localhost:8080`.
If you want to see the output from the node, open the browser debug console.
