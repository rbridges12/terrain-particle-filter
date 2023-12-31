cmake_minimum_required(VERSION 3.16)
project(terrain_pf VERSION 1.0.0 LANGUAGES CXX)

## Compile as C++17, supported in ROS Kinetic and newer, enable some static analysis
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(PF_CPP_COMPILE_OPTIONS -Wall -Wextra -pedantic)

# Generate compile_commands.json for clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# ROS packages list
set(PF_PACKAGES
        rospy
        roscpp
        std_msgs
        message_generation
        rostest
        sensor_msgs
        )

# Message files
set(PF_MESSAGE_FILES
        AvailableCameras.msg
        Calibrated.msg
        CalibrationStatus.msg
        CameraCmd.msg
        Carousel.msg
        Chassis.msg
        Course.msg
        Diagnostic.msg
        EnableAuton.msg
        GPSWaypoint.msg
        GPSPointList.msg
        HeaterData.msg
        ImuAndMag.msg
        LimitSwitchData.msg
        MastGimbal.msg
        JointLock.msg
        MoteusState.msg
        MotorsStatus.msg
        NavMetadata.msg
        NetworkBandwidth.msg
        ScienceTemperature.msg
        Spectral.msg
        Waypoint.msg
        WaypointType.msg
        GPSPointList.msg
        )

# Generate messages list
set(PF_ROS_MESSAGES
        sensor_msgs
        std_msgs
        sensor_msgs
        )

# Dynamic reconfigure parameter file list
set(PF_PARAMETERS
        config/DetectorParams.cfg
        )

# catkin packages list
set(PF_CATKIN_PACKAGES
        roscpp rospy std_msgs message_runtime
        )

macro(rosify_cpp_target_macro target)
    target_link_libraries(${target} PRIVATE ${catkin_LIBRARIES})
    target_include_directories(${target} SYSTEM PRIVATE ${catkin_INCLUDE_DIRS} "src/util")
    add_dependencies(${target} ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

    target_compile_options(${target} PRIVATE $<$<COMPILE_LANGUAGE:CXX>:${PF_CPP_COMPILE_OPTIONS}>)

    install(TARGETS ${target}
            ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
            LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
            RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
            )
endmacro()

macro(add_cpp_library_macro name sources includes)
    file(GLOB_RECURSE CPP_LIB_SOURCES ${sources})
    add_library(${name} ${CPP_LIB_SOURCES})
    target_include_directories(${name} PUBLIC ${includes})
    rosify_cpp_target_macro(${name})
endmacro()

macro(add_cpp_node_macro name sources)
    file(GLOB_RECURSE CPP_NODE_SOURCES ${sources})
    add_executable(${name} ${CPP_NODE_SOURCES})
    rosify_cpp_target_macro(${name})
endmacro()

macro(add_cpp_nodelet_macro name sources includes)
    # A nodelet runs inside another process so it is a library
    add_cpp_library_macro(${name} ${sources} ${includes})
endmacro()

macro(add_gazebo_plugin_macro name sources includes)
    add_cpp_library_macro(${name} ${sources} ${includes})

    # TODO: find a proper variable name that points to /opt/ros/noetic/lib
    target_link_directories(${name} PRIVATE ${GAZEBO_LIBRARY_DIRS} /opt/ros/noetic/lib)
    target_link_libraries(${name} PRIVATE ${GAZEBO_LIBRARIES} ${Boost_LIBRARIES})
    target_include_directories(${name} SYSTEM PRIVATE ${GAZEBO_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS})
endmacro()

# launch install macro
macro(install_launch_macro)
    install(DIRECTORY launch/
            DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
            )
endmacro()

macro(add_tests_macro)
    # Add C++ unit tests
    catkin_add_gtest(example-cpp-test test/example/cpp_test.cpp)

    # Python unit tests
    catkin_add_nosetests(test/navigation/drive_test.py)
    catkin_add_nosetests(test/teleop/teleop_test.py)
    catkin_add_nosetests(test/util/SE3_test.py)
    catkin_add_nosetests(test/util/SO3_test.py)

    # Integration tests (python and c++)
    find_package(rostest REQUIRED)
    add_rostest(test/example/basic_integration_test.test)
    add_rostest(test/integration/integration.test)
    add_rostest(test/util/SE3_tf_test.test)
endmacro()

# Subdirectories before message declarations
set(CMAKE_SUBDIRS "")


# -=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-
# Specify build details by appending to lists
# and implementing the some extra macros
# Add new devices as elseif blocks
# Select device with --cmake-flags -D DEVICE=<fill in device>
# -=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-

    # Add any laptop specific packages here
    list(APPEND PF_PACKAGES
            tf2_geometry_msgs
            tf2_ros
            tf2
            tf2_web_republisher
            visualization_msgs
            image_transport
            cv_bridge
            rosbridge_server
            teleop_twist_joy
            gazebo_ros
            moveit_commander
            moveit_core
            moveit_fake_controller_manager
            moveit_kinematics
            moveit_msgs
            moveit_planners_chomp
            moveit_planners_ompl
            moveit_ros_benchmarks
            moveit_ros_control_interface
            moveit_ros_manipulation
            moveit_ros_move_group
            moveit_ros_occupancy_map_monitor
            moveit_ros_perception
            moveit_ros_planning
            moveit_ros_planning_interface
            moveit_ros_robot_interaction
            moveit_ros_visualization
            moveit_ros_warehouse
            moveit_setup_assistant
            moveit_simple_controller_manager
            )

    # -=-=-=-=-
    # Macros
    # -=-=-=-=-

    # These packages need to be found individually
    macro(add_packages_macro)
        find_package(OpenCV REQUIRED COMPONENTS core aruco)
        find_package(gazebo REQUIRED)
        find_package(Eigen3 REQUIRED)
        find_package(manif REQUIRED)
        find_package(PCL REQUIRED)
        include_directories(${PCL_INCLUDE_DIRS})
        link_directories(${PCL_LIBRARY_DIRS})
        add_definitions(${PCL_DEFINITIONS})
    endmacro()

    # define an add and link macro
    # Put items here to build
    macro(add_and_link_macro)
        # add_cpp_nodelet_macro(tag_detector_nodelet "src/perception/tag_detector/*.cpp" "src/perception/tag_detector")
        # target_link_libraries(tag_detector_nodelet PRIVATE ${OpenCV_LIBRARIES} lie)

        # add_cpp_node_macro(brushed_motors "src/esw/brushed_motors/*")

        # add_cpp_library_macro(lie "src/util/lie/*.cpp" "src/util/lie")

        add_cpp_library_macro(filters "src/localization/filters/*.cpp" "src/localization/filters")
        add_cpp_node_macro(terrain_particle_filter "src/localization/*")
        target_include_directories(terrain_particle_filter SYSTEM PRIVATE ${manif_INCLUDE_DIRS})
        target_link_libraries(terrain_particle_filter PRIVATE filters ${PCL_LIBRARIES})

        add_gazebo_plugin_macro(differential_drive_plugin_6w "src/gazebo/differential_drive_6w.cpp" "src")

        add_gazebo_plugin_macro(kinect_plugin "src/gazebo/gazebo_ros_openni_kinect.cpp" "src/gazebo")
        target_link_libraries(kinect_plugin PRIVATE gazebo_ros_camera_utils DepthCameraPlugin Eigen3::Eigen)
        set_target_properties(kinect_plugin PROPERTIES CXX_CLANG_TIDY "")

        # if (ZED_FOUND)
        #     #            add_cpp_node_macro(zed_node "src/perception/zed_wrapper/*.c*")
        #     #            target_link_options(zed_node PRIVATE "LINKER:--copy-dt-needed-entries")
        #     #            target_include_directories(zed_node SYSTEM PRIVATE ${ZED_INCLUDE_DIRS} ${CUDA_INCLUDE_DIRS})
        #     #            target_link_libraries(zed_node PRIVATE ${ZED_LIBRARIES} ${OpenCV_LIBRARIES} ${SPECIAL_OS_LIBS} lie tag_detector_nodelet)

        #     add_cpp_nodelet_macro(zed_nodelet "src/perception/zed_wrapper/*.c*" , "src/perception/zed_wrapper")
        #     target_include_directories(zed_nodelet SYSTEM PRIVATE ${ZED_INCLUDE_DIRS} ${CUDA_INCLUDE_DIRS})
        #     target_link_libraries(zed_nodelet PRIVATE ${ZED_LIBRARIES} ${OpenCV_LIBRARIES} ${SPECIAL_OS_LIBS} lie)
        #     target_compile_definitions(zed_nodelet PRIVATE PF_IS_NODELET)
        #     target_compile_options(zed_nodelet PRIVATE $<$<COMPILE_LANGUAGE:CUDA>:--expt-extended-lambda>)
        # endif ()
    endmacro()


# 3. Find Packages
find_package(
        catkin REQUIRED COMPONENTS
        ${PF_PACKAGES}
)

if (COMMAND add_packages_macro)
    add_packages_macro()
endif ()


# 4. Python module support
catkin_python_setup()


# 4.5. Subdirectories before message declarations
if (NOT "${CMAKE_SUBDIRS}" MATCHES "")
    add_subdirectory(
            ${CMAKE_SUBDIRS}
    )
endif ()


# 5. Message Generators (add_xxx)
add_message_files(
        FILES
        ${PF_MESSAGE_FILES}
)

# 6. Invoke messages (generate_messages)
generate_messages(
        DEPENDENCIES
        ${PF_ROS_MESSAGES}
)

# 7. Specify package build info export (catkin_package)
catkin_package(
        CATKIN_DEPENDS
        ${PF_CATKIN_PACKAGES}
)


if (COMMAND add_and_link_macro)
    add_and_link_macro()
endif ()


# 9. Tests to build
if (COMMAND add_tests_macro)
    add_tests_macro()
endif ()


# 10. Install rules
install_launch_macro()
if (COMMAND additional_install_macro)
    additional_install_macro()
endif ()
