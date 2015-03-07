# Setup Biicode
include(${CMAKE_HOME_DIRECTORY}/biicode.cmake)
include(biicode/boost/setup)
INIT_BIICODE_BLOCK()

# Remove empty sources if not on Windows
if(NOT WIN32)
    unset(BII_LIB_SRC)
endif()

#add_definitions(-DBOOST_NOWIDE_NO_LIB)

# Add an additional test target that matches test_env; different defines will be set later.
if(WIN32)
    LIST(APPEND BII_BLOCK_EXES test_env_win)
    SET(BII_test_env_win_SRC ${BII_test_env_SRC})
    SET(BII_test_env_win_DEPS ${BII_test_env_DEPS})
    SET(BII_test_env_win_SYSTEM_HEADERS ${BII_test_env_SYSTEM_HEADERS})
endif()

ADD_BIICODE_TARGETS()

# Include Boost and local includes
bii_find_boost()
target_include_directories(${BII_BLOCK_TARGET} INTERFACE ${Boost_INCLUDE_DIRS})

if(MSVC)
    target_compile_options(${BII_BLOCK_TARGET} INTERFACE /EHsc /W3)
endif()

# Configure extra tests
if(WIN32)
    set_target_properties(${BII_BLOCK_TARGETS} PROPERTIES COMPILE_DEFINITIONS NOWIDE_WINDOWS)
    set_target_properties(${BII_test_env_win_TARGET} PROPERTIES COMPILE_DEFINITIONS NOWIDE_TEST_INCLUDE_WINDOWS)
endif()

# Add tests
set(NOWIDE_TESTS ${BII_BLOCK_TARGETS})
list(REMOVE_ITEM NOWIDE_TESTS ${BII_LIB_TARGET} ${BII_test_system_TARGET})
foreach(TEST ${NOWIDE_TESTS})
    add_test(NAME ${TEST} COMMAND ${TEST})
endforeach()

add_test(NAME ${BII_test_system_TARGET}_n COMMAND ${BII_test_system_TARGET} -n)
add_test(NAME ${BII_test_system_TARGET}_w COMMAND ${BII_test_system_TARGET} -w)
