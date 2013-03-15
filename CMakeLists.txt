# the name of your project
project(openssl)
cmake_minimum_required(VERSION 2.8)

include(ExternalProject)

include(cmake/PlatformSettings.cmake)
include(cmake/ConfigureArtifactPath.cmake)
include(cmake/CommonCompilerFlags.cmake)

if (UNIX)

    set(SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
    set(BINARY_DIR ${CMAKE_CURRENT_SOURCE_DIR})

    # Reomve our strict compilation rules
    string(REGEX REPLACE "-Werror" "" OPENSSL_CFLAGS ${CMAKE_C_FLAGS})
    string(REGEX REPLACE "-Wall" "" OPENSSL_CFLAGS ${CMAKE_C_FLAGS})
    string(REGEX REPLACE "-Wextra" "" OPENSSL_CFLAGS ${CMAKE_C_FLAGS})
    set(OPENSSL_LDFLAGS ${CMAKE_MODULE_LINKER_FLAGS})

    if (${PLATFORM} STREQUAL "Android")
        set(OPENSSL_TARGET_PLATFORM android)
        foreach(includeDir ${ANDROID_SYSTEM_INCLUDE_DIRS})
            set(OPENSSL_ANDROID_INCLUDES "${OPENSSL_ANDROID_INCLUDES} -I${includeDir}")
        endforeach()
        set(OPENSSL_CFLAGS "${OPENSSL_CFLAGS} ${OPENSSL_ANDROID_INCLUDES}")

        foreach(libraryDir ${ANDROID_SYSTEM_LIB_DIRS})
            set(OPENSSL_ANDROID_LIB_DIRS "${OPENSSL_ANDROID_LIB_DIRS} -L${libraryDir}")
        endforeach()
        set(OPENSSL_LDFLAGS "${OPENSSL_LDFLAGS} ${OPENSSL_ANDROID_LIB_DIRS}")
    elseif(${PLATFORM} STREQUAL "Linux")
        set(OPENSSL_TARGET_PLATFORM linux-x86_64)
    elseif(${PLATFORM} STREQUAL "iOS")
        set(OPENSSL_TARGET_PLATFORM iphoneos-cross)
    elseif(${PLATFORM} STREQUAL "Mac OS X")
        set(OPENSSL_TARGET_PLATFORM darwin64-x86_64-cc)
    endif()

    configure_file(configure_command.sh.in configure_command.sh)
    configure_file(make_command.sh.in make_command.sh)

    set(CONFIGURE_COMMAND bash ${CMAKE_CURRENT_BINARY_DIR}/configure_command.sh)
    set(BUILD_COMMAND bash ${CMAKE_CURRENT_BINARY_DIR}/make_command.sh)

    message(STATUS "SOURCE_DIR = ${SOURCE_DIR}")
    message(STATUS "BINARY_DIR = ${BINARY_DIR}")
    message(STATUS "CONFIGURE_COMMAND = ${CONFIGURE_COMMAND}")
    message(STATUS "BUILD_COMMAND = ${BUILD_COMMAND}")

    ExternalProject_Add(openssl
        SOURCE_DIR ${SOURCE_DIR}
        BINARY_DIR ${BINARY_DIR}
        CONFIGURE_COMMAND ${CONFIGURE_COMMAND}
        BUILD_COMMAND ${BUILD_COMMAND})

elseif(${PLATFORM} STREQUAL "Visual Studio 2010")

    message(FATAL_ERROR "Fix Visual Studio build")

else()
 
    message(FATAL_ERROR "Generator/platform is not supported")

endif()
