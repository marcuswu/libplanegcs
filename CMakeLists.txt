cmake_minimum_required(VERSION 3.9)
# Uncomment this code if you want to try Eigen's multithread #find_package(OpenMP) #if (OPENMP_FOUND) set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
#    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
#endif()

# set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DEIGEN_NO_DEBUG") # Necessary to avoid SparseQR crash with Eigen 3.2 due to a wrong assertion
project(PlaneGCS VERSION 1.0.0)
string(TOLOWER ${PROJECT_NAME} PROJECT_NAME_LOWERCASE)
set(LIBRARY_NAME ${PROJECT_NAME_LOWERCASE})
set(LIBRARY_FOLDER ${PROJECT_NAME_LOWERCASE})
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(GENERATED_DIR       "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(GENERATED_HEADERS_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated_headers")
set(VERSION_CONFIG_FILE "${GENERATED_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(PROJECT_CONFIG_FILE "${GENERATED_DIR}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(CONFIG_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")

find_package (Boost REQUIRED COMPONENTS graph)
find_package (Eigen3 REQUIRED)
set (INCLUDE_PATH /usr/local/include /usr/include/eigen3)
set (LIB_PATH /usr/local/lib /usr/lib)
include_directories(${INCLUDE_PATH})
include_directories(${Boost_INCLUDE_DIRS})
include_directories(${Eigen3_INCLUDE_DIRS})
link_directories(${LIB_PATH})

include_directories(
    ${EIGEN3_INCLUDE_DIR}
)

SET(PlaneGCS_LIBS
    ${Eigen3_LIBRARIES}
    ${Boost_LIBRARIES}
)

SET(PlaneGCS_SRCS
    src/GCS.cpp
    src/GCS.h
    src/Util.h
    src/Geo.cpp
    src/Geo.h
    src/Constraints.cpp
    src/Constraints.h
    src/SubSystem.cpp
    src/SubSystem.h
    src/qp_eq.cpp
    src/qp_eq.h
)

set(HEADERS_PUBLIC
    src/GCS.h
)

SOURCE_GROUP("${LIBRARY_NAME}" FILES ${PlaneGCS_SRCS})

add_library(${LIBRARY_NAME} SHARED ${PlaneGCS_SRCS})
target_link_libraries(${LIBRARY_NAME} ${PlaneGCS_LIBS})

target_include_directories(
    ${LIBRARY_NAME} PUBLIC
      "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>"
      "$<BUILD_INTERFACE:${GENERATED_HEADERS_DIR}>"
      "$<INSTALL_INTERFACE:.>"
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${VERSION_CONFIG_FILE}"
    VERSION "${${PROJECT_NAME}_VERSION}"
    COMPATIBILITY SameMajorVersion
)
configure_package_config_file(
    "${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in"
    "${PROJECT_CONFIG_FILE}"
    INSTALL_DESTINATION "${CONFIG_INSTALL_DIR}"
)

include(GNUInstallDirs)
install(
    TARGETS              "${LIBRARY_NAME}"
    EXPORT               "${TARGETS_EXPORT_NAME}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

install(
    FILES ${HEADERS_PUBLIC}
    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${LIBRARY_FOLDER}"
)

install(
    FILES "${PROJECT_CONFIG_FILE}"
          "${VERSION_CONFIG_FILE}"
    DESTINATION "${CONFIG_INSTALL_DIR}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    FILE   "${PROJECT_NAME}Targets.cmake"
    DESTINATION "${CONFIG_INSTALL_DIR}"
    NAMESPACE "${PROJECT_NAME}::"
)

# Suppress some very long Eigen3 warnings of older versions
if (EIGEN3_NO_DEPRECATED_COPY)
    set_source_files_properties(
        src/GCS.cpp
        src/SubSystem.cpp
        src/qp_eq.cpp
        PROPERTIES COMPILE_FLAGS ${EIGEN3_NO_DEPRECATED_COPY})
endif ()

# INSTALL(TARGETS PlaneGCS DESTINATION ${CMAKE_INSTALL_LIBDIR})
