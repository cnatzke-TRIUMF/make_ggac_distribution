#------------------------------------------------------------------
#Setup the project
#------------------------------------------------------------------
cmake_minimum_required(VERSION 3.2)

project(GenerateDistribution VERSION 0.1)

# configure header file
#configure_file(GenerateDistribution.h.in GenerateDistribution.h)

#Locate the ROOT package and defines a number of variables (e.g. ROOT_INCLUDE_DIRS)
find_package(ROOT CONFIG REQUIRED)
include(${ROOT_USE_FILE})

set(CMAKE_C_COMPILER "gcc")
set(CMAKE_CXX_COMPILER "g++")

set(CMAKE_CXX_FLAGS "-Wall -O3 ${CMAKE_CXX_FLAGS} ${GRSI_CONFIG}")

# pick up c++ standard from ROOT install
string(APPEND CMAKE_CXX_FLAGS "${ROOT_CXX_FLAGS} -pedantic -Wno-long-long -g")

# Connect ROOT to project
list(APPEND CMAKE_PREFIX_PATH $ENV{ROOTSYS})

# Connect GRSISORT headers
set(GRSI_INCLUDE_DIRS $ENV{GRSISYS}/include $ENV{GRSISYS}/GRSIData/include)

# Grsisort libraries for linking
execute_process(COMMAND grsi-config --all-libs --GRSIData-libs --root
   OUTPUT_VARIABLE GRSI_CONFIG
   OUTPUT_STRIP_TRAILING_WHITESPACE
)

# Add fitting libraries
set(GRSI_CONFIG "${GRSI_CONFIG} -lTPeakFitting ${GRSI_CONFIG}")
#set(GRSI_CONFIG "${GRSI_CONFIG} ${GRSI_CONFIG_2} -lTPeakFitting")

# Message for debugging
message(STATUS "Found Grsisort libraries: ${GRSI_CONFIG}")

# Adding src files
file(GLOB SOURCES ${PROJECT_SOURCE_DIR}/src/*.cpp)

# Naming main executable
add_executable(GenerateDistribution ${SOURCES})

#add_custom_command(TARGET GenerateDistribution PRE_BUILD
#    COMMAND ${CMAKE_COMMAND} -E copy_directory
#${PROJECT_SOURCE_DIR}/misc/ $<TARGET_FILE_DIR:GenerateDistribution>)

# linking libraries
target_link_libraries(GenerateDistribution PUBLIC
   ${GRSI_CONFIG}
)

# add the binary tree to the search path for include files so that we will find AnalysisTreeConverter.h
target_include_directories(GenerateDistribution PUBLIC
    "${PROJECT_BINARY_DIR}"
    "${GRSI_INCLUDE_DIRS}"
    include
    )

# add install targets
install(TARGETS GenerateDistribution DESTINATION "${PROJECT_BINARY_DIR}/bin")
install(FILES "${PROJECT_BINARY_DIR}/GenerateDistribution.h"
   DESTINATION "${PROJECT_BINARY_DIR}/include"
)
