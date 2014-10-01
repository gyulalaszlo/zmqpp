message(STATUS "${CMAKE_CXX_FLAGS}")
cmake_minimum_required(VERSION 2.8)
enable_testing()

# prepare C++11
set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")

# show all warnings
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic -Wextra")


# Set a consistent MACOSX_RPATH default across all CMake versions.
# When CMake 2.8.12 is required, change this default to 1.
# When CMake 3.0.0 is required, remove this block (see CMP0042).
if(NOT DEFINED CMAKE_MACOSX_RPATH)
  set(CMAKE_MACOSX_RPATH 0)
endif()


set( ZMQPP_BUILD_EXAMPLES false   CACHE BOOL "Build the ZMQPP examples" )
set( ZMQPP_BUILD_CLIENT   false   CACHE BOOL "Build the ZMQPP client" )
set( ZMQPP_BUILD_TESTS    false   CACHE BOOL "Build the ZMQPP tests" )

set( ZEROMQ_LIB_DIR       ""      CACHE PATH "The library directory for libzmq" )
set( ZEROMQ_INCLUDE_DIR   ""      CACHE PATH "The include directory for ZMQ" )
set( IS_TRAVIS_CI_BUILD   true    CACHE bool "Defines TRAVIS_CI_BUILD - Should the tests avoid running cases where memory is scarce." )

find_library(ZEROMQ_LIBRARY_STATIC zmq-static PATHS ${ZEROMQ_LIB_DIR})
find_library(ZEROMQ_LIBRARY_SHARED zmq-static PATHS ${ZEROMQ_LIB_DIR})
find_path(ZEROMQ_INCLUDE zmq.h PATHS ${ZEROMQ_INCLUDE_DIR})

include_directories(${ZEROMQ_INCLUDE})

# Do not run some tests when building on travis-ci (this cause oom error and kill the test
# process)
# These tests seem to be:
#    - sending_large_messages_string
if (IS_TRAVIS_CI_BUILD)
  add_definitions( -DTRAVIS_CI_BUILD)
endif()



# libzmqpp
# --------

set( LIBZMQPP_SOURCES
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

add_library( zmqpp_objects OBJECT ${LIBZMQPP_SOURCES})

add_library( zmqpp SHARED $<TARGET_OBJECTS:zmqpp_objects>)
add_library( zmqpp-static STATIC $<TARGET_OBJECTS:zmqpp_objects>)

target_link_libraries( zmqpp ${ZEROMQ_LIBRARY_SHARED} )
target_link_libraries( zmqpp-static ${ZEROMQ_LIBRARY_STATIC} )

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


# Find boost if necessary
# -----------------------

if (ZMQPP_BUILD_TESTS OR ZMQPP_BUILD_CLIENT )
  set(BOOST_COMPONENTS_USED )

  # Client needs program options
  if (ZMQPP_BUILD_CLIENT)
    list(APPEND BOOST_COMPONENTS_USED program_options)
  endif()

  # Tests need boost test
  if (ZMQPP_BUILD_TESTS)
    list(APPEND BOOST_COMPONENTS_USED thread system unit_test_framework)
  endif()

  # ---------------------
  set(Boost_USE_STATIC_LIBS        OFF) # only find static libs
  set(Boost_USE_MULTITHREADED      ON)
  set(Boost_USE_STATIC_RUNTIME    OFF)

  find_package(Boost
    REQUIRED
    COMPONENTS ${BOOST_COMPONENTS_USED}
    )

  include_directories( ${Boost_INCLUDE_DIRS} )
endif()


# Client
# ------

if( ZMQPP_BUILD_CLIENT )
  add_executable( zmqpp-client
    src/client/main.cpp
    src/client/options.cpp
    )
  target_link_libraries( zmqpp-client zmqpp ${Boost_LIBRARIES} )
endif()

# Tests
# -----

if( ZMQPP_BUILD_TESTS )
  # Boost
  # -----

  add_executable( zmqpp-test-runner
    src/tests/test_actor.cpp
    src/tests/test_context.cpp
    src/tests/test_inet.cpp
    src/tests/test_load.cpp
    src/tests/test_message.cpp
    src/tests/test_message_stream.cpp
    src/tests/test_poller.cpp
    src/tests/test_reactor.cpp
    src/tests/test_sanity.cpp
    src/tests/test_socket.cpp
    src/tests/test_socket_options.cpp
    src/tests/test_z85.cpp
    )
  target_link_libraries( zmqpp-test-runner zmqpp ${Boost_LIBRARIES})
  add_test( zmqpp-test zmqpp-test-runner --log-level=test-suite )
endif()


# Install
# -------
install(TARGETS zmqpp zmqpp-static
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

install(DIRECTORY src/zmqpp DESTINATION include/
        FILES_MATCHING PATTERN "*.hpp")
