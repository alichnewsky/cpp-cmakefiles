# ~~~
# Copyright 2019 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ~~~

cmake_minimum_required(VERSION 3.5)

# Define the project name and where to report bugs.
set(PACKAGE_BUGREPORT "https://github.com/googleapis/google-cloud-cpp/issues")
project(googleapis-cpp-protos CXX C)

set(GOOGLEAPIS_CPP_PROTOS_VERSION_MAJOR 0)
set(GOOGLEAPIS_CPP_PROTOS_VERSION_MINOR 2)
set(GOOGLEAPIS_CPP_PROTOS_VERSION_PATCH 0)

string(CONCAT GOOGLE_APIS_CPP_PROTOS_VERSION
              "${GOOGLEAPIS_CPP_PROTOS_VERSION_MAJOR}"
              "."
              "${GOOGLEAPIS_CPP_PROTOS_VERSION_MINOR}"
              "."
              "${GOOGLEAPIS_CPP_PROTOS_VERSION_PATCH}")

# Configure the compiler options, we will be using C++11 features.
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Give application developers a hook to configure the version and hash
# downloaded from GitHub.
set(
    GOOGLE_CLOUD_CPP_GOOGLEAPIS_URL
    "https://github.com/googleapis/googleapis/archive/a8ee1416f4c588f2ab92da72e7c1f588c784d3e6.tar.gz"
    )
set(GOOGLE_CLOUD_CPP_GOOGLEAPIS_SHA256
    "6b8a9b2bcb4476e9a5a9872869996f0d639c8d5df76dd8a893e79201f211b1cf")

set(GOOGLEAPIS_CPP_SOURCE
    "${CMAKE_BINARY_DIR}/external/googleapis/src/googleapis_download")

include(ExternalProject)
ExternalProject_Add(
    googleapis_download
    EXCLUDE_FROM_ALL ON
    PREFIX "${CMAKE_BINARY_DIR}/external/googleapis"
    URL ${GOOGLE_CLOUD_CPP_GOOGLEAPIS_URL}
    URL_HASH SHA256=${GOOGLE_CLOUD_CPP_GOOGLEAPIS_SHA256}
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    INSTALL_COMMAND ""
    BUILD_BYPRODUCTS
        "${GOOGLEAPIS_CPP_SOURCE}/google/api/http.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/api/annotations.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/api/auth.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/api/resource.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/type/expr.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/rpc/error_details.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/rpc/status.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/iam/v1/policy.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/iam/v1/iam_policy.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/longrunning/operations.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/admin/v2/bigtable_instance_admin.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/admin/v2/bigtable_table_admin.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/admin/v2/common.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/admin/v2/instance.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/admin/v2/table.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/v2/bigtable.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/v2/data.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/admin/database/v1/spanner_database_admin.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/admin/instance/v1/spanner_instance_admin.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/keys.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/mutation.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/query_plan.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/result_set.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/spanner.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/transaction.proto"
        "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/type.proto"
    LOG_DOWNLOAD OFF)

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")
find_package(ProtobufTargets REQUIRED)
find_package(gRPC REQUIRED)

# Sometimes (this happens often with vcpkg) protobuf is installed in a non-
# standard directory. We need to find out where, and then add that directory to
# the search path for protos.
find_path(PROTO_INCLUDE_DIR google/protobuf/descriptor.proto)
if (PROTO_INCLUDE_DIR)
    list(INSERT PROTOBUF_IMPORT_DIRS 0 "${PROTO_INCLUDE_DIR}")
endif ()

add_library(googleapis_cpp_common_flags INTERFACE)

include(SelectMSVCRuntime)

# Include the functions to compile proto files.
include(CompileProtos)

google_cloud_cpp_add_protos_property()

function (googleapis_cpp_set_version_and_alias short_name)
    add_dependencies("googleapis_cpp_${short_name}" googleapis_download)
    set_target_properties("googleapis_cpp_${short_name}"
                          PROPERTIES VERSION
                                     "${GOOGLE_APIS_CPP_PROTOS_VERSION}"
                                     SOVERSION
                                     ${GOOGLEAPIS_CPP_PROTOS_VERSION_MAJOR})
    add_library("googleapis-c++::${short_name}" ALIAS
                "googleapis_cpp_${short_name}")
endfunction ()

google_cloud_cpp_grpcpp_library(googleapis_cpp_api_http_protos
                                "${GOOGLEAPIS_CPP_SOURCE}/google/api/http.proto"
                                PROTO_PATH_DIRECTORIES
                                "${GOOGLEAPIS_CPP_SOURCE}"
                                "${PROTO_INCLUDE_DIR}")
googleapis_cpp_set_version_and_alias(api_http_protos)
target_link_libraries(googleapis_cpp_api_http_protos
                      PRIVATE googleapis_cpp_common_flags)

google_cloud_cpp_grpcpp_library(
    googleapis_cpp_api_annotations_protos
    "${GOOGLEAPIS_CPP_SOURCE}/google/api/annotations.proto"
    PROTO_PATH_DIRECTORIES
    "${GOOGLEAPIS_CPP_SOURCE}"
    "${PROTO_INCLUDE_DIR}")
googleapis_cpp_set_version_and_alias(api_annotations_protos)
target_link_libraries(googleapis_cpp_api_annotations_protos
                      PUBLIC googleapis-c++::api_http_protos
                      PRIVATE googleapis_cpp_common_flags)

google_cloud_cpp_grpcpp_library(googleapis_cpp_api_auth_protos
                                "${GOOGLEAPIS_CPP_SOURCE}/google/api/auth.proto"
                                PROTO_PATH_DIRECTORIES
                                "${GOOGLEAPIS_CPP_SOURCE}"
                                "${PROTO_INCLUDE_DIR}")
googleapis_cpp_set_version_and_alias(api_auth_protos)
target_link_libraries(googleapis_cpp_api_auth_protos
                      PUBLIC googleapis-c++::api_annotations_protos
                      PRIVATE googleapis_cpp_common_flags)

google_cloud_cpp_grpcpp_library(
    googleapis_cpp_api_resource_protos
    "${GOOGLEAPIS_CPP_SOURCE}/google/api/resource.proto"
    PROTO_PATH_DIRECTORIES
    "${GOOGLEAPIS_CPP_SOURCE}"
    "${PROTO_INCLUDE_DIR}")
googleapis_cpp_set_version_and_alias(api_resource_protos)
target_link_libraries(googleapis_cpp_api_resource_protos
                      PRIVATE googleapis_cpp_common_flags)

google_cloud_cpp_grpcpp_library(
    googleapis_cpp_type_expr_protos
    "${GOOGLEAPIS_CPP_SOURCE}/google/type/expr.proto"
    PROTO_PATH_DIRECTORIES
    "${GOOGLEAPIS_CPP_SOURCE}"
    "${PROTO_INCLUDE_DIR}")
googleapis_cpp_set_version_and_alias(type_expr_protos)
target_link_libraries(googleapis_cpp_type_expr_protos
                      PRIVATE googleapis_cpp_common_flags)

google_cloud_cpp_grpcpp_library(
    googleapis_cpp_rpc_error_details_protos
    "${GOOGLEAPIS_CPP_SOURCE}/google/rpc/error_details.proto"
    PROTO_PATH_DIRECTORIES
    "${GOOGLEAPIS_CPP_SOURCE}"
    "${PROTO_INCLUDE_DIR}")
googleapis_cpp_set_version_and_alias(rpc_error_details_protos)
target_link_libraries(googleapis_cpp_rpc_error_details_protos
                      PRIVATE googleapis_cpp_common_flags)

google_cloud_cpp_grpcpp_library(
    googleapis_cpp_rpc_status_protos
    "${GOOGLEAPIS_CPP_SOURCE}/google/rpc/status.proto"
    PROTO_PATH_DIRECTORIES
    "${GOOGLEAPIS_CPP_SOURCE}"
    "${PROTO_INCLUDE_DIR}")
googleapis_cpp_set_version_and_alias(rpc_status_protos)
target_link_libraries(googleapis_cpp_rpc_status_protos
                      PUBLIC googleapis-c++::rpc_error_details_protos
                      PRIVATE googleapis_cpp_common_flags)

google_cloud_cpp_grpcpp_library(
    googleapis_cpp_iam_v1_policy_protos
    "${GOOGLEAPIS_CPP_SOURCE}/google/iam/v1/policy.proto"
    PROTO_PATH_DIRECTORIES
    "${GOOGLEAPIS_CPP_SOURCE}"
    "${PROTO_INCLUDE_DIR}")
googleapis_cpp_set_version_and_alias(iam_v1_policy_protos)
target_link_libraries(googleapis_cpp_iam_v1_policy_protos
                      PUBLIC googleapis-c++::api_annotations_protos
                             googleapis-c++::api_resource_protos
                             googleapis-c++::type_expr_protos
                      PRIVATE googleapis_cpp_common_flags)

google_cloud_cpp_grpcpp_library(
    googleapis_cpp_iam_v1_iam_policy_protos
    "${GOOGLEAPIS_CPP_SOURCE}/google/iam/v1/iam_policy.proto"
    PROTO_PATH_DIRECTORIES
    "${GOOGLEAPIS_CPP_SOURCE}"
    "${PROTO_INCLUDE_DIR}")
googleapis_cpp_set_version_and_alias(iam_v1_iam_policy_protos)
target_link_libraries(googleapis_cpp_iam_v1_iam_policy_protos
                      PUBLIC googleapis-c++::api_annotations_protos
                             googleapis-c++::iam_v1_policy_protos
                      PRIVATE googleapis_cpp_common_flags)

google_cloud_cpp_grpcpp_library(
    googleapis_cpp_longrunning_operations_protos
    "${GOOGLEAPIS_CPP_SOURCE}/google/longrunning/operations.proto"
    PROTO_PATH_DIRECTORIES
    "${GOOGLEAPIS_CPP_SOURCE}"
    "${PROTO_INCLUDE_DIR}")
googleapis_cpp_set_version_and_alias(longrunning_operations_protos)
target_link_libraries(googleapis_cpp_longrunning_operations_protos
                      PUBLIC googleapis-c++::api_annotations_protos
                             googleapis-c++::rpc_status_protos
                      PRIVATE googleapis_cpp_common_flags)

google_cloud_cpp_grpcpp_library(
    googleapis_cpp_bigtable_protos
    "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/admin/v2/bigtable_instance_admin.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/admin/v2/bigtable_table_admin.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/admin/v2/common.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/admin/v2/instance.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/admin/v2/table.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/v2/bigtable.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/bigtable/v2/data.proto"
    PROTO_PATH_DIRECTORIES
    "${GOOGLEAPIS_CPP_SOURCE}"
    "${PROTO_INCLUDE_DIR}")
googleapis_cpp_set_version_and_alias(bigtable_protos)
target_link_libraries(googleapis_cpp_bigtable_protos
                      PUBLIC googleapis-c++::api_annotations_protos
                             googleapis-c++::api_auth_protos
                             googleapis-c++::longrunning_operations_protos
                             googleapis-c++::rpc_status_protos
                             googleapis-c++::iam_v1_iam_policy_protos
                      PRIVATE googleapis_cpp_common_flags)

google_cloud_cpp_grpcpp_library(
    googleapis_cpp_spanner_protos
    "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/admin/database/v1/spanner_database_admin.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/admin/instance/v1/spanner_instance_admin.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/keys.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/mutation.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/query_plan.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/result_set.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/spanner.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/transaction.proto"
    "${GOOGLEAPIS_CPP_SOURCE}/google/spanner/v1/type.proto"
    PROTO_PATH_DIRECTORIES
    "${GOOGLEAPIS_CPP_SOURCE}"
    "${PROTO_INCLUDE_DIR}")
googleapis_cpp_set_version_and_alias(spanner_protos)
target_link_libraries(googleapis_cpp_spanner_protos
                      PUBLIC googleapis-c++::api_annotations_protos
                             googleapis-c++::longrunning_operations_protos
                             googleapis-c++::rpc_status_protos
                             googleapis-c++::iam_v1_iam_policy_protos
                      PRIVATE googleapis_cpp_common_flags)

# Install the libraries and headers in the locations determined by
# GNUInstallDirs
include(GNUInstallDirs)

set(googleapis_cpp_installed_libraries_list
    googleapis_cpp_bigtable_protos
    googleapis_cpp_spanner_protos
    googleapis_cpp_longrunning_operations_protos
    googleapis_cpp_api_http_protos
    googleapis_cpp_api_annotations_protos
    googleapis_cpp_api_auth_protos
    googleapis_cpp_api_resource_protos
    googleapis_cpp_iam_v1_policy_protos
    googleapis_cpp_iam_v1_iam_policy_protos
    googleapis_cpp_rpc_error_details_protos
    googleapis_cpp_rpc_status_protos
    googleapis_cpp_type_expr_protos)

install(TARGETS ${googleapis_cpp_installed_libraries_list}
                googleapis_cpp_common_flags
        EXPORT googleapis-targets
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

foreach (target ${googleapis_cpp_installed_libraries_list})
    google_cloud_cpp_install_proto_library_headers("${target}")
    google_cloud_cpp_install_proto_library_protos("${target}"
                                                  "${GOOGLEAPIS_CPP_SOURCE}")
endforeach ()

# Export the CMake targets to make it easy to create configuration files.
install(EXPORT googleapis-targets
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/googleapis")

# Setup global variables used in the following *.in files.
set(GOOGLE_CLOUD_CPP_CONFIG_VERSION_MAJOR
    ${GOOGLEAPIS_CPP_PROTOS_VERSION_MAJOR})
set(GOOGLE_CLOUD_CPP_CONFIG_VERSION_MINOR
    ${GOOGLEAPIS_CPP_PROTOS_VERSION_MINOR})
set(GOOGLE_CLOUD_CPP_CONFIG_VERSION_PATCH
    ${GOOGLEAPIS_CPP_PROTOS_VERSION_PATCH})

# Use a function to create a scope for the variables.
function (googleapis_cpp_install_pc target)
    string(REPLACE "googleapis_cpp_"
                   ""
                   _short_name
                   ${target})
    string(REPLACE "_protos"
                   ""
                   _short_name
                   ${_short_name})
    set(GOOGLE_CLOUD_CPP_PC_NAME
        "The Google APIS C++ ${_short_name} Proto Library")
    set(GOOGLE_CLOUD_CPP_PC_DESCRIPTION "Compiled proto for C++.")
    # Examine the target LINK_LIBRARIES property, use that to pull the
    # dependencies between the googleapis-c++::* libraries.
    set(_target_pc_requires)
    get_target_property(_target_deps ${target} LINK_LIBRARIES)
    foreach (dep ${_target_deps})
        if ("${dep}" MATCHES "^googleapis-c\\+\\+::")
            string(REPLACE "googleapis-c++::"
                           "googleapis_cpp_"
                           dep
                           "${dep}")
            list(APPEND _target_pc_requires " " "${dep}")
        endif ()
    endforeach ()
    # These dependencies are required for all the googleapis-c++::* libraries.
    list(APPEND _target_pc_requires
                " grpc++"
                " grpc"
                " openssl"
                " protobuf"
                " zlib"
                " libcares")
    string(CONCAT GOOGLE_CLOUD_CPP_PC_REQUIRES ${_target_pc_requires})
    set(GOOGLE_CLOUD_CPP_PC_LIBS "-l${target}")
    configure_file("cmake/config.pc.in" "${target}.pc" @ONLY)
    install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${target}.pc"
            DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")
endfunction ()

# Create and install the pkg-config files.
foreach (target ${googleapis_cpp_installed_libraries_list})
    googleapis_cpp_install_pc("${target}")
endforeach ()

# Create and install the googleapis pkg-config file for backwards compatibility.
set(GOOGLE_CLOUD_CPP_PC_NAME "The Google APIS C++ Proto Library")
set(GOOGLE_CLOUD_CPP_PC_DESCRIPTION
    "Provides C++ APIs to access Google Cloud Platforms.")
# Note the use of spaces, `string(JOIN)` is not available in cmake-3.5, so we
# need to add the separator ourselves.
string(CONCAT GOOGLE_CLOUD_CPP_PC_REQUIRES
              "googleapis_cpp_bigtable_protos"
              " googleapis_cpp_iam_v1_iam_policy_protos"
              " googleapis_cpp_iam_v1_policy_protos"
              " googleapis_cpp_longrunning_operations_protos"
              " googleapis_cpp_api_auth_protos"
              " googleapis_cpp_api_annotations_protos"
              " googleapis_cpp_api_http_protos"
              " googleapis_cpp_rpc_status_protos"
              " googleapis_cpp_rpc_error_details_protos"
              " grpc++"
              " grpc"
              " openssl"
              " protobuf"
              " zlib"
              " libcares")
set(GOOGLE_CLOUD_CPP_PC_LIBS "")
configure_file("cmake/config.pc.in" "googleapis.pc" @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/googleapis.pc"
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")

# Create and install the CMake configuration files.
configure_file("cmake/config.cmake.in" "googleapis-config.cmake" @ONLY)
configure_file("cmake/config-version.cmake.in" "googleapis-config-version.cmake"
               @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/googleapis-config.cmake"
              "${CMAKE_CURRENT_BINARY_DIR}/googleapis-config-version.cmake"
              "${PROJECT_SOURCE_DIR}/cmake/FindgRPC.cmake"
              "${PROJECT_SOURCE_DIR}/cmake/FindProtobufTargets.cmake"
              "${PROJECT_SOURCE_DIR}/cmake/CompileProtos.cmake"
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/googleapis")
