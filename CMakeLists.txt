cmake_minimum_required(VERSION 3.15)
project(nesizm LANGUAGES C CXX ASM VERSION "01.00.0000")

include(GenerateG3A)

configure_file(src/config.h.in include/config.h)
add_compile_definitions(TARGET_PRIZM=1 DEBUG=0)

file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS
  "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/*.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/*.S")

include_directories(
  "${CMAKE_CURRENT_SOURCE_DIR}/src"
  "${CMAKE_CURRENT_SOURCE_DIR}/include"
  "${CMAKE_CURRENT_BINARY_DIR}/include")

# Utils static libraries
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/utils/zx7"            "utils/zx7")
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/utils/calctype"       "utils/calctype")
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/utils/snd"            "utils/snd")
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/utils/scope_timer"    "utils/scope_timer")
add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/utils/ptune2_simple"  "utils/ptune2_simple")

add_executable(nesizm ${SOURCES})

target_compile_options(nesizm PRIVATE
  -O2 -Wall -funroll-loops -fno-trapping-math -fno-trapv
  -fno-unwind-tables -Wno-switch -Wno-stringop-truncation
  -Wno-class-memaccess -Wno-narrowing -Wno-format-overflow
  -mpretend-cmove -fdelayed-branch
  $<$<COMPILE_LANGUAGE:C>:-std=c99>
  $<$<COMPILE_LANGUAGE:CXX>:-std=c++11 -fpermissive -fno-rtti
    -fno-exceptions -fno-threadsafe-statics
    -fno-use-cxa-get-exception-ptr>)

target_link_options(nesizm PRIVATE
  -L "${CMAKE_CURRENT_SOURCE_DIR}"
  -T "${CMAKE_CURRENT_SOURCE_DIR}/prizm.ld"
  -Wl,-Map=map -Wl,-print-memory-usage)

target_link_libraries(nesizm PRIVATE
  zx7 calctype snd scope_timer ptune2_simple
  -lc -lgcc -lsyscall-${SYSCALL_PLATFORM})

if("${FXSDK_PLATFORM_LONG}" STREQUAL fxCG50)
  if("${SYSCALL_PLATFORM}" STREQUAL cw)
    generate_g3a(TARGET nesizm OUTPUT "nesizm_cg100.g3a"
      NAME "NESizm" ICONS icon-uns_w.png icon-sel_w.png)
  else()
    generate_g3a(TARGET nesizm OUTPUT "nesizm_cg50.g3a"
      NAME "NESizm" ICONS icon-uns_w.png icon-sel_w.png)
    generate_g3a(TARGET nesizm OUTPUT "nesizm_cg10.g3a"
      NAME "NESizm" ICONS icon-uns_b.png icon-sel_b.png)
  endif()
endif()
