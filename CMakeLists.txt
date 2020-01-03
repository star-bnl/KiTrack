########################################################
# cmake file for building Marlin example Package
# @author Jan Engels, Desy IT
CMAKE_MINIMUM_REQUIRED(VERSION 2.6 FATAL_ERROR)
########################################################


# project name
PROJECT( KiTrack )


# project version
SET( ${PROJECT_NAME}_VERSION_MAJOR 1 )
SET( ${PROJECT_NAME}_VERSION_MINOR 10 )
SET( ${PROJECT_NAME}_VERSION_PATCH 0 )

cmake_policy(SET CMP0008 NEW)

# By default build shared libraries but allow the user to change if desired
OPTION(BUILD_SHARED_LIBS "Global flag to cause add_library to create shared libraries if on" ON)

SET(CMAKE_CXX_STANDARD 11)

FIND_PACKAGE( ROOT REQUIRED ) 
INCLUDE_DIRECTORIES( ${ROOT_INCLUDE_DIRS} )
LINK_LIBRARIES( ${ROOT_LIBRARIES} )
ADD_DEFINITIONS( ${ROOT_DEFINITIONS} )




### DOCUMENTATION ###########################################################

OPTION( INSTALL_DOC "Set to OFF to skip build/install Documentation" OFF )

IF( INSTALL_DOC AND EXISTS "${PROJECT_SOURCE_DIR}/doc/CMakeLists.txt" )
    ADD_SUBDIRECTORY( ./doc )
ENDIF()



### LIBRARY #################################################################

# definitions to pass to the compiler
#ADD_DEFINITIONS( "-Wall -ansi -pedantic" )
#ADD_DEFINITIONS( "-Wno-long-long" )

# include directories
INCLUDE_DIRECTORIES( BEFORE
   ./include
   )
INSTALL( DIRECTORY ./include DESTINATION . FILES_MATCHING PATTERN "*.h" )

# add library
AUX_SOURCE_DIRECTORY( ./src/marlin library_sources )
AUX_SOURCE_DIRECTORY( ./src/Criteria library_sources )
AUX_SOURCE_DIRECTORY( ./src/KiTrack library_sources )  








ADD_LIBRARY( ${PROJECT_NAME} ${library_sources} )
INSTALL( TARGETS ${PROJECT_NAME} DESTINATION lib )




### TESTING #################################################################

MACRO( ADD_UNIT_TEST _name )
    IF( BUILD_TESTING )
        ADD_EXECUTABLE( test_${_name} ${ARGN} )
    ELSE()
        ADD_EXECUTABLE( test_${_name} EXCLUDE_FROM_ALL ${ARGN} )
    ENDIF()        
    TARGET_LINK_LIBRARIES( test_${_name} ${PROJECT_NAME} )
    GET_TARGET_PROPERTY( _executable test_${_name} LOCATION )
    ADD_TEST( t_${_name} ${_executable} )
ENDMACRO()

INCLUDE_DIRECTORIES( ${ILCTEST_INCLUDE_DIRS} )



# ADD_UNIT_TEST( simple_circle ./src/testing/test_simple_circle.cc )
# SET_TESTS_PROPERTIES( t_simple_circle PROPERTIES FAIL_REGULAR_EXPRESSION "TEST_FAILED" )
# SET_TESTS_PROPERTIES( t_simple_circle PROPERTIES PASS_REGULAR_EXPRESSION "TEST_PASSED" )
# SET_TESTS_PROPERTIES( t_simple_circle PROPERTIES WILL_FAIL TRUE )





CONFIGURE_FILE(
    "${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
    "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake" @ONLY
)
INSTALL( FILES "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake" DESTINATION . )

CONFIGURE_FILE(
    "${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}ConfigVersion.cmake.in"
    "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake" @ONLY
)
INSTALL( FILES "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake" DESTINATION . )
