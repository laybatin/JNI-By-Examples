cmake_minimum_required(VERSION 2.8)
project(jnitests)
set(CMAKE_BUILD_TYPE Release)

set (_java_libarch amd64)		#input arch
file(TO_CMAKE_PATH "$ENV{JAVA_HOME}" _JAVA_HOME)	#get env JAVA_HOME from shell
    set(_JDK_DIRS "${_JAVA_HOME}/jre/lib/${_java_libarch}/*"
                  "${_JAVA_HOME}/jre/lib/${_java_libarch}"
                  "${_JAVA_HOME}/jre/lib/*"
                  "${_JAVA_HOME}/jre/lib"
                  "${_JAVA_HOME}/lib/*"
                  "${_JAVA_HOME}/lib"
                  "${_JAVA_HOME}/include/*"
                  "${_JAVA_HOME}/include"
                  "${_JAVA_HOME}"
    )
    message("ARCHI ${_java_libarch}")
    find_path(JAVA_INCLUDE_PATH
        NAMES jni.h
        PATHS ${_JDK_DIRS}
        NO_DEFAULT_PATH)
    #In IBM java, it's jniport.h instead of jni_md.h
    find_path(JAVA_INCLUDE_PATH2
        NAMES jni_md.h jniport.h
        PATHS ${_JDK_DIRS}
        NO_DEFAULT_PATH)
    SET(JNI_INCLUDE_DIRS ${JAVA_INCLUDE_PATH} ${JAVA_INCLUDE_PATH2})
    FIND_LIBRARY(JAVA_JVM_LIBRARY
        NAMES jvm JavaVM
        PATHS ${_JDK_DIRS}
        NO_DEFAULT_PATH)
    SET(JNI_LIBRARIES ${JAVA_JVM_LIBRARY})
    message("JAVA_HOME=${_JAVA_HOME}, JAVA_JVM_LIBRARY=${JAVA_JVM_LIBRARY}")
    message("JAVA_INCLUDE_PATH=${JAVA_INCLUDE_PATH}, JAVA_INCLUDE_PATH2=${JAVA_INCLUDE_PATH2}")
    IF(JAVA_JVM_LIBRARY AND JAVA_INCLUDE_PATH AND JAVA_INCLUDE_PATH2)
        message("Located all JNI components successfully.")
    ELSE()
        message(FATAL_ERROR "Failed to find a viable JVM installation under JAVA_HOME.")
    ENDIF()

#find_package(JNI REQUIRED)

find_library(EXAMPLE_LIBRARY
    NAMES example
    PATHS
    /usr/local/lib
)

message("EXAMPLE LIB ${EXAMPLE_LIBRARY}")
include_directories(${JNI_INCLUDE_DIRS})
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
set(SOURCE_FILES Utilities.cpp)
add_library(jnitests SHARED ${SOURCE_FILES})
target_link_libraries(jnitests PRIVATE ${EXAMPLE_LIBRARY})
