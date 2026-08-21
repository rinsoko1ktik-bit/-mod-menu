cmake_minimum_required(VERSION 3.20)

project(ZokiiStoryMenu LANGUAGES CXX)

if(NOT DEFINED ENV{OO_PS4_TOOLCHAIN})
    message(FATAL_ERROR
        "OO_PS4_TOOLCHAIN is not set. "
        "It must point to the OpenOrbis toolchain."
    )
endif()

set(OO_PS4_TOOLCHAIN "$ENV{OO_PS4_TOOLCHAIN}")

set(CMAKE_CXX_COMPILER clang++)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(SOURCE_FILE "${CMAKE_SOURCE_DIR}/src/main.cpp")
set(OUT_DIR "${CMAKE_BINARY_DIR}/orbis")
set(ELF_FILE "${OUT_DIR}/ZokiiStoryMenu.elf")
set(OELF_FILE "${OUT_DIR}/ZokiiStoryMenu.oelf")
set(EBOOT_FILE "${CMAKE_BINARY_DIR}/eboot.bin")

file(MAKE_DIRECTORY "${OUT_DIR}")

add_custom_command(
    OUTPUT "${ELF_FILE}"
    COMMAND clang++
        --target=x86_64-pc-freebsd12-elf
        -fPIC
        -funwind-tables
        -c
        "${SOURCE_FILE}"
        -o "${OUT_DIR}/main.o"
        -isysroot "${OO_PS4_TOOLCHAIN}"
        -isystem "${OO_PS4_TOOLCHAIN}/include"
        -isystem "${OO_PS4_TOOLCHAIN}/include/c++/v1"
    COMMAND ld.lld
        -m elf_x86_64
        -pie
        --script "${OO_PS4_TOOLCHAIN}/link.x"
        --eh-frame-hdr
        -L"${OO_PS4_TOOLCHAIN}/lib"
        -lc
        -lkernel
        -lc++
        "${OO_PS4_TOOLCHAIN}/lib/crt1.o"
        "${OUT_DIR}/main.o"
        -o "${ELF_FILE}"
    DEPENDS "${SOURCE_FILE}"
)

add_custom_command(
    OUTPUT "${EBOOT_FILE}"
    COMMAND "${OO_PS4_TOOLCHAIN}/bin/linux/create-fself"
        -in="${ELF_FILE}"
        -out="${OELF_FILE}"
        --eboot "${EBOOT_FILE}"
        --paid 0x3800000000000011
    DEPENDS "${ELF_FILE}"
)

add_custom_target(ZokiiStoryMenu ALL
    DEPENDS "${EBOOT_FILE}"
)

message(STATUS "OpenOrbis toolchain: ${OO_PS4_TOOLCHAIN}")
message(STATUS "PS4 eboot output: ${EBOOT_FILE}")
