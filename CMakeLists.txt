###############################################################################
#  (c) 2024 Copyright, Real-Time Innovations, Inc. (RTI) All rights reserved. #
#                                                                             #
#  RTI grants Licensee a license to use, modify, compile, and create          #
#  derivative works of the software solely for use with RTI Connext DDS.      #
#  Licensee may redistribute copies of the software provided that all such    #
#  copies are subject to this license.                                        #
#  The software is provided "as is", with no warranty of any type, including  #
#  any warranty for fitness for any purpose. RTI is under no obligation to    #
#  maintain or support the software.  RTI shall not be liable for any         #
#  incidental or consequential damages arising out of the use or inability to #
#  use the software.                                                          #
#                                                                             #
###############################################################################

# Load the dependencies of dds-datamodel-utils and rticonnextdds-cmake-utils.
# The dds-datamodel-utils repo may be provided by setting the variable
# DDS_DATAMODELS_UTILS_DIR to point to a local copy of this repository.
# If this is not provided, this macro downloads it as a dependency.
# This macro alsso loads all necessary files from the dependencies.
# This sets one variable that may be used in the parent scope:
#   * DDS_DATAMODELS_UTILS_DIR: the path to the datamodels-utils repo

option(DDS_DATAMODELS_CONVERT_TO_XML "Conver datamodel to XML" OFF)
option(DDS_DATAMODELS_BUILD_CXX11_LIB "Build a C++11 library from the types in this datamodel" ON)

macro (configure_datamodels_dependencies)
    # Get the absolute path for the dds-datamodels-utils repo if it is defined
    if (NOT DEFINED DDS_DATAMODELS_UTILS_DIR)
        set(DDS_DATAMODELS_UTILS_DIR "${CMAKE_CURRENT_BINARY_DIR}/dds-datamodels-utils")
    endif()

    get_filename_component(datamodels_utils_name_ "${DDS_DATAMODELS_UTILS_DIR}" NAME)

    if(NOT IS_DIRECTORY ${DDS_DATAMODELS_UTILS_DIR})
        include(FetchContent)

        # Set this policy to avoid updating any submodule repositories
        cmake_policy(SET CMP0097 NEW)

        # Fetch the content of the rticonnextdds-cmake-utils repo
        FetchContent_Declare(${datamodels_utils_name_}
            GIT_REPOSITORY https://github.com/rticommunity/dds-datamodels-utils.git
            GIT_TAG main
            SOURCE_DIR "${DDS_DATAMODELS_UTILS_DIR}"
        )
        FetchContent_MakeAvailable(${datamodels_utils_name_})
    endif()

    unset(datamodels_utils_name_)

    get_filename_component(RTI_CMAKE_UTILS_DIR "${DDS_DATAMODELS_UTILS_DIR}/rticonnextdds-cmake-utils" ABSOLUTE)

    list(APPEND CMAKE_MODULE_PATH "${DDS_DATAMODELS_UTILS_DIR}/cmake")
    list(APPEND CMAKE_MODULE_PATH "${RTI_CMAKE_UTILS_DIR}/cmake/Modules")
endmacro()

# Starting the library project
cmake_minimum_required(VERSION 3.16)

set(project_name DdsDatamodelsOaris)
set(library_name rtidatamodeloaris)

project(${project_name} VERSION 1.3.1)
set(CMAKE_CXX_STANDARD 11)

# Downloads and configure the dependencies to build this datamodel
configure_datamodels_dependencies()

find_package(
    RTIConnextDDS 6.1.2
    COMPONENTS core
    REQUIRED
)

include(ConnextDdsDatamodelsGenerateCode)
include(ConnextDdsDatamodelsConvertToXml)

# get a list of files to ignore
file(GLOB ignored_files_full_name "datamodel/idl/*Dcps.idl")

set(ignored_files)
foreach(idl_file_path in ${ignored_files_full_name})
    get_filename_component(idl_name ${idl_file_path} NAME_WE)
    list(APPEND ignored_files ${idl_name})
endforeach()

if (DDS_DATAMODELS_CONVERT_TO_XML)

    connextdds_datamodels_convert_to_xml(
        INPUT_FOLDERS "datamodel/idl"
        OUTPUT_FOLDER "${CMAKE_CURRENT_BINARY_DIR}/datamodel/xml"
        IGNORE_IDL_NAMES ${ignored_files}
        IDL_DEPENDENCIES_FOLDERS "${DDS_DATAMODELS_UTILS_DIR}/dds"
        CODEGEN_EXTRA_ARGS "-verbosity" "1"
    )
endif()
if (DDS_DATAMODELS_BUILD_CXX11_LIB)
        connextdds_datamodels_generate_code(
        INPUT_FOLDERS "datamodel/idl"
        OUTPUT_FOLDER "${CMAKE_CURRENT_BINARY_DIR}/datamodel/idl"
        LANG C++11
        IGNORE_IDL_NAMES ${ignored_files}
        IDL_DEPENDENCIES_FOLDERS "${DDS_DATAMODELS_UTILS_DIR}/dds"
        CODEGEN_EXTRA_ARGS "-verbosity" "1"
    )

    add_library(${library_name}
        SHARED
            ${GENERATED_SRC_FILES}
    )

    target_include_directories(${library_name}
        PRIVATE
            ${CONNEXTDDS_INCLUDE_DIRS}
            "${CMAKE_CURRENT_BINARY_DIR}/datamodel/idl"
            "${CMAKE_CURRENT_BINARY_DIR}/dependencies"
    )

    target_link_libraries(${library_name}
        PRIVATE
            RTIConnextDDS::cpp2_api
    )
endif()

