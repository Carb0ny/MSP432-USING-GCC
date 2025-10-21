cmake_minimum_required(VERSION 3.13)

# ==========================================================
#  1️⃣  必须在 project() 之前声明这些，否则CMake会检测Host编译器
# ==========================================================
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
set(CMAKE_EXECUTABLE_SUFFIX ".elf")  # 防止出现 .elf.exe
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR cortex-m4)

# ==========================================================
#  2️⃣  指定交叉编译工具链路径
# ==========================================================
set(TOOLCHAIN_PREFIX E:/arm_tool/bin/arm-none-eabi)
set(CMAKE_C_COMPILER ${TOOLCHAIN_PREFIX}-gcc.exe)
set(CMAKE_ASM_COMPILER ${TOOLCHAIN_PREFIX}-gcc.exe)
set(CMAKE_OBJCOPY ${TOOLCHAIN_PREFIX}-objcopy)
set(CMAKE_SIZE ${TOOLCHAIN_PREFIX}-size)

# ==========================================================
#  3️⃣  项目信息
# ==========================================================
project(MSP432P401R C ASM)

# ==========================================================
#  4️⃣  编译与链接选项
# ==========================================================
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mcpu=cortex-m4 -mthumb -ffreestanding -Wall -O2 -g")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -T${CMAKE_SOURCE_DIR}/linker/msp432p401r.lds -nostartfiles -Wl,--gc-sections --specs=nosys.specs")
# ==========================================================
#  5️⃣  添加头文件目录（根据SDK路径调整）
# ==========================================================


include_directories(
    ${CMAKE_SOURCE_DIR}/inc
    ${CMAKE_SOURCE_DIR}/ti/devices/msp432p4xx/driverlib
    ${CMAKE_SOURCE_DIR}/ti/devices/msp432p4xx/inc
    ${CMAKE_SOURCE_DIR}/ti/CMSIS/Include
    ${CMAKE_SOURCE_DIR}
)

# ==========================================================
#  6️⃣  指定目标芯片宏定义
# ==========================================================
add_definitions(-D__MSP432P401R__)

# ==========================================================
#  7️⃣  源文件（自动收集driverlib）
# ==========================================================
file(GLOB DRIVERLIB_SRC
    ${CMAKE_SOURCE_DIR}/ti/devices/msp432p4xx/driverlib/*.c
)
file(GLOB USER_SOURCE_FILE
    ${CMAKE_SOURCE_DIR}/src/*.c)

add_executable(${PROJECT_NAME}
    ti/startup_msp432p401r_gcc.c
    ti/system_msp432p401r.c
    ${DRIVERLIB_SRC}
    ${USER_SOURCE_FILE}
)
set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX ".elf")
# ==========================================================
#  8️⃣  构建后生成二进制文件
# ==========================================================
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    # 使用生成器表达式获取正确的文件名
    COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${PROJECT_NAME}> ${PROJECT_NAME}.bin
    COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${PROJECT_NAME}>
)
