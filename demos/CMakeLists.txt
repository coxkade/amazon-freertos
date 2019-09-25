# -------------------------------------------------------------------------------------------------
# Demos
# -------------------------------------------------------------------------------------------------

#Custom find file
function(internal_find_file var file_name dir)
    string(STRIP ${file_name} strip_name)
    FILE(GLOB children RELATIVE ${dir} ${dir}/*)
    unset(${var})
    foreach(child ${children})
    string(STRIP ${child} strip_child)
    if("${strip_child}" STREQUAL "${strip_name}")
        get_filename_component(t_path ${dir} ABSOLUTE)
        set(${var} "${t_path}/${file_name}" PARENT_SCOPE)
        return()
    endif()
    endforeach(child ${children})
endfunction()

#Function that attempts to find a file in a list of paths
function(find_file_in_path_list var file_name path_list)
    unset(${var} PARENT_SCOPE)
    foreach(dir ${path_list})
    get_filename_component(abs_dir ${dir} ABSOLUTE)
    internal_find_file(${var} ${file_name} ${abs_dir})
    if(${var})
        set(${var} ${${var}} PARENT_SCOPE)
        return() #The file was found
    endif()
    endforeach()
endfunction()

#macro to help me add items to the base sources
macro(add_header_to_base_sources header)
    if(DEFINED ESP_OVERRIDE_INCLUDES)
        find_file_in_path_list(found_file ${header} ${ESP_OVERRIDE_INCLUDES})
    endif()
    if(NOT found_file)
        unset(def_demo_path)
        list(APPEND def_demo_path ${AFR_DEMOS_DIR}/include)
        find_file_in_path_list(found_file ${header} ${def_demo_path})
    endif()
    if(NOT found_file)
        message(FATAL_ERROR "failed to find header file: ${header}")
    else()
        list(APPEND base_sources ${found_file})
    endif()
endmacro()

list(APPEND base_sources 
        "$<$<TARGET_EXISTS:AFR::freertos_plus_tcp>:${AFR_DEMOS_DIR}/demo_runner/aws_demo_network_addr.c>"
        "${AFR_DEMOS_DIR}/demo_runner/aws_demo_version.c"
        "${AFR_DEMOS_DIR}/demo_runner/aws_demo.c"
        "${AFR_DEMOS_DIR}/network_manager/aws_iot_network_manager.c"
        "${AFR_DEMOS_DIR}/network_manager/aws_iot_demo_network.c"
        "${AFR_DEMOS_DIR}/network_manager/iot_network_manager_private.h"
        "${AFR_DEMOS_DIR}/demo_runner/iot_demo_freertos.c"
        "${AFR_DEMOS_DIR}/demo_runner/iot_demo_runner.c"
)
add_header_to_base_sources(aws_application_version.h)
add_header_to_base_sources(aws_clientcredential.h)
add_header_to_base_sources(aws_clientcredential_keys.h)
add_header_to_base_sources(aws_demo.h)
add_header_to_base_sources(aws_iot_demo_network.h)
add_header_to_base_sources(aws_ota_codesigner_certificate.h)
add_header_to_base_sources(iot_config_common.h)
add_header_to_base_sources(iot_demo_logging.h)
add_header_to_base_sources(iot_demo_runner.h)

# Base demo target
afr_demo_module(base)
afr_module_sources(
    ${AFR_CURRENT_MODULE}
    INTERFACE
        ${base_sources}
)
afr_module_include_dirs(
    ${AFR_CURRENT_MODULE}
    INTERFACE
        "${AFR_DEMOS_DIR}/include"
        "${AFR_DEMOS_DIR}/network_manager"
)
afr_module_dependencies(
    ${AFR_CURRENT_MODULE}
    INTERFACE
        AFR::kernel
        AFR::common
)
if(TARGET AFR::pkcs11_implementation::mcu_port)
    afr_module_dependencies(
        ${AFR_CURRENT_MODULE}
        INTERFACE
            AFR::dev_mode_key_provisioning
    )
endif()

if(TARGET AFR::wifi::mcu_port AND TARGET AFR::ble_hal::mcu_port)
    afr_module_dependencies(
        ${AFR_CURRENT_MODULE}
        INTERFACE
        AFR::ble_wifi_provisioning
    )
endif()

# Add other modules.
file(GLOB afr_demos "${AFR_DEMOS_DIR}/*")
foreach(demo IN LISTS afr_demos)
    if(IS_DIRECTORY "${demo}" AND EXISTS "${demo}/CMakeLists.txt")
        afr_add_subdirectory("${demo}")
    endif()
endforeach()
