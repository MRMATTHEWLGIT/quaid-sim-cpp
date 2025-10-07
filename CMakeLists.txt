cmake_minimum_required(VERSION 3.25)
project(quaid_sim_cpp)

set(CMAKE_CXX_STANDARD 20)

# --- Includes ---
include_directories(${CMAKE_SOURCE_DIR}/include)

# --- MuJoCo ---
set(MUJOCO_PATH "C:/Users/Mathew/Desktop/Quaid_Sim/mujoco-3.3.0")
include_directories(${MUJOCO_PATH}/include)
link_directories(${MUJOCO_PATH}/lib)

# --- Other dependencies (vcpkg) ---
find_package(glfw3 CONFIG REQUIRED)
find_package(yaml-cpp CONFIG REQUIRED)

# Generate a fix file for missing EMPTY_BIN symbol
file(WRITE ${CMAKE_BINARY_DIR}/paho_fix.cpp
"#include <mqtt/message.h>
namespace mqtt {
    const std::string message::EMPTY_BIN;
}
")

add_executable(quaid_sim_cpp
    src/main.cpp
    src/quaid_controller.cpp
    src/mqtt_controller.cpp
    src/servo_shield.cpp
    src/utils.cpp
    src/settings.cpp
    ${CMAKE_BINARY_DIR}/paho_fix.cpp
)

target_include_directories(quaid_sim_cpp PRIVATE
    ${CMAKE_SOURCE_DIR}/include
    ${VCPKG_INSTALLED_DIR}/x64-windows/include
)

# --- Manually ensure correct link order ---
link_directories(${VCPKG_INSTALLED_DIR}/x64-windows/lib)

target_link_libraries(quaid_sim_cpp
    # MuJoCo
    "${MUJOCO_PATH}/lib/mujoco.lib"

    # vcpkg-managed libs
    "${VCPKG_INSTALLED_DIR}/x64-windows/lib/glfw3dll.lib"
    "${VCPKG_INSTALLED_DIR}/x64-windows/lib/yaml-cpp.lib"

    # MQTT: link only the C++ wrapper and its async backend
    "${VCPKG_INSTALLED_DIR}/x64-windows/lib/paho-mqttpp3.lib"
    "${VCPKG_INSTALLED_DIR}/x64-windows/lib/paho-mqtt3as.lib"
)

# Compatibility define
add_compile_definitions(uint=unsigned\ int)
target_compile_definitions(quaid_sim_cpp PRIVATE YAML_CPP_STATIC_DEFINE)


