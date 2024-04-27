cmake_minimum_required(VERSION 3.5)

project(OptimLib)


set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED YES)
set(CMAKE_CXX_EXTENSIONS NO)

add_library(optim SHARED)

set(OPTIM_SRC_DIR ${CMAKE_SOURCE_DIR}/src)
set(OPTIM_HEADER_DIR ${CMAKE_SOURCE_DIR}/include)

add_compile_options(-mcpu=native -ffp-contract=fast -flto)

if(NOT DEFINED OPTIM_FPN_TYPE)
    set(OPTIM_FPN_TYPE double)
endif()

target_compile_definitions(optim PRIVATE OPTIM_FPN_TYPE=${OPTIM_FPN_TYPE})
message(STATUS "floating-point number type: ${OPTIM_FPN_TYPE}")

target_include_directories(optim PRIVATE ${OPTIM_HEADER_DIR})
file(GLOB OPTIM_SRC_FILES ${OPTIM_SRC_DIR}/**/*.cpp)
target_sources(optim PRIVATE ${OPTIM_SRC_FILES})

if(OPTIM_LINEAR_ALG_LIB STREQUAL "eigen")
    target_compile_definitions(optim PRIVATE OPTIM_ENABLE_EIGEN_WRAPPERS)
    find_package(Eigen3 REQUIRED)
    target_include_directories(optim PRIVATE ${EIGEN3_INCLUDE_DIR})
elseif(OPTIM_LINEAR_ALG_LIB STREQUAL "arma")
    target_compile_definitions(optim PRIVATE OPTIM_ENABLE_ARMA_WRAPPERS)
    find_package(Armadillo REQUIRED)
    target_include_directories(optim PRIVATE ${ARMADILLO_INCLUDE_DIRS})
    target_link_libraries(optim PUBLIC ${ARMADILLO_LIBRARIES})
else()
    message(FATAL_ERROR "OPTIM_LINEAR_ALG_LIB must be set.")
endif()

find_package(OpenMP)
if (OpenMP_CXX_FOUND)
    target_link_libraries(optim PUBLIC OpenMP::OpenMP_CXX)
    message(STATUS "OpenMP features: enabled")
else()
    message(STATUS "OpenMP features: disabled")
endif()

include(FindBLAS)

if(BLAS_FOUND)
    target_compile_definitions(optim PRIVATE OPTIM_ENABLE_BLAS_WRAPPERS)
    target_include_directories(optim PRIVATE ${BLAS_INCLUDE_DIRS})
    target_link_libraries(optim PUBLIC ${BLAS_LIBRARIES})
    message(STATUS "BLAS features: enabled")
else()
    message(STATUS "BLAS features: disabled")
endif()

message(STATUS "OPTIM_LINEAR_ALG_LIB: ${OPTIM_LINEAR_ALG_LIB}")

# Tests

if (BUILD_TESTING)
    enable_testing()
    add_subdirectory(tests)
endif()

# Install

install(TARGETS optim DESTINATION lib)
message(STATUS "installing headers from ${OPTIM_HEADER_DIR} to include/optim")
install(DIRECTORY ${OPTIM_HEADER_DIR}/ DESTINATION include/optim FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp" PATTERN "BaseMatrixOps" EXCLUDE)
install(DIRECTORY ${OPTIM_HEADER_DIR}/BaseMatrixOps/include/ DESTINATION include/optim/BaseMatrixOps/include FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp")
