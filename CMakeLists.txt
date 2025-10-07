cmake_minimum_required(VERSION 2.8...4.0)

set(PROJECT_NAME VerUp LANGUAGES CXX)

Project(${PROJECT_NAME})

if(${CMAKE_BUILD_TYPE} MATCHES "Debug")
    add_definitions(-D_DEBUG)
elseif(${CMAKE_BUILD_TYPE} MATCHES "Release")
    add_definitions(-D_RELEASE) 
endif()

file(
  DOWNLOAD
  https://github.com/cpm-cmake/CPM.cmake/releases/download/v0.40.8/CPM.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake
  EXPECTED_HASH SHA256=78ba32abdf798bc616bab7c73aac32a17bbd7b06ad9e26a6add69de8f3ae4791
)
include(${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake)

set(AddBoost.cmake_VERSION 3.7.3)
CPMAddPackage(
  NAME AddBoost.cmake
  VERSION "${AddBoost.cmake_VERSION}"
  URL "https://github.com/Arniiiii/AddBoost.cmake/archive/refs/tags/${AddBoost.cmake_VERSION}.tar.gz"
)

set(PROCESSOR_ARHITECTURE_TYPE "UNKNOWN")

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(PROCESSOR_ARHITECTURE_TYPE "x64")
elseif(CMAKE_SIZEOF_VOID_P EQUAL 4)
    set(PROCESSOR_ARHITECTURE_TYPE "x32")
else()
    string(TOLOWER "${CMAKE_SYSTEM_PROCESSOR}" LOWER_CASE_PROC)

    if(${LOWER_CASE_PROC} STREQUAL "arm" OR ${LOWER_CASE_PROC} STREQUAL "aarch64")
        set(PROCESSOR_ARHITECTURE_TYPE "ARM")
    elseif(${LOWER_CASE_PROC} STREQUAL "powerpc" OR ${LOWER_CASE_PROC} STREQUAL "ppc64le")
        set(PROCESSOR_ARHITECTURE_TYPE "PowerPC")
    elseif(${LOWER_CASE_PROC} STREQUAL "mips" OR ${LOWER_CASE_PROC} STREQUAL "mipsel")
        set(PROCESSOR_ARHITECTURE_TYPE "MIPS")
    elseif(${LOWER_CASE_PROC} STREQUAL "riscv64")
        set(PROCESSOR_ARHITECTURE_TYPE "RISC-V 64-bit")
    else()
        message(ERROR "Unrecognized processor type: ${CMAKE_SYSTEM_PROCESSOR}. Setting PROCESSOR_ARHITECTURE_TYPE to Unknown.")
    endif()
endif()

message(STATUS "Building for arhitecture: ${PROCESSOR_ARHITECTURE_TYPE}, build type: ${CMAKE_BUILD_TYPE}")

macro(add_os_specific_libraries TARGET_NAME)
    if(WIN32 OR WINDOWS)
        target_link_libraries(${TARGET_NAME} PUBLIC ws2_32 Mswsock)
    elseif(APPLE)
        target_link_libraries(${TARGET_NAME} PUBLIC CoreFoundation)
    elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
        target_link_libraries(${TARGET_NAME} PUBLIC pthread rt)
    else()
        message(FATAL_ERROR "Unsupported platform detected!")
    endif()
endmacro()

set(HEADER_FILES  

)

set(SOURCE_FILES 
    ${CMAKE_SOURCE_DIR}/src/main.cpp
)

add_executable(${PROJECT_NAME} ${HEADER_FILES} ${SOURCE_FILES})
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/include)
target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_17)
add_os_specific_libraries(${PROJECT_NAME})

#target properties
set_target_properties(${PROJECT_NAME}
    PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/${PROCESSOR_ARHITECTURE_TYPE}/lib"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/${PROCESSOR_ARHITECTURE_TYPE}/lib"
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/${PROCESSOR_ARHITECTURE_TYPE}/bin"
    PDB_OUTPUT_DIRECTORY     "${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}/${PROCESSOR_ARHITECTURE_TYPE}/bin"
)

#boost
set(TRY_BOOST_VERSION "1.88.0.beta1")
set(BOOST_MY_OPTIONS "BOOST_ENABLE_PYTHON ON;")
set(BOOST_NOT_HEADER_ONLY_COMPONENTS "thread;system;log")
set(BOOST_HEADER_ONLY_COMPONENTS "asio;beast;uuid;container;cobalt;signals2")

add_boost(
  TRY_BOOST_VERSION BOOST_HEADER_ONLY_COMPONENTS
  BOOST_NOT_HEADER_ONLY_COMPONENTS ${PROJECT_NAME}
)
#end(boost)

#rapidjson
    #target_link_libraries(${PROJECT_NAME} RapidJSON::RapidJSON)
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/vendor/rapidjson/include)
#end(rapidjson)