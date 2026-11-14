if(POLICY CMP0135)
  cmake_policy(SET CMP0135 NEW)
endif()

include(FetchContent)
FetchContent_Declare(
  googletest
  URL https://github.com/google/googletest/archive/refs/tags/v1.14.0.zip
)
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

add_executable(unit_tests
  smoke_test.cpp
  resp_parser_test.cpp
)
target_link_libraries(unit_tests PRIVATE miniredis_core GTest::gtest_main)

include(GoogleTest)
gtest_discover_tests(unit_tests)