include(MacroQbtCompilerSettings)
qbt_set_compiler_options()

include(QbtTargetSources)

find_package(Boost ${requiredBoostVersion} REQUIRED)
find_package(LibtorrentRasterbar ${requiredLibtorrentVersion} REQUIRED)
find_package(OpenSSL ${requiredOpensslVersion} REQUIRED)

if (Boost_VERSION VERSION_LESS 106000)
    add_definitions(-DBOOST_NO_CXX11_RVALUE_REFERENCES)
endif()

find_package(Qt5 ${requiredQtVersion} REQUIRED COMPONENTS Core Network Xml LinguistTools)
if (GUI)
    find_package(Qt5Widgets ${requiredQtVersion} REQUIRED)
    find_package(Qt5DBus ${requiredQtVersion})
endif()

set_package_properties(Qt5DBus PROPERTIES
    DESCRIPTION "Qt5 module for inter-process communication over the D-Bus protocol"
    PURPOSE "Enables communication with other system components (e.g. notification service) via D-Bus. "
    TYPE RECOMMENDED
)

set(CMAKE_AUTOMOC True)
list(APPEND CMAKE_AUTORCC_OPTIONS -compress 9 -threshold 5)
if (APPLE)
    # Workaround CMake bug (autogen does not pass required parameters to moc)
    # Relevant issue: https://gitlab.kitware.com/cmake/cmake/issues/18041
    list(APPEND CMAKE_AUTOMOC_MOC_OPTIONS -DQ_OS_MACOS -DQ_OS_DARWIN)
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# defines
add_definitions(-DQT_DEPRECATED_WARNINGS)
add_definitions(-DQT_NO_CAST_TO_ASCII)
add_definitions(-DQT_USE_QSTRINGBUILDER)
add_definitions(-DQT_STRICT_ITERATORS)

if (CMAKE_BUILD_TYPE MATCHES "Debug")
    message(STATUS "Project is built in DEBUG mode.")
else()
    message(STATUS "Project is built in RELEASE mode.")
    message(STATUS "Disabling debug output.")
    add_definitions(-DQT_NO_DEBUG_OUTPUT)
endif()

configure_file(config.h.cmakein ${CMAKE_CURRENT_BINARY_DIR}/config.h)

add_subdirectory(app)
add_subdirectory(base)

if (GUI)
    add_subdirectory(gui)
endif ()

if (WEBUI)
    add_subdirectory(webui)
endif()
