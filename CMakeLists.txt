cmake_minimum_required(VERSION 3.10)

# set the project name
project(SIPp)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

include_directories(${PROJECT_SOURCE_DIR}/include)

option(BUILD_STATIC "Build a statically-linked binary" OFF)
option(USE_SSL "Build with SIPS support" OFF)
option(USE_SCTP "Build with SCTP support" OFF)
option(USE_PCAP "Build with PCAP playback support" OFF)
option(USE_GSL "Build with improved statistical support" ON)
option(USE_EPOLL "Build with epoll support" ON)

file(GLOB all_SRCS
  "${PROJECT_SOURCE_DIR}/src/*.cpp"
  "${PROJECT_SOURCE_DIR}/src/*.c"
  )

list(REMOVE_ITEM all_SRCS
  "${PROJECT_SOURCE_DIR}/src/sipp_unittest.cpp")

if(NOT USE_SSL)
  list(REMOVE_ITEM all_SRCS
    "${PROJECT_SOURCE_DIR}/src/sslsocket.cpp")
endif(NOT USE_SSL)

if(NOT USE_PCAP)
  list(REMOVE_ITEM all_SRCS
    "${PROJECT_SOURCE_DIR}/src/prepare_pcap.c")
  list(REMOVE_ITEM all_SRCS
    "${PROJECT_SOURCE_DIR}/src/send_packets.c")
endif(NOT USE_PCAP)

if(USE_PCAP)
  add_definitions("-DPCAPPLAY")
endif(USE_PCAP)

if(USE_GSL)
  add_definitions("-DHAVE_GSL")
endif(USE_GSL)

add_definitions("-DRTP_STREAM -DHAVE_EPOLL")

# add the executable
add_executable(sipp ${all_SRCS})
if(BUILD_STATIC)
  set(CMAKE_EXE_LINKER_FLAGS "-static-libgcc -static-libstdc++ -static")
endif(BUILD_STATIC)
target_link_libraries(sipp dl ncursesw pthread)
if(USE_PCAP)
  target_link_libraries(sipp pcap)
endif(USE_PCAP)
