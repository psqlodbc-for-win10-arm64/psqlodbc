cmake_minimum_required(VERSION 3.29)
project(pgxalib C CXX)

set(pgxalib_srcPath "../../")

add_library(pgxalib SHARED
  ${pgxalib_srcPath}pgxalib.cpp
  ${pgxalib_srcPath}xalibname.c
)

target_compile_definitions(pgxalib PUBLIC
  _HANDLE_ENLIST_IN_DTC_
  _CRT_SECURE_NO_DEPRECATE
  PGENLIST_EXPORTS
)

target_link_libraries(pgxalib PRIVATE
  "wsock32.lib"
  "ws2_32.lib"
  "odbc32.lib"
  "legacy_stdio_definitions.lib" # MSVC_VER >= 1900 # https://stackoverflow.com/a/37080275
  "odbccp32.lib"
)
