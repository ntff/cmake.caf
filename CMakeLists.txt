cmake_minimum_required(VERSION 3.4)

project(caf VERSION 0.14.4)

include(ExternalProject)

find_package(sodium REQUIRED)

set(CMAKE_CONFIGURATION_TYPES Debug Release)

foreach(CMAKE_BUILD_TYPE ${CMAKE_CONFIGURATION_TYPES})

    ExternalProject_Add(${CMAKE_BUILD_TYPE}
        PREFIX ${CMAKE_CURRENT_BINARY_DIR}
        SOURCE_DIR ${CMAKE_CURRENT_BINARY_DIR}/src/${PROJECT_NAME}
        URL https://github.com/actor-framework/actor-framework/archive/${PROJECT_VERSION}.tar.gz
        URL_MD5 0479edd43c1f03fdbaeb5c82ad88a193
        INSTALL_DIR ${CMAKE_CURRENT_BINARY_DIR}
        CMAKE_ARGS
            -DCMAKE_INSTALL_PREFIX:path=<INSTALL_DIR>
            -DCMAKE_BUILD_TYPE:string=${CMAKE_BUILD_TYPE}
            -DCMAKE_DEBUG_POSTFIX:string=-d
            -DCAF_BUILD_STATIC_ONLY:bool=true
            -DCAF_NO_UNIT_TESTS:bool=true
            -DCAF_NO_EXAMPLES:bool=true
        BUILD_IN_SOURCE 0
    )

endforeach()

set(INSTALL_DIR ${CMAKE_CURRENT_BINARY_DIR})

string(TOLOWER "${PROJECT_NAME}" PROJECT_NAME_LOWER)

file(GLOB _TARGET_IN_FILES
    LIST_DIRECTORIES false
    RELATIVE "${CMAKE_CURENT_SOURCE_DIR}"
    ${CMAKE_CURRENT_SOURCE_DIR}/target-*.cmake.in
)

foreach(_IN_FILE ${_TARGET_IN_FILES})
    get_filename_component(_IN_FILE_NAME ${_IN_FILE} NAME)
    string(REGEX REPLACE ".in$" "" _OUT_FILE_NAME "${_IN_FILE_NAME}")
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/${_IN_FILE_NAME}
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWER}-${_OUT_FILE_NAME}
        @ONLY
    )
endforeach()

configure_file(config.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWER}-config.cmake"
    @ONLY)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWER}-config-version.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)


install(DIRECTORY
    "${INSTALL_DIR}/include"
    DESTINATION "lib/${CMAKE_LIBRARY_ARCHITECTURE}/${PROJECT_NAME_LOWER}-${PROJECT_VERSION}"
)

install(DIRECTORY
    "${INSTALL_DIR}/lib"
    DESTINATION "lib/${CMAKE_LIBRARY_ARCHITECTURE}/${PROJECT_NAME_LOWER}-${PROJECT_VERSION}"
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWER}-config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWER}-config-version.cmake"
    DESTINATION "lib/${CMAKE_LIBRARY_ARCHITECTURE}/${PROJECT_NAME_LOWER}-${PROJECT_VERSION}"
)

export(PACKAGE ${PROJECT_NAME})

set(CPACK_GENERATOR "TBZ2")
set(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME_LOWER}-${PROJECT_VERSION}-${CMAKE_LIBRARY_ARCHITECTURE}")
include(CPack)
