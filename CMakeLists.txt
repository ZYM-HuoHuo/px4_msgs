cmake_minimum_required(VERSION 3.5)

project(px4_msgs)

list(INSERT CMAKE_MODULE_PATH 0 "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
	add_compile_options(-Wall -Wextra)
endif()

find_package(ament_cmake REQUIRED)
find_package(builtin_interfaces REQUIRED)
find_package(rosidl_default_generators REQUIRED)

# ##############################################################################
# Generate ROS messages, ROS2 interfaces and IDL files #
# ##############################################################################

# 只生成项目实际使用的消息：msg/ 目录下有 200+ 条消息，全量生成会显著拖慢编译。
# 需要新的 PX4 消息时，把消息名追加到下面列表即可。
set(PX4_MSG_NAMES
	OffboardControlMode
	TrajectorySetpoint
	VehicleCommand
	VehicleOdometry
	SensorCombined
	VehicleLocalPosition
	BatteryStatus
	VehicleStatus
	VehicleAttitude
	InputRc
	# srv/VehicleCommand.srv 的 reply 类型，缺了会导致代码生成失败
	VehicleCommandAck
)

set(PX4_MSGS "")
foreach(msg_name ${PX4_MSG_NAMES})
	set(msg_file "msg/${msg_name}.msg")
	if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/${msg_file}")
		message(FATAL_ERROR "px4_msgs: 未找到 ${msg_file}")
	endif()
	list(APPEND PX4_MSGS "${msg_file}")
endforeach()

# get all srv files
set(SRVS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/srv")
file(GLOB PX4_SRVS RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" "${SRVS_DIR}/*.srv")

# Generate introspection typesupport for C and C++ and IDL files
rosidl_generate_interfaces(${PROJECT_NAME}
	${PX4_MSGS}
	${PX4_SRVS}
	DEPENDENCIES builtin_interfaces
	ADD_LINTER_TESTS
)

ament_export_dependencies(rosidl_default_runtime)

ament_package()
