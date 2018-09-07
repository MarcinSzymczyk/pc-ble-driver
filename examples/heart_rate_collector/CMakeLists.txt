cmake_minimum_required(VERSION 3.3)

# Use multithreaded Boost libraries
set(Boost_USE_MULTITHREADED ON)

# Use static boost libraries so the dynamic library 
# can run anywhere
set(Boost_USE_STATIC_LIBS ON)

set(CMAKE_C_FLAGS "-pthread")

# Find the necessary boost components on the system.
# Minimum version required is 1.54.0
find_package ( Boost 1.54.0 REQUIRED COMPONENTS thread system regex date_time chrono )

link_directories (
    ../../build
)

include_directories (
    ../../include/common/sdk_compat
    ../../include/common
    ../../include/common/internal/transport
    ../../../civetweb/include
)

if(MSVC)
    add_definitions(-DPC_BLE_DRIVER_STATIC)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

# Build executable
add_executable(hrc_v2 main.c ../../../civetweb/src/civetweb.c ../../../civetweb/src/md5.inl ../../../civetweb/src/sha1.inl ../../../civetweb/src/handle_form.inl ../../../civetweb/src/timer.inl)
add_executable(hrc_v5 main.c ../../../civetweb/src/civetweb.c ../../../civetweb/src/md5.inl ../../../civetweb/src/sha1.inl ../../../civetweb/src/handle_form.inl ../../../civetweb/src/timer.inl)

target_compile_definitions(hrc_v2 PRIVATE -DNRF_SD_BLE_API=2)
target_compile_definitions(hrc_v5 PRIVATE -DNRF_SD_BLE_API=5)

target_include_directories(hrc_v2 PRIVATE ../../src/sd_api_v2/sdk/components/softdevice/s132/headers)
target_include_directories(hrc_v5 PRIVATE ../../src/sd_api_v5/sdk/components/softdevice/s132/headers)

# Specify libraries to link with
if(MSVC)
    target_link_libraries(hrc_v2 PRIVATE pc_ble_driver_static_sd_api_v2)
    target_link_libraries(hrc_v5 PRIVATE pc_ble_driver_static_sd_api_v5)
else()
    target_link_libraries(hrc_v2 PRIVATE pc_ble_driver_shared_sd_api_v2)
    target_link_libraries(hrc_v5 PRIVATE pc_ble_driver_shared_sd_api_v5)
endif()

target_link_libraries(hrc_v2 PRIVATE ${Boost_LIBRARIES} dl)
target_link_libraries(hrc_v5 PRIVATE ${Boost_LIBRARIES} dl)
