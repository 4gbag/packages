cmake_minimum_required(VERSION 2.6)

PROJECT(nginx-util CXX)
SET(CMAKE_CXX_STANDARD 17)

INCLUDE(CheckFunctionExists)

IF(UBUS)
FIND_PATH(ubus_include_dir libubus.h)
FIND_LIBRARY(ubox NAMES ubox)
FIND_LIBRARY(ubus NAMES ubus)
INCLUDE_DIRECTORIES(${ubus_include_dir})
ENDIF()

ADD_DEFINITIONS(-Os -Wall -Werror -Wextra -g3)
ADD_DEFINITIONS(-Wno-unused-parameter -Wmissing-declarations)

SET(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS "")


IF(UBUS)

ADD_COMPILE_DEFINITIONS(VERSION=${VERSION})

ADD_EXECUTABLE(nginx-util nginx-util.cpp)
TARGET_COMPILE_DEFINITIONS(nginx-util PUBLIC -DNO_SSL)
TARGET_LINK_LIBRARIES(nginx-util ${ubox} ${ubus} pthread)
INSTALL(TARGETS nginx-util RUNTIME DESTINATION bin)

ADD_EXECUTABLE(nginx-ssl-util nginx-util.cpp)
TARGET_LINK_LIBRARIES(nginx-ssl-util ${ubox} ${ubus} pthread ssl crypto pcre)
INSTALL(TARGETS nginx-ssl-util RUNTIME DESTINATION bin)

ADD_EXECUTABLE(nginx-ssl-util-nopcre nginx-util.cpp)
TARGET_COMPILE_DEFINITIONS(nginx-ssl-util-nopcre PUBLIC -DNO_PCRE)
TARGET_LINK_LIBRARIES(nginx-ssl-util-nopcre ${ubox} ${ubus} pthread ssl crypto)
INSTALL(TARGETS nginx-ssl-util-nopcre RUNTIME DESTINATION bin)

ELSE()

CONFIGURE_FILE(test-px5g.sh test-px5g.sh COPYONLY)
CONFIGURE_FILE(test-nginx-util.sh test-nginx-util.sh COPYONLY)
CONFIGURE_FILE(test-nginx-util-root.sh test-nginx-util-root.sh COPYONLY)

ADD_EXECUTABLE(px5g px5g.cpp)
TARGET_LINK_LIBRARIES(px5g ssl crypto)
INSTALL(TARGETS px5g RUNTIME DESTINATION bin)

ADD_EXECUTABLE(nginx-ssl-util-noubus nginx-util.cpp)
TARGET_COMPILE_DEFINITIONS(nginx-ssl-util-noubus PUBLIC -DNO_UBUS)
TARGET_LINK_LIBRARIES(nginx-ssl-util-noubus pthread ssl crypto pcre)
INSTALL(TARGETS nginx-ssl-util-noubus RUNTIME DESTINATION bin)

ADD_EXECUTABLE(nginx-ssl-util-nopcre-noubus nginx-util.cpp)
TARGET_COMPILE_DEFINITIONS(nginx-ssl-util-nopcre-noubus PUBLIC -DNO_PCRE -DNO_UBUS)
TARGET_LINK_LIBRARIES(nginx-ssl-util-nopcre-noubus pthread ssl crypto)
INSTALL(TARGETS nginx-ssl-util-nopcre-noubus RUNTIME DESTINATION bin)

ENDIF()
