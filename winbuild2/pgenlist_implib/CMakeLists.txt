cmake_minimum_required(VERSION 3.29)
project(pgenlist_implib C CXX)

set(pgenlist_srcPath "../../")

if(ANSI_VERSION)
  set(pgenlist_library_name pgenlista)
  set(pgenlist_def_file "pgenlista.def")
else()
  set(pgenlist_library_name pgenlist)
  set(pgenlist_def_file "pgenlist.def")
endif()

add_library(${pgenlist_library_name} SHARED
  ${pgenlist_srcPath}${pgenlist_def_file} 
  dummy.c
)

set_source_files_properties(
  ${pgenlist_srcPath}${pgenlist_def_file} PROPERTIES HEADER_FILE_ONLY TRUE
)

get_filename_component(pgenlist_def_file_abs "${pgenlist_srcPath}${pgenlist_def_file}" ABSOLUTE)

set_target_properties(${pgenlist_library_name} PROPERTIES LINK_FLAGS
  "/DEF:${pgenlist_def_file_abs}"
)
