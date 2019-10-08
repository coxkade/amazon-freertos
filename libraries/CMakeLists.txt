# FreeRTOS module.
afr_module(NAME freertos INTERFACE KERNEL)
afr_glob_src(freertos_inc DIRECTORY "${AFR_KERNEL_DIR}/include" RECURSE)
if(NOT DEFINED AFR_KERNEL_SOURCE_OVERRIDES)
message(STATUS "\r\n Using default Kernel Source \r\n")
afr_module_sources(
    freertos
    INTERFACE
        "${AFR_KERNEL_DIR}/event_groups.c"
        "${AFR_KERNEL_DIR}/list.c"
        "${AFR_KERNEL_DIR}/queue.c"
        "${AFR_KERNEL_DIR}/stream_buffer.c"
        "${AFR_KERNEL_DIR}/tasks.c"
        "${AFR_KERNEL_DIR}/timers.c"
        ${freertos_inc}
)
else()
message(STATUS "\r\n Using Override Kernel Source \r\n")
afr_module_sources(
    freertos
    INTERFACE
        ${AFR_KERNEL_SOURCE_OVERRIDES}
)
endif() #if(NOT DEFINED AFR_KERNEL_SOURCE_OVERRIDES)
if(NOT DEFINED AFR_KERNEL_INC_OVERRIDE)
afr_module_include_dirs(
    freertos
    INTERFACE
        "${AFR_KERNEL_DIR}/include"
)
else()
afr_module_include_dirs(
    freertos
    INTERFACE
        "${AFR_KERNEL_INC_OVERRIDE}"
)
endif() #if(NOT DEFINED AFR_KERNEL_INC_OVERRIDE)

# Kernel module.
afr_module(NAME kernel INTERNAL KERNEL)

afr_module_dependencies(
    kernel
    PRIVATE
        AFR::freertos
        AFR::kernel::mcu_port
    PUBLIC
        AFR::compiler::mcu_port
)

# Add FreeRTOS and mcu portable layer include path to kernel's public interface.
afr_module_include_dirs(
    kernel
    PUBLIC
        $<TARGET_PROPERTY:AFR::freertos,INTERFACE_INCLUDE_DIRECTORIES>
        $<TARGET_PROPERTY:AFR::kernel::mcu_port,INTERFACE_INCLUDE_DIRECTORIES>
        $<${AFR_IS_TESTING}:${AFR_3RDPARTY_DIR}/unity/src>
)

# Add other modules.
file(GLOB afr_modules_csdk "${AFR_MODULES_DIR}/c_sdk/*/*")
file(GLOB afr_modules_freertos_plus "${AFR_MODULES_DIR}/freertos_plus/*/*")
file(GLOB afr_modules_abstractions "${AFR_MODULES_DIR}/abstractions/*")
foreach(module IN LISTS afr_modules_csdk afr_modules_freertos_plus afr_modules_abstractions)
    if(IS_DIRECTORY "${module}" AND EXISTS "${module}/CMakeLists.txt")
        afr_add_subdirectory("${module}")
    endif()
endforeach()
