cmake_minimum_required (VERSION 3.6)

include(ExternalProject)

project (SuiteSparse LANGUAGES C )

option(BUILD_STATIC "build static libraries" ON)
option(BUILD_AMD "routines for pre-ordering sparse matrices prior to factorization" ON)
option(BUILD_CAMD "routines for permuting sparse matrices prior to factorization" ON)
option(BUILD_CCOLAMD "constrained column approximate minimum degree ordering" ON)
option(BUILD_COLAMD "column approximate minimum degree ordering" ON)
option(BUILD_CHOLMOD "sparse CHOLesky MODification package" ON)
option(BUILD_UMFPACK "routines solving sparse linear systems via LU factorization" ON)

option(ALLOW_64BIT_BLAS "look for 32-bit BLAS only or concurrent 32- and 64-bit BLAS" OFF)

# this are the common arguments for all individual cmake projects
set(CMAKE_ARGS
  -DNSTATIC:BOOL=(NOT BUILD_STATIC) 
  -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX})

# the CMAKE_SOURCE_DIR points to the root directory, independent from where it's CMakeLists.txt is called
# the add the desired subpackages. Via DEPENDS the proper order is guaranteed
ExternalProject_Add(SuiteSparse_config
  PREFIX "SuiteSparse_config"
  SOURCE_DIR "${CMAKE_SOURCE_DIR}/SuiteSparse_config"
  BINARY_DIR "${CMAKE_SOURCE_DIR}/SuiteSparse_config/build"
  CMAKE_ARGS 
    ${CMAKE_ARGS} 
    -DALLOW_64BIT_BLAS=${ALLOW_64BIT_BLAS})

if (BUILD_AMD OR BUILD_CHOLMOD OR BUILD_UMFPACK)
  if (NOT BUILD_AMD)
    message(FATAL_ERROR "enable BUILD_AMD, required by other package")
    # unfortunately set(BUILD_AMD ON FORCE) is ignored :(
  endif()
  ExternalProject_Add(AMD
    PREFIX "AMD"
    SOURCE_DIR "${CMAKE_SOURCE_DIR}/AMD"
    BINARY_DIR "${CMAKE_SOURCE_DIR}/AMD/build"
    DEPENDS SuiteSparse_config
    CMAKE_ARGS ${CMAKE_ARGS} )
endif()

if (BUILD_CAMD OR BUILD_CHOLMOD)
  if (NOT BUILD_CAMD)
    message(FATAL_ERROR "enable BUILD_CAMD, required by CHOLMOD")
  endif()
  ExternalProject_Add(CAMD
    PREFIX "CAMD"
    SOURCE_DIR "${CMAKE_SOURCE_DIR}/CAMD"
    BINARY_DIR "${CMAKE_SOURCE_DIR}/CAMD/build"
    DEPENDS SuiteSparse_config
    CMAKE_ARGS ${CMAKE_ARGS} )
endif()

if (BUILD_CCOLAMD OR BUILD_CHOLMOD)
  if (NOT BUILD_CCOLAMD)
    message(FATAL_ERROR "enable BUILD_CCOLAMD, required by CHOLMOD")
  endif()
  ExternalProject_Add(CCOLAMD
    PREFIX "CCOLAMD"
    SOURCE_DIR "${CMAKE_SOURCE_DIR}/CCOLAMD"
    BINARY_DIR "${CMAKE_SOURCE_DIR}/CCOLAMD/build"
    DEPENDS SuiteSparse_config
    CMAKE_ARGS ${CMAKE_ARGS} )
endif()

if (BUILD_COLAMD OR BUILD_CHOLMOD)
  if (NOT BUILD_COLAMD)
    message(FATAL_ERROR "enable BUILD_COLAMD, required by CHOLMOD")
  endif()
  ExternalProject_Add(COLAMD
    PREFIX "COLAMD"
    SOURCE_DIR "${CMAKE_SOURCE_DIR}/COLAMD"
    BINARY_DIR "${CMAKE_SOURCE_DIR}/COLAMD/build"
    DEPENDS SuiteSparse_config
    CMAKE_ARGS ${CMAKE_ARGS} )
endif()

if (BUILD_CHOLMOD)
  ExternalProject_Add(CHOLMOD
    PREFIX "CHOLMOD"
    SOURCE_DIR "${CMAKE_SOURCE_DIR}/CHOLMOD"
    BINARY_DIR "${CMAKE_SOURCE_DIR}/CHOLMOD/build"
    DEPENDS SuiteSparse_config COLAMD CCOLAMD AMD
    CMAKE_ARGS 
      ${CMAKE_ARGS} 
      -DALLOW_64BIT_BLAS=${ALLOW_64BIT_BLAS} )
endif()

if (BUILD_UMFPACK)
  ExternalProject_Add(UMFPACK
    PREFIX "UMFPACK"
    SOURCE_DIR "${CMAKE_SOURCE_DIR}/UMFPACK"
    BINARY_DIR "${CMAKE_SOURCE_DIR}/UMFPACK/build"
    DEPENDS SuiteSparse_config 
    CMAKE_ARGS 
      ${CMAKE_ARGS} 
      -DALLOW_64BIT_BLAS=${ALLOW_64BIT_BLAS} )
endif ()

