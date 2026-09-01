cmake_minimum_required(VERSION 3.29)
project(pgenlist C CXX)

set(pgenlist_srcPath "../../")

if(ANSI_VERSION)
  set(pgenlist_library_name pgenlista)
  set(psqlodbc_lib_file "psqlodbc30a.lib")
  set(pgenlist_def_file "pgenlista.def")
else()
  set(pgenlist_library_name pgenlist)
  set(psqlodbc_lib_file "psqlodbc35w.lib")
  set(pgenlist_def_file "pgenlist.def")
endif()

add_library(${pgenlist_library_name} SHARED
  ${pgenlist_srcPath}msdtc_enlist.cpp
  ${pgenlist_srcPath}xalibname.c
  ${pgenlist_srcPath}${pgenlist_def_file}
)

set_source_files_properties(
  ${pgenlist_srcPath}${pgenlist_def_file} PROPERTIES HEADER_FILE_ONLY TRUE
)

target_compile_definitions(${pgenlist_library_name} PUBLIC
  _HANDLE_ENLIST_IN_DTC_
  _CRT_SECURE_NO_DEPRECATE # odbccp32.lib(dllload.obj) : error LNK2019: unresolved external symbol _vsnwprintf_s referenced in function StringCchPrintfW
  PGENLIST_EXPORTS
)

target_include_directories(${pgenlist_library_name} PUBLIC
)

target_link_libraries(${pgenlist_library_name} PRIVATE
  "xolehlp.lib"
  ${psqlodbc_lib_file}
)

target_link_directories(${pgenlist_library_name} PRIVATE
  ${PSQLODBC_LIB}
)

get_filename_component(pgenlist_def_file_abs "${pgenlist_srcPath}${pgenlist_def_file}" ABSOLUTE)

set_target_properties(${pgenlist_library_name} PROPERTIES LINK_FLAGS
  "/DELAYLOAD:XOLEHLP.dll /DEF:${pgenlist_def_file_abs}"
)
