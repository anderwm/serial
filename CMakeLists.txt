cmake_minimum_required(VERSION 2.8.3)
project(serial)

if(UNIX AND NOT APPLE)
    # If Linux, add rt and pthread
    set(PROJ_LIBS rt pthread)
else()
    set(PROJ_LIBS "")
endif()

option(BUILD_WITH_CATKIN "Build using catkin." ON)

# Find catkin
if (BUILD_WITH_CATKIN)
    find_package(catkin REQUIRED)
    catkin_package(
        LIBRARIES ${PROJECT_NAME}
        INCLUDE_DIRS include
        DEPENDS ${PROJ_LIBS}
    )
else()
    SET(CATKIN_PACKAGE_LIB_DESTINATION "${CMAKE_INSTALL_PREFIX}/lib")
    SET(CATKIN_PACKAGE_BIN_DESTINATION "${CMAKE_INSTALL_PREFIX}/bin")
    SET(CATKIN_GLOBAL_INCLUDE_DESTINATION "${CMAKE_INSTALL_PREFIX}/include")
    SET(CATKIN_ENABLE_TESTING OFF)
endif()

## Sources
set(serial_SRCS
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)
if(UNIX)
    # If unix
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
    # If windows
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif()

## Add serial library
add_library(${PROJECT_NAME} ${serial_SRCS})
if(UNIX AND NOT APPLE)
    target_link_libraries(${PROJECT_NAME} rt)
endif()

## Uncomment for example
add_executable(serial_example examples/serial_example.cc)
add_dependencies(serial_example ${PROJECT_NAME})
target_link_libraries(serial_example ${PROJECT_NAME})

## Include headers
include_directories(include)

## Install executable
install(TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)

## Install headers
install(FILES include/serial/serial.h include/serial/v8stdint.h
  DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}/serial)

## Tests
if(CATKIN_ENABLE_TESTING)
    catkin_add_gtest(${PROJECT_NAME}-test tests/serial_tests.cc)
    target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME} ${Boost_LIBRARIES})
    if(UNIX AND NOT APPLE)
        target_link_libraries(${PROJECT_NAME}-test util)
    endif()
endif()
