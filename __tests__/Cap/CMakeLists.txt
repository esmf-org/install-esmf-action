cmake_minimum_required(VERSION 3.19)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
find_package(ESMF 8.4.0 MODULE REQUIRED)

if(NOT DEFINED CMAKE_Fortran_COMPILER)
  set(CMAKE_Fortran_COMPILER "${ESMF_F90COMPILER}")
endif()
if(NOT DEFINED CMAKE_Fortran_FLAGS)
  set(CMAKE_Fortran_FLAGS "${ESMF_F90COMPILEOPTS}")
endif()

set(CMAKE_Fortran_MODULE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/mod")

project(Cap
        VERSION 1.0
        LANGUAGES Fortran)

add_library(Cap cap.F90)
target_link_libraries(Cap ESMF)
target_include_directories(Cap INTERFACE $<BUILD_INTERFACE:${CMAKE_Fortran_MODULE_DIRECTORY}>
                                         $<INSTALL_INTERFACE:mod>)

install(TARGETS Cap
        EXPORT Cap
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        INCLUDES DESTINATION mod
)
install(EXPORT Cap DESTINATION cmake)
install(DIRECTORY ${CMAKE_Fortran_MODULE_DIRECTORY} DESTINATION ${CMAKE_INSTALL_PREFIX})
