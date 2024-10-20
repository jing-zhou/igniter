cmake_minimum_required(VERSION 3.6)
project(igniter)

include(CMakePrintHelpers)
add_definitions(-DENABLE_ANDROID_LOG)
# As long as we use OpenSSL 1.1.1, we are safe
add_definitions(-DENABLE_TLS13_CIPHERSUITES)
find_library( # Sets the name of the path variable.
          androidLogLib

          # Specifies the name of the NDK library that
          # you want CMake to locate.
          log)

include(FetchContent)
function(CloneRepository repositoryURL branchName projectName sourceDir)
    #Commands are left empty so that we only checkout the source and no not perform any kind of build
    message("Starting to clone ${projectName} into ${sourceDir}")
    FetchContent_Declare(
            "${projectName}"
            GIT_REPOSITORY "${repositoryURL}"
            SOURCE_DIR "${sourceDir}"
            GIT_TAG "origin/${branchName}"
            CONFIGURE_COMMAND ""
            BUILD_COMMAND ""
            INSTALL_COMMAND ""
    )
    FetchContent_MakeAvailable(${projectName})
endfunction(CloneRepository)
CloneRepository("https://github.com/jing-zhou/trojan.git" "igniter" "trojan" "${CMAKE_CURRENT_LIST_DIR}/trojan")

add_library(igniter
    trojan/src/core/authenticator.cpp
    trojan/src/core/config.cpp
    trojan/src/core/log.cpp
    trojan/src/core/service.cpp
    trojan/src/core/version.cpp
    trojan/src/proto/socks5address.cpp
    trojan/src/proto/trojanrequest.cpp
    trojan/src/proto/udppacket.cpp
    trojan/src/session/clientsession.cpp
    trojan/src/session/forwardsession.cpp
    trojan/src/session/natsession.cpp
    trojan/src/session/serversession.cpp
    trojan/src/session/session.cpp
    trojan/src/session/udpforwardsession.cpp
    trojan/src/ssl/ssldefaults.cpp
    trojan/src/ssl/sslsession.cpp)

find_package(Boost REQUIRED COMPONENTS system program_options)
include_directories(${Boost_INCLUDE_DIR})

target_include_directories(igniter PRIVATE
        ${CMAKE_CURRENT_LIST_DIR}/libs/include
        ${CMAKE_CURRENT_LIST_DIR}/trojan/src)
target_link_libraries(igniter
        ${CMAKE_CURRENT_LIST_DIR}/libs/lib/${ANDROID_ABI}/libboost_program_options.a
        ${CMAKE_CURRENT_LIST_DIR}/libs/lib/${ANDROID_ABI}/libboost_system.a
        ${CMAKE_CURRENT_LIST_DIR}/libs/lib/libssl.a
        ${CMAKE_CURRENT_LIST_DIR}/libs/lib/libcrypto.a
        ${Boost_LIBRARIES}
        ${androidLogLib})
add_library(jni-helper SHARED jni-helper.cpp)
target_include_directories(jni-helper PRIVATE
    ${CMAKE_CURRENT_LIST_DIR}/trojan/src
    ${CMAKE_CURRENT_LIST_DIR}/libn2t/src
    ${CMAKE_CURRENT_LIST_DIR}/libs/include)
target_link_libraries(jni-helper igniter)
