cmake_minimum_required(VERSION 3.2)

set(APP_NAME helloworld)
if (DEFINED ENV{TRAVIS_BUILD_NUMBER})
    project(${APP_NAME} VERSION 0.0.$ENV{TRAVIS_BUILD_NUMBER})
else()
    project(${APP_NAME} VERSION 0.0.1)
endif()

find_package(GTest REQUIRED)

configure_file(version.h.in version.h)

add_executable(helloworld main.cpp)
add_library(helloworld_lib lib.cpp)
add_executable(test_version test_version.cpp)

set_target_properties(helloworld helloworld_lib test_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)
target_include_directories(helloworld_lib
    PRIVATE "${CMAKE_BINARY_DIR}"
)
set_target_properties(test_version PROPERTIES
    INCLUDE_DIRECTORIES ${GTEST_INCLUDE_DIRS}
)

target_link_libraries(helloworld PRIVATE
    helloworld_lib
)
target_link_libraries(test_version
    ${GTEST_BOTH_LIBRARIES}
    pthread
    helloworld_lib
)

if (MSVC)
    target_compile_options(helloworld PRIVATE
        /W4
    )
    target_compile_options(helloworld_lib PRIVATE
        /W4
    )
    target_compile_options(test_version PRIVATE
        /W4
    )
else ()
    target_compile_options(helloworld PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(helloworld_lib PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(test_version PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
endif()



install(TARGETS helloworld RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)

set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")

set(CPACK_PACKAGE_CONTACT scfocus@yandex.ru)

include(CPack)

enable_testing()

add_test(test_version test_version)
