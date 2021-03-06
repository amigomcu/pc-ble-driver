enable_testing()

# Use the latest SoftDevice API version in the tests
#list(GET SD_API_VER_NUMS -1 SD_API_VER)

message(STATUS "Using SoftDevice API version ${SD_API_VER} in tests.")

if(MSVC)
    add_definitions(-DPC_BLE_DRIVER_STATIC)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

include_directories (
    ../3rdparty
    ../include/common/sdk_compat
    ../include/common
    ../include/common/internal/transport
    ../src/sd_api_v${SD_API_VER}/sdk/components/softdevice/s132/headers
)

file(GLOB tests_src "test_*.cpp")

foreach(SD_API_VER ${SD_API_VER_NUMS})
    foreach(test_src ${tests_src})
        get_filename_component(test_name ${test_src} NAME_WE)
        set(test_name "${test_name}_v${SD_API_VER}")

        # Build executable
        add_executable(${test_name} ${test_src})
        target_compile_definitions(${test_name} PRIVATE -DNRF_SD_BLE_API=${SD_API_VER} -DNRF_LOG_FILENAME="${test_name}.txt")
        target_include_directories(${test_name} PRIVATE ../src/sd_api_v${SD_API_VER}/sdk/components/softdevice/s132/headers)

        if(WIN32)
            target_link_libraries(${test_name} PRIVATE pc_ble_driver_static_sd_api_v${SD_API_VER})
        elseif(APPLE)
            target_link_libraries(${test_name} PRIVATE pc_ble_driver_static_sd_api_v${SD_API_VER})
        else()
            # Assume Linux
            target_link_libraries(${test_name} PRIVATE pc_ble_driver_static_sd_api_v${SD_API_VER} "pthread")
        endif()

        add_test(NAME ${test_name} COMMAND ${test_name})

        message(STATUS "Added test ${test_src}, with test name ${test_name}.")
    endforeach(test_src)
endforeach(SD_API_VER)
