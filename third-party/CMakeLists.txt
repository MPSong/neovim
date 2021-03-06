# This is not meant to be included by the top-level.
cmake_minimum_required (VERSION 2.8.7)
project(NVIM_DEPS)

# Point CMake at any custom modules we may ship
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")

# In Windows/MSVC CMAKE_BUILD_TYPE changes the paths/linking of the build
# recipes (libuv, msgpack), make sure it is set
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

set(DEPS_INSTALL_DIR "${CMAKE_BINARY_DIR}/usr" CACHE PATH "Dependencies install directory.")
set(DEPS_BIN_DIR "${DEPS_INSTALL_DIR}/bin" CACHE PATH "Dependencies binary install directory.")
set(DEPS_LIB_DIR "${DEPS_INSTALL_DIR}/lib" CACHE PATH "Dependencies library install directory.")
set(DEPS_BUILD_DIR "${CMAKE_BINARY_DIR}/build" CACHE PATH "Dependencies build directory.")
set(DEPS_DOWNLOAD_DIR "${DEPS_BUILD_DIR}/downloads" CACHE PATH "Dependencies download directory.")

option(USE_BUNDLED "Use bundled dependencies." ON)

option(USE_BUNDLED_JEMALLOC "Use the bundled jemalloc." ${USE_BUNDLED})
option(USE_BUNDLED_UNIBILIUM "Use the bundled unibilium." ${USE_BUNDLED})
option(USE_BUNDLED_LIBTERMKEY "Use the bundled libtermkey." ${USE_BUNDLED})
option(USE_BUNDLED_LIBVTERM "Use the bundled libvterm." ${USE_BUNDLED})
option(USE_BUNDLED_LIBUV "Use the bundled libuv." ${USE_BUNDLED})
option(USE_BUNDLED_MSGPACK "Use the bundled msgpack." ${USE_BUNDLED})
option(USE_BUNDLED_LUAJIT "Use the bundled version of luajit." ${USE_BUNDLED})
option(USE_BUNDLED_LUAROCKS "Use the bundled version of luarocks." ${USE_BUNDLED})
option(USE_BUNDLED_LUV "Use the bundled version of luv." ${USE_BUNDLED})
#XXX(tarruda): Lua is only used for debugging the functional test client, no
# build it unless explicitly requested
option(USE_BUNDLED_LUA "Use the bundled version of lua." OFF)

if(USE_BUNDLED  AND (NOT WIN32))
  option(USE_BUNDLED_GPERF "Use the bundled version of gperf." ON)
else()
  option(USE_BUNDLED_GPERF "Use the bundled version of gperf." OFF)
endif()

option(USE_EXISTING_SRC_DIR "Skip download of deps sources in case of existing source directory." OFF)

if(UNIX)
  find_program(MAKE_PRG NAMES gmake make)
  if(MAKE_PRG)
    execute_process(
      COMMAND "${MAKE_PRG}" --version
      OUTPUT_VARIABLE MAKE_VERSION_INFO)
    if(NOT "${OUTPUT_VARIABLE}" MATCHES ".*GNU.*")
      unset(MAKE_PRG)
    endif()
  endif()
  if(NOT MAKE_PRG)
    message(FATAL_ERROR "GNU Make is required to build the dependencies.")
  else()
    message(STATUS "Found GNU Make at ${MAKE_PRG}")
  endif()
endif()

# When using make, use the $(MAKE) variable to avoid warning about the job
# server.
if(CMAKE_GENERATOR MATCHES "Makefiles")
  set(MAKE_PRG "$(MAKE)")
endif()

if(CMAKE_C_COMPILER_ARG1)
  set(DEPS_C_COMPILER "${CMAKE_C_COMPILER} ${CMAKE_C_COMPILER_ARG1}")
else()
  set(DEPS_C_COMPILER "${CMAKE_C_COMPILER}")
endif()

# Cross compiling: use these for dependencies built for the
# HOST system, when not crosscompiling these should be the
# same as DEPS_*. Except when targeting Unix in which case
# want all the dependencies to use the same compiler.
if(CMAKE_CROSSCOMPILING AND NOT UNIX)
  set(HOSTDEPS_INSTALL_DIR "${CMAKE_BINARY_DIR}/host")
  set(HOSTDEPS_BIN_DIR "${HOSTDEPS_INSTALL_DIR}/bin")
  set(HOSTDEPS_LIB_DIR "${HOSTDEPS_INSTALL_DIR}/lib")
  set(HOSTDEPS_C_COMPILER "${HOST_C_COMPILER}")
else()
  set(HOSTDEPS_INSTALL_DIR "${DEPS_INSTALL_DIR}")
  set(HOSTDEPS_BIN_DIR "${DEPS_BIN_DIR}")
  set(HOSTDEPS_LIB_DIR "${DEPS_LIB_DIR}")
  set(HOSTDEPS_C_COMPILER "${DEPS_C_COMPILER}")
endif()

include(ExternalProject)

set(LIBUV_URL https://github.com/libuv/libuv/archive/v1.9.1.tar.gz)
set(LIBUV_SHA256 a6ca9f0648973d1463f46b495ce546ddcbe7cce2f04b32e802a15539e46c57ad)

set(MSGPACK_URL https://github.com/msgpack/msgpack-c/archive/cpp-1.0.0.tar.gz)
set(MSGPACK_SHA256 afda64ca445203bb7092372b822bae8b2539fdcebbfc3f753f393628c2bcfe7d)

set(LUAJIT_URL https://github.com/neovim/deps/raw/master/opt/LuaJIT-2.0.4.tar.gz)
set(LUAJIT_SHA256 620fa4eb12375021bef6e4f237cbd2dd5d49e56beb414bee052c746beef1807d)

set(LUA_URL https://www.lua.org/ftp/lua-5.1.5.tar.gz)
set(LUA_SHA256 2640fc56a795f29d28ef15e13c34a47e223960b0240e8cb0a82d9b0738695333)

set(LUAROCKS_URL https://github.com/keplerproject/luarocks/archive/5d8a16526573b36d5b22aa74866120c998466697.tar.gz)
set(LUAROCKS_SHA256 cae709111c5701235770047dfd7169f66b82ae1c7b9b79207f9df0afb722bfd9)

set(UNIBILIUM_URL https://github.com/mauke/unibilium/archive/v1.2.0.tar.gz)
set(UNIBILIUM_SHA256 623af1099515e673abfd3cae5f2fa808a09ca55dda1c65a7b5c9424eb304ead8)

set(LIBTERMKEY_URL http://www.leonerd.org.uk/code/libtermkey/libtermkey-0.19.tar.gz)
set(LIBTERMKEY_SHA256 c505aa4cb48c8fa59c526265576b97a19e6ebe7b7da20f4ecaae898b727b48b7)

set(LIBVTERM_URL https://github.com/neovim/libvterm/archive/a9c7c6fd20fa35e0ad3e0e98901ca12dfca9c25c.tar.gz)
set(LIBVTERM_SHA256 1a4272be91d9614dc183a503786df83b6584e4afaab7feaaa5409f841afbd796)

set(JEMALLOC_URL https://github.com/jemalloc/jemalloc/releases/download/4.3.1/jemalloc-4.3.1.tar.bz2)
set(JEMALLOC_SHA256 f7bb183ad8056941791e0f075b802e8ff10bd6e2d904e682f87c8f6a510c278b)

set(LUV_URL https://github.com/luvit/luv/archive/1.9.1-0.tar.gz)
set(LUV_SHA256 86a199403856018cd8e5529c8527450c83664a3d36f52d5253cbe909ea6c5a06)

set(GPERF_URL http://ftp.gnu.org/pub/gnu/gperf/gperf-3.0.4.tar.gz)
set(GPERF_SHA256 767112a204407e62dbc3106647cf839ed544f3cf5d0f0523aaa2508623aad63e)

# 7za.exe cat.exe curl.exe ca-bundle.crt diff.exe tee.exe tidy.exe xxd.exe
set(WINTOOLS_URL https://github.com/neovim/deps/raw/2f9acbecf06365c10baa3c0087f34a54c9c6f949/opt/win32tools.zip)
set(WINTOOLS_SHA256 8bfce7e3a365721a027ce842f2ec1cf878f1726233c215c05964aac07300798c)

set(WINGUI_URL https://github.com/equalsraf/neovim-qt/releases/download/v0.2.6/neovim-qt.zip)
set(WINGUI_SHA256 90217351e9e51c81ef5bba39066f00d05e15ef1f881882c3c682e61cd446c211)

set(WIN32YANK_URL https://github.com/equalsraf/win32yank/releases/download/v0.0.3/win32yank.zip)
set(WIN32YANK_SHA256 b474439ed2854a9a24941d66970c7fcfece219401eaaa5ebc0ffcc962e69887a)

set(WINPTY_URL https://github.com/rprichard/winpty/releases/download/0.4.2/winpty-0.4.2-msvc2015.zip)
set(WINPTY_SHA256 b465f2584ff394b3fe27c01aa1dcfc679583c1ee951d0e83de3f859d8b8305b8)

if(USE_BUNDLED_UNIBILIUM)
  include(BuildUnibilium)
endif()

if(USE_BUNDLED_LIBTERMKEY)
  include(BuildLibtermkey)
endif()

if(USE_BUNDLED_LIBVTERM)
  include(BuildLibvterm)
endif()

if(USE_BUNDLED_LIBUV)
  include(BuildLibuv)
endif()

if(USE_BUNDLED_MSGPACK)
  include(BuildMsgpack)
endif()

if(USE_BUNDLED_LUAJIT)
  include(BuildLuajit)
endif()

if(USE_BUNDLED_LUA AND NOT CMAKE_CROSSCOMPILING)
  include(BuildLua)
endif()

if(USE_BUNDLED_LUAROCKS)
  include(BuildLuarocks)
endif()

if(USE_BUNDLED_JEMALLOC)
  include(BuildJeMalloc)
endif()

if(USE_BUNDLED_LUV)
  include(BuildLuv)
endif()

if(USE_BUNDLED_GPERF)
  include(BuildGperf)
endif()

include(GetBinaryDeps)

if(WIN32)
  GetBinaryDep(TARGET wintools
    INSTALL_COMMAND ${CMAKE_COMMAND} -E copy_directory . ${DEPS_INSTALL_DIR}/bin)

  GetBinaryDep(TARGET win32yank
    INSTALL_COMMAND ${CMAKE_COMMAND} -E copy win32yank.exe ${DEPS_INSTALL_DIR}/bin)

  GetBinaryDep(TARGET wingui
    INSTALL_COMMAND ${CMAKE_COMMAND} -E copy_directory bin ${DEPS_INSTALL_DIR}/bin)

  include(TargetArch)
  if("${TARGET_ARCH}" STREQUAL "X86_64")
    set(TARGET_ARCH x64)
  elseif(TARGET_ARCH STREQUAL "X86")
    set(TARGET_ARCH ia32)
  endif()
  GetBinaryDep(TARGET winpty
    INSTALL_COMMAND ${CMAKE_COMMAND} -E make_directory ${DEPS_INSTALL_DIR}/bin
      COMMAND ${CMAKE_COMMAND} -DFROM_GLOB=${DEPS_BUILD_DIR}/src/winpty/${TARGET_ARCH}/bin/*
        -DTO=${DEPS_INSTALL_DIR}/bin/
        -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/CopyFilesGlob.cmake
      COMMAND ${CMAKE_COMMAND} -DFROM_GLOB=${DEPS_BUILD_DIR}/src/winpty/include/*
        -DTO=${DEPS_INSTALL_DIR}/include/
        -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/CopyFilesGlob.cmake
      COMMAND ${CMAKE_COMMAND} -DFROM_GLOB=${DEPS_BUILD_DIR}/src/winpty/${TARGET_ARCH}/lib/*
        -DTO=${DEPS_INSTALL_DIR}/lib/
        -P ${CMAKE_CURRENT_SOURCE_DIR}/cmake/CopyFilesGlob.cmake)
endif()

add_custom_target(clean-shared-libraries
  COMMAND ${CMAKE_COMMAND}
    -DREMOVE_FILE_GLOB=${DEPS_INSTALL_DIR}/lib/${CMAKE_SHARED_LIBRARY_PREFIX}*${CMAKE_SHARED_LIBRARY_SUFFIX}*
    -P ${PROJECT_SOURCE_DIR}/cmake/RemoveFiles.cmake
  DEPENDS ${THIRD_PARTY_DEPS}
)

add_custom_target(third-party ALL
  COMMAND ${CMAKE_COMMAND} -E touch .third-party
  DEPENDS clean-shared-libraries)
