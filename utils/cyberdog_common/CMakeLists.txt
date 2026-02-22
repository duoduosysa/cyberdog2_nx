cmake_minimum_required(VERSION 3.8)
project(cyberdog_common)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  # 暂时使用-Wno-class-memaccess屏蔽掉rapidjson的编译警告
  # 该警告来自于高等级(>=9)的gcc检查
  add_compile_options(-Wall -Wextra -Wpedantic -Wno-class-memaccess)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rapidjson REQUIRED)
find_package(toml REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_srvs REQUIRED)
find_package(lcm REQUIRED)
find_package(xpack REQUIRED)

set(dependencies
  toml
  rapidjson
  rclcpp
  std_srvs
  lcm
  xpack
)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

add_library(cyberdog_log SHARED
  include/cyberdog_common/cyberdog_log.hpp
  src/cyberdog_log.cpp)

target_include_directories(cyberdog_log PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
target_compile_definitions(cyberdog_log
  PRIVATE "CYBERDOG_LOG_DLL")
ament_target_dependencies(
  cyberdog_log
  ${dependencies})

if(NOT WIN32)
  ament_environment_hooks(
    "${ament_cmake_package_templates_ENVIRONMENT_HOOK_LIBRARY_PATH}")
endif()

install(DIRECTORY include/
  DESTINATION include/)

install(
  TARGETS cyberdog_log
  EXPORT cyberdog_log
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

# Re-export the system's closed-source libcyberdog_fds.so
# The system-installed cyberdog_common (/opt/ros2/cyberdog/) ships cyberdog_fds,
# but this open-source overlay only builds cyberdog_log. Without re-exporting,
# downstream packages (motion_manager, etc.) fail to find cyberdog_common::cyberdog_fds.
#
# CMake does not allow install(TARGETS) on IMPORTED targets, so we:
# 1) Copy the .so into our install/lib via install(FILES)
# 2) Generate a cmake export file that recreates the imported target for downstream packages
# 3) Install the export file to share/cyberdog_common/cmake/
set(_SYSTEM_FDS_LIB "/opt/ros2/cyberdog/lib/libcyberdog_fds.so")
if(EXISTS ${_SYSTEM_FDS_LIB})
  # 1) Copy the .so to our install directory
  install(FILES ${_SYSTEM_FDS_LIB} DESTINATION lib)

  # 2) Generate cmake export file (matches the format ament_export_targets expects)
  file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/cyberdog_fdsExport.cmake" "\
# Re-exported closed-source cyberdog_fds from system install
if(TARGET cyberdog_common::cyberdog_fds)
  return()
endif()

get_filename_component(_IMPORT_PREFIX \"\${CMAKE_CURRENT_LIST_FILE}\" PATH)
get_filename_component(_IMPORT_PREFIX \"\${_IMPORT_PREFIX}\" PATH)
get_filename_component(_IMPORT_PREFIX \"\${_IMPORT_PREFIX}\" PATH)
get_filename_component(_IMPORT_PREFIX \"\${_IMPORT_PREFIX}\" PATH)
if(_IMPORT_PREFIX STREQUAL \"/\")
  set(_IMPORT_PREFIX \"\")
endif()

add_library(cyberdog_common::cyberdog_fds SHARED IMPORTED)
set_target_properties(cyberdog_common::cyberdog_fds PROPERTIES
  INTERFACE_INCLUDE_DIRECTORIES \"\${_IMPORT_PREFIX}/include\;/opt/ros2/cyberdog/include\"
  IMPORTED_LOCATION \"\${_IMPORT_PREFIX}/lib/libcyberdog_fds.so\"
  IMPORTED_SONAME \"libcyberdog_fds.so\"
)
unset(_IMPORT_PREFIX)
")

  # 3) Generate a CONFIG_EXTRAS file that ament_package will auto-include for downstream packages
  file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/cyberdog_fds-extras.cmake"
    "include(\"\${cyberdog_common_DIR}/cyberdog_fdsExport.cmake\")\n")

  # 4) Install the export file to the cmake directory
  install(FILES "${CMAKE_CURRENT_BINARY_DIR}/cyberdog_fdsExport.cmake"
    DESTINATION share/${PROJECT_NAME}/cmake/)
endif()

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

if(LOCAL_BUILD_TESTING)
  find_package(ament_cmake_gtest REQUIRED)
  include_directories(include)
  include_directories(test)
  ament_add_gtest(jsontest test/json_test.cpp src/cyberdog_log.cpp)
  ament_add_gtest(tomltest test/toml_test.cpp src/cyberdog_log.cpp)
  ament_add_gtest(logtest test/log_test.cpp src/cyberdog_log.cpp)
  ament_add_gtest(lcmtest test/lcm_test.cpp src/cyberdog_log.cpp)
  ament_add_gtest(msgqueuetest test/msg_queue_test.cpp src/cyberdog_log.cpp)
  target_link_libraries(lcmtest lcm)
  target_compile_definitions(tomltest PRIVATE BenchmarkPath="${CMAKE_INSTALL_PREFIX}/lib/cyberdog_common")
  target_compile_definitions(jsontest PRIVATE BenchmarkPath="${CMAKE_INSTALL_PREFIX}/lib/cyberdog_common")
  ament_target_dependencies(tomltest rclcpp toml)
  ament_target_dependencies(jsontest rclcpp rapidjson)
  ament_target_dependencies(logtest rclcpp)
  ament_target_dependencies(lcmtest rclcpp xpack)
  ament_target_dependencies(msgqueuetest rclcpp)
  install(TARGETS
    jsontest tomltest logtest lcmtest msgqueuetest
    DESTINATION lib/${PROJECT_NAME})
  install(FILES
    test/benchmark.toml test/benchmark.json test/test_cs.lcm
    DESTINATION lib/${PROJECT_NAME})
  install(DIRECTORY
    test/lcmtestcs test/structor
    DESTINATION lib/${PROJECT_NAME})
endif()
ament_export_include_directories(include)
ament_export_targets(cyberdog_log HAS_LIBRARY_TARGET)
ament_export_dependencies(${dependencies})
if(EXISTS ${_SYSTEM_FDS_LIB})
  ament_package(CONFIG_EXTRAS "${CMAKE_CURRENT_BINARY_DIR}/cyberdog_fds-extras.cmake")
else()
  ament_package()
endif()
