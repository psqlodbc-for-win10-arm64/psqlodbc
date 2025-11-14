cmake_minimum_required(VERSION 3.29)
project(psqlodbc C CXX)

set(psqlodbc_srcPath "../../")

if(ANSI_VERSION)
  set(psqlodbc_library_name psqlodbc30a)
  set(maindef ${psqlodbc_srcPath}psqlodbca.def)
else()
  set(psqlodbc_library_name psqlodbc35w)
  set(maindef ${psqlodbc_srcPath}psqlodbc.def)
endif()

add_library(${psqlodbc_library_name} SHARED
  ${psqlodbc_srcPath}catfunc.h
  ${psqlodbc_srcPath}bind.c
  ${psqlodbc_srcPath}columninfo.c
  ${psqlodbc_srcPath}connection.c
  ${psqlodbc_srcPath}convert.c
  ${psqlodbc_srcPath}descriptor.c
  ${psqlodbc_srcPath}dlg_specific.c
  ${psqlodbc_srcPath}dlg_wingui.c
  ${psqlodbc_srcPath}drvconn.c
  ${psqlodbc_srcPath}environ.c
  ${psqlodbc_srcPath}execute.c
  ${psqlodbc_srcPath}info.c
  $<IF:$<BOOL:${MEMORY_DEBUG}>,${psqlodbc_srcPath}inouealc.c,>
  ${psqlodbc_srcPath}loadlib.c
  ${psqlodbc_srcPath}lobj.c
  ${psqlodbc_srcPath}misc.c
  ${psqlodbc_srcPath}multibyte.c
  ${psqlodbc_srcPath}mylog.c
  ${psqlodbc_srcPath}odbcapi.c
  ${psqlodbc_srcPath}odbcapi30.c
  $<IF:$<BOOL:${ANSI_VERSION}>,,${psqlodbc_srcPath}odbcapi30w.c>
  $<IF:$<BOOL:${ANSI_VERSION}>,,${psqlodbc_srcPath}odbcapiw.c>
  ${psqlodbc_srcPath}options.c
  ${psqlodbc_srcPath}parse.c
  ${psqlodbc_srcPath}pgapi30.c
  ${psqlodbc_srcPath}pgtypes.c
  ${psqlodbc_srcPath}psqlodbc.c
  ${psqlodbc_srcPath}qresult.c
  ${psqlodbc_srcPath}results.c
  ${psqlodbc_srcPath}setup.c
  ${psqlodbc_srcPath}statement.c
  ${psqlodbc_srcPath}tuple.c
  ${psqlodbc_srcPath}secure_sscanf.c
  $<IF:$<BOOL:${ANSI_VERSION}>,,${psqlodbc_srcPath}win_unicode.c>
  $<IF:$<BOOL:${MSDTC}>,${psqlodbc_srcPath}xalibname.c,>
  ${psqlodbc_srcPath}psqlodbc.rc
  ${maindef}
)

set_source_files_properties(
  ${maindef}
  PROPERTIES HEADER_FILE_ONLY TRUE
)

target_compile_definitions(${psqlodbc_library_name} PUBLIC
  DYNAMIC_LOAD
  WIN_MULTITHREAD_SUPPORT
  _CRT_SECURE_NO_DEPRECATE # odbccp32.lib(dllload.obj) : error LNK2019: unresolved external symbol _vsnwprintf_s referenced in function StringCchPrintfW
  $<IF:$<BOOL:${MSDTC}>,_HANDLE_ENLIST_IN_DTC_,>
)

if(ANSI_VERSION)
  target_compile_definitions(${psqlodbc_library_name} PUBLIC

  )
  set(DTCDLL "pgenlista")
else()
  target_compile_definitions(${psqlodbc_library_name} PUBLIC
    UNICODE_SUPPORT
    UNICODE_SUPPORTXX
  )
  set(DTCDLL "pgenlist")
endif()

if(MEMORY_DEBUG)
  target_compile_definitions(${psqlodbc_library_name} PUBLIC
    _MEMORY_DEBUG_
  )
endif()

if(MIMALLOC)
  target_compile_definitions(${psqlodbc_library_name} PUBLIC
    _MIMALLOC_
  )
  set(MIMALLOC_include "../../libs/mimalloc/include")
  set(CALL_LIB "mimalloc-static.lib")
else()
  set(MIMALLOC_include "")
  set(CALL_LIB "")
endif()

target_include_directories(${psqlodbc_library_name} PUBLIC
  ${PG_INC}
  ${PG_INC}/internal
  ${PG_INC}/server
  ${PG_INC}/server/port/win32
  ${MIMALLOC_include}
)

target_link_libraries(${psqlodbc_library_name} PRIVATE
  ${CALL_LIB}
  "libpq.lib"
  "advapi32.lib"
  "shell32.lib"
  "ole32.lib"
  "oleaut32.lib"
  "uuid.lib"
  "odbc32.lib"
  "legacy_stdio_definitions.lib" # MSVC_VER >= 1900 # https://stackoverflow.com/a/37080275
  "odbccp32.lib"
  "wsock32.lib"
  "ws2_32.lib"
  "secur32.lib"
  "winmm.lib"
)

target_link_directories(${psqlodbc_library_name} PRIVATE
  ${PG_LIB}
  ${PGENLIST_LIB}
)

get_filename_component(maindef_abs "${maindef}" ABSOLUTE)

set_target_properties(${psqlodbc_library_name} PROPERTIES LINK_FLAGS
  "/DELAYLOAD:secur32.dll /DELAYLOAD:${DTCDLL}.dll /DEF:${maindef_abs}"
)
