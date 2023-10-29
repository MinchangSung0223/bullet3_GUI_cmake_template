cmake_minimum_required(VERSION 3.0)
PROJECT(test_bullet)

set(BULLET_PHYSICS_SOURCE_DIR  "/opt/bullet3")
set(BULLET_PHYSICS_LIB  "/opt/bullet3/build_cmake/local_install/lib")


INCLUDE_DIRECTORIES( 
		${BULLET_PHYSICS_SOURCE_DIR}/src
		${BULLET_PHYSICS_SOURCE_DIR}/examples
		${BULLET_PHYSICS_SOURCE_DIR}/examples/SharedMemory
		${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator		
		${BULLET_PHYSICS_SOURCE_DIR}/examples/ThirdPartyLibs
		${BULLET_PHYSICS_SOURCE_DIR}/examples/ThirdPartyLibs/enet/include
		${BULLET_PHYSICS_SOURCE_DIR}/examples/ThirdPartyLibs/clsocket/src
  )


SET(RobotSimulator_SRCS
	src/test.cpp
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/b3RobotSimulatorClientAPI.cpp
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/b3RobotSimulatorClientAPI.h	
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/MinitaurSetup.cpp
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/MinitaurSetup.h
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/../../examples/ExampleBrowser/InProcessExampleBrowser.cpp
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/../SharedMemory/GraphicsServerExample.cpp
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/../SharedMemory/GraphicsClientExample.cpp
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/../SharedMemory/RemoteGUIHelper.cpp
#	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/../SharedMemory/RemoteGUIHelperTCP.cpp
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/../SharedMemory/GraphicsServerExample.h
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/../SharedMemory/GraphicsClientExample.h
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/../SharedMemory/RemoteGUIHelper.h
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/../SharedMemory/GraphicsSharedMemoryCommands.h
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/../SharedMemory/GraphicsSharedMemoryPublic.h
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/../../examples/SharedMemory/PhysicsServerExample.cpp
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/../../examples/SharedMemory/PhysicsServerExampleBullet2.cpp
	${BULLET_PHYSICS_SOURCE_DIR}/examples/RobotSimulator/../../examples/SharedMemory/SharedMemoryInProcessPhysicsC_API.cpp		
)	



IF(BUILD_CLSOCKET)
 ADD_DEFINITIONS(-DBT_ENABLE_CLSOCKET)
 ADD_DEFINITIONS(${OSDEF})
ENDIF(BUILD_CLSOCKET)


#some code to support OpenGL and Glew cross platform
IF (WIN32)
	INCLUDE_DIRECTORIES(
			${BULLET_PHYSICS_SOURCE_DIR}/btgui/OpenGLWindow/GlewWindows
	)
	ADD_DEFINITIONS(-DGLEW_STATIC)
	LINK_LIBRARIES( ${OPENGL_gl_LIBRARY} ${OPENGL_glu_LIBRARY} )
ELSE(WIN32)
	IF(APPLE)
		find_library(COCOA NAMES Cocoa)
		MESSAGE(${COCOA})
		link_libraries(${COCOA} ${OPENGL_gl_LIBRARY} ${OPENGL_glu_LIBRARY})

	ELSE(APPLE)
		INCLUDE_DIRECTORIES(
				${BULLET_PHYSICS_SOURCE_DIR}/btgui/OpenGLWindow/GlewWindows
		)
		ADD_DEFINITIONS("-DGLEW_INIT_OPENGL11_FUNCTIONS=1")
		ADD_DEFINITIONS("-DGLEW_STATIC")
		ADD_DEFINITIONS("-DGLEW_DYNAMIC_LOAD_ALL_GLX_FUNCTIONS=1")

		FIND_PACKAGE(Threads)
		LINK_LIBRARIES( ${CMAKE_THREAD_LIBS_INIT} ${DL} )
	ENDIF(APPLE)
ENDIF(WIN32)


ADD_EXECUTABLE(App_RobotSimulator  ${RobotSimulator_SRCS})

#SET_TARGET_PROPERTIES(App_RobotSimulator PROPERTIES VERSION ${BULLET_VERSION})
#SET_TARGET_PROPERTIES(App_RobotSimulator PROPERTIES DEBUG_POSTFIX "_d")
#SET_TARGET_PROPERTIES(App_RobotSimulator PROPERTIES COMPILE_DEFINITIONS "B3_USE_ROBOTSIM_GUI")


TARGET_LINK_LIBRARIES(App_RobotSimulator BulletRobotics BulletExampleBrowserLib BulletFileLoader BulletWorldImporter BulletSoftBody BulletDynamics BulletCollision BulletInverseDynamicsUtils BulletInverseDynamics LinearMath OpenGLWindow gwen Bullet3Common)
target_compile_definitions(App_RobotSimulator PUBLIC -DBT_USE_DOUBLE_PRECISION)

IF(WIN32)
	IF(BUILD_ENET OR BUILD_CLSOCKET)
		TARGET_LINK_LIBRARIES(App_RobotSimulator ws2_32 Winmm)
	ENDIF(BUILD_ENET OR BUILD_CLSOCKET)
ENDIF(WIN32)
