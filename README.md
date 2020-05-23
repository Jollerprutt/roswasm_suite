# roswasm_suite
![](https://github.com/nilsbore/roswasm_suite/workflows/CI/badge.svg)

Libraries for compiling C++ ROS nodes to Webassembly using Emscripten. Allows you to write c++ ROS nodes similar to how you would otherwise, and to then have them run in a webbrowser and communicate with ROS through `rosbridge_websocket`.

## Dependencies

* [Emscripten](https://emscripten.org/docs/getting_started/downloads.html) tested with version `1.39.10` but latest should do
* [rosbridge_suite](https://github.com/RobotWebTools/rosbridge_suite) after [the commit adding cbor-raw compression](https://github.com/RobotWebTools/rosbridge_suite/commit/dc7fcb282d1326d573abe83579cc7d989ae71739), latest develop should do

## Writing a roswasm node

The `roswasm` client library presents an API similar to `roscpp`, with the
main differences being that most interfaces are heap allocated, and that Emscripten
manages the event loop. Below is a shortened version of the corresponding
`listener` example implementation:

```cpp
#include <emscripten.h>
#include <roswasm/roswasm.h>
#include <std_msgs/String.h>

roswasm::NodeHandle* n;
roswasm::Subscriber* sub;

void chatterCallback(const std_msgs::String& msg)
{
    printf("I heard: [%s]\n", msg.data.c_str());
}

void loop() {}

extern "C" int main(int argc, char** argv)
{
    n = new roswasm::NodeHandle();
    sub = n->subscribe<std_msgs::String>("chatter", chatterCallback);
    emscripten_set_main_loop(loop, 10, 1);
    return 0;
}

```


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
project(listener)

find_package(catkin REQUIRED COMPONENTS roscpp roswasm std_msgs)
catkin_package()

if (DEFINED ENV{EMSDK})

include_directories(
  ${catkin_INCLUDE_DIRS}
)
add_executable(listener src/listener.cpp)
set_target_properties(listener PROPERTIES OUTPUT_NAME "listener.js")
target_link_libraries(listener ${roswasm_LIBRARIES})
configure_file(www/listener.html ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_BIN_DESTINATION}/listener.html COPYONLY)

endif()
```

## Running an Emscripten node

After building your node, you can run it similar to the following command:
```
rosrun roswasm run.py _pkg:=roswasm_tutorials _node:=listener.html _display_port:=8080
```
This will start a webserver and allow you to view the page at `localhost:8080`.
If you want to see the output from the node, open the browser debug console.
