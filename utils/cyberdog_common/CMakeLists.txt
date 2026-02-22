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
set(_SYSTEM_FDS_LIB "/opt/ros2/cyberdog/lib/libcyberdog_fds.so")
if(EXISTS ${_SYSTEM_FDS_LIB})
  add_library(cyberdog_fds SHARED IMPORTED)
  set_target_properties(cyberdog_fds PROPERTIES
    IMPORTED_LOCATION ${_SYSTEM_FDS_LIB}
    IMPORTED_SONAME "libcyberdog_fds.so"
    INTERFACE_INCLUDE_DIRECTORIES "/opt/ros2/cyberdog/include;${CMAKE_CURRENT_SOURCE_DIR}/include"
    INTERFACE_LINK_LIBRARIES "rclcpp::rclcpp;std_srvs::std_srvs__rosidl_generator_c;std_srvs::std_srvs__rosidl_typesupport_introspection_c;std_srvs::std_srvs__rosidl_typesupport_c;std_srvs::std_srvs__rosidl_generator_cpp;std_srvs::std_srvs__rosidl_typesupport_introspection_cpp;std_srvs::std_srvs__rosidl_typesupport_cpp"
  )
  install(
    TARGETS cyberdog_fds
    EXPORT cyberdog_fds
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
  )
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
if(EXISTS ${_SYSTEM_FDS_LIB})
  ament_export_targets(cyberdog_log cyberdog_fds HAS_LIBRARY_TARGET)
else()
  ament_export_targets(cyberdog_log HAS_LIBRARY_TARGET)
endif()
ament_export_dependencies(${dependencies})
ament_package()
