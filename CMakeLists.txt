message(STATUS "${CMAKE_CXX_FLAGS}")
cmake_minimum_required(VERSION 2.8)
# prepare C++11
set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")

# show all warnings
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic -Wextra")


set( ZMQPP_BUILD_EXAMPLES false   CACHE BOOL "Build the ZMQPP examples" )
set( ZMQPP_BUILD_CLIENT   false   CACHE BOOL "Build the ZMQPP client" )

set( ZEROMQ_LIB_DIR       ""      CACHE STRING "The library directory for libzmq" )
set( ZEROMQ_INCLUDE_DIR   ""      CACHE STRING "The include directory for ZMQ" )



find_library(ZEROMQ_LIBRARY zmq HINTS ${ZMQ_LIB_DIR})
find_path(ZEROMQ_INCLUDE zmq.h HINTS ${ZEROMQ_INCLUDE_DIR})



include_directories(${ZEROMQ_INCLUDE})



# libzmqpp
# --------

add_library( zmqpp STATIC

  src/zmqpp/actor.cpp
  src/zmqpp/context.cpp
  src/zmqpp/curve.cpp
  src/zmqpp/frame.cpp
  src/zmqpp/message.cpp
  src/zmqpp/poller.cpp
  src/zmqpp/reactor.cpp
  src/zmqpp/signal.cpp
  src/zmqpp/socket.cpp
  src/zmqpp/z85.cpp
  src/zmqpp/zmqpp.cpp
  )

target_link_libraries( zmqpp ${ZEROMQ_LIBRARIES} )

# Examples
# --------

if(ZMQPP_BUILD_EXAMPLES)
  add_executable( zmqpp-simple-server
    examples/simple_server
    )

  add_executable( zmqpp-simple-client
    examples/simple_client
    )

  target_link_libraries( zmqpp-simple-server zmqpp )
  target_link_libraries( zmqpp-simple-client zmqpp )
endif()

# Client
# ------

if( ZMQPP_BUILD_CLIENT )
  add_executable( zmqpp-client
    src/client/main.cpp
    src/client/options.cpp
    )
  target_link_libraries( zmqpp-client zmqpp )
endif()


# Install
# -------
install(TARGETS zmqpp
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

install(DIRECTORY src/zmqpp DESTINATION include/zmqpp
        FILES_MATCHING PATTERN "*.hpp")
