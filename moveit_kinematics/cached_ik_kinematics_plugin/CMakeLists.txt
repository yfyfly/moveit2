find_package(trac_ik_kinematics_plugin QUIET)
find_package(ur_kinematics QUIET)

set(MOVEIT_LIB_NAME moveit_cached_ik_kinematics_base)
add_library(${MOVEIT_LIB_NAME} SHARED src/ik_cache.cpp)
set_target_properties(${MOVEIT_LIB_NAME} PROPERTIES VERSION "${${PROJECT_NAME}_VERSION}")
ament_target_dependencies(${MOVEIT_LIB_NAME}
  rclcpp
  moveit_core
  moveit_msgs
)

if(trac_ik_kinematics_plugin_FOUND)
    include_directories(${trac_ik_kinematics_plugin_INCLUDE_DIRS})
endif()

set(MOVEIT_LIB_NAME moveit_cached_ik_kinematics_plugin)
add_library(${MOVEIT_LIB_NAME} SHARED src/cached_ik_kinematics_plugin.cpp)
set_target_properties(${MOVEIT_LIB_NAME} PROPERTIES VERSION "${${PROJECT_NAME}_VERSION}")
ament_target_dependencies(${MOVEIT_LIB_NAME}
  rclcpp
  moveit_core
  moveit_msgs
)
target_link_libraries(${MOVEIT_LIB_NAME}
    moveit_cached_ik_kinematics_base
    moveit_kdl_kinematics_plugin
    moveit_srv_kinematics_plugin)
if(trac_ik_kinematics_plugin_FOUND)
    target_link_libraries(${MOVEIT_LIB_NAME} ${trac_ik_kinematics_plugin_LIBRARIES})
    set_target_properties(${MOVEIT_LIB_NAME} PROPERTIES COMPILE_DEFINITIONS "CACHED_IK_KINEMATICS_TRAC_IK")
endif()

# This is just for testing purposes; the arms from Universal Robots have
# analytic solvers, so caching just adds extra overhead.
if(ur_kinematics_FOUND)
    include_directories(${ur_kinematics_INCLUDE_DIRS})
    foreach(UR 3 5 10)
        set(MOVEIT_LIB_NAME moveit_cached_ur${UR}_kinematics_plugin)
        add_library(${MOVEIT_LIB_NAME} SHARED src/cached_ur_kinematics_plugin.cpp)
        set_target_properties(${MOVEIT_LIB_NAME} PROPERTIES VERSION "${${PROJECT_NAME}_VERSION}")
        find_library(ur${UR}_pluginlib ur${UR}_moveit_plugin PATHS ${ur_kinematics_LIBRARY_DIRS})
        target_link_libraries(${MOVEIT_LIB_NAME}
            moveit_cached_ik_kinematics_base
            ${ur${UR}_pluginlib})
        install(TARGETS ${MOVEIT_LIB_NAME}
                EXPORT ${MOVEIT_LIB_NAME}
                LIBRARY DESTINATION lib
                ARCHIVE DESTINATION lib
                RUNTIME DESTINATION bin)
    endforeach()
endif()

install(DIRECTORY include/ DESTINATION include)
install(DIRECTORY launch DESTINATION share/${PROJECT_NAME})
