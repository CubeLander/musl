cmake_minimum_required(VERSION 3.10)
project(musl C)



# 从原来的构建命令转换
include(ExternalProject)

# 设置输出目录
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR riscv64)
# riscv64-unknown-elf
set(CMAKE_C_COMPILER riscv64-linux-gnu-gcc)
set(CMAKE_CXX_COMPILER riscv64-linux-gnu-g++)
set(CMAKE_ASM_COMPILER riscv64-linux-gnu-gcc)
set(MUSL_OUTPUT_DIR ${CMAKE_BINARY_DIR}/lib)

# 获取处理器核心数
execute_process(COMMAND nproc OUTPUT_VARIABLE NUM_PROCESSORS OUTPUT_STRIP_TRAILING_WHITESPACE)
# 简化ExternalProject配置，只使用一个脚本处理全部步骤
ExternalProject_Add(musl_build
    SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}
    DOWNLOAD_COMMAND ""
    CONFIGURE_COMMAND ""  # 不使用单独的配置命令
    BUILD_COMMAND ${CMAKE_SOURCE_DIR}/script/build_musl.sh  # 使用我们的构建脚本
    INSTALL_COMMAND ""  # 不使用单独的安装命令
    BUILD_IN_SOURCE 0  # 不使用build-in-source，我们的脚本自己会进入源码目录
    LOG_BUILD TRUE     # 记录构建日志
)

# 创建一个接口库，以便在CMake中使用
add_library(musl INTERFACE)
add_dependencies(musl musl_build)

# 设置包含目录和库路径
target_include_directories(musl INTERFACE 
    ${CMAKE_CURRENT_BINARY_DIR}/temp_install/include
)

target_link_directories(musl INTERFACE 
    ${MUSL_OUTPUT_DIR}
)

# 将头文件安装到主项目的include目录
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/temp_install/include/
        DESTINATION ${CMAKE_SOURCE_DIR}/include/musl
        FILES_MATCHING PATTERN "*.h")

# 添加自定义目标来执行清理
add_custom_target(clean_musl
    COMMAND rm -rf ${CMAKE_CURRENT_BINARY_DIR}/temp_install
    COMMAND rm -f ${MUSL_OUTPUT_DIR}/libc.a
    COMMAND cd ${CMAKE_CURRENT_SOURCE_DIR} && make clean
    COMMENT "Cleaning musl build directories"
)

# 导出输出目录
set(MUSL_OUTPUT_DIR ${MUSL_OUTPUT_DIR} PARENT_SCOPE)