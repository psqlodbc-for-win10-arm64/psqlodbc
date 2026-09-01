cmake_minimum_required(VERSION 3.29)
project(psqlsetup C CXX)

set(psqlsetup_srcPath "../../")

if(ANSI_VERSION)
  set(psqlsetup_library_name psqlsetupa)
  set(CALL_LIB "psqlodbc30a.lib")
  set(DELAY_LOAD_DLLS_OPT "/DELAYLOAD:psqlodbc30a.dll /DELAYLOAD:pgenlista.dll")
else()
  set(psqlsetup_library_name psqlsetup)
  set(CALL_LIB "psqlodbc35w.lib")
  set(DELAY_LOAD_DLLS_OPT "/DELAYLOAD:psqlodbc35w.dll /DELAYLOAD:pgenlist.dll")
endif()

add_library(${psqlsetup_library_name} SHARED
  ${psqlsetup_srcPath}psqlsetup.c
  ${psqlsetup_srcPath}setup.c
  ${psqlsetup_srcPath}dlg_wingui.c
  ${psqlsetup_srcPath}dlg_specific.c
  ${psqlsetup_srcPath}mylog.c
  ${psqlsetup_srcPath}xalibname.c
  ${psqlsetup_srcPath}misc.c
  ${psqlsetup_srcPath}loadlib.c
  ${psqlsetup_srcPath}secure_sscanf.c
  ${psqlsetup_srcPath}psqlodbc.rc
)

target_compile_definitions(${psqlsetup_library_name} PUBLIC
  PG_BIN="${PG_BIN}"
  DYNAMIC_LOAD
  _HANDLE_ENLIST_IN_DTC_
  _CRT_SECURE_NO_DEPRECATE # odbccp32.lib(dllload.obj) : error LNK2019: unresolved external symbol _vsnwprintf_s referenced in function StringCchPrintfW
  PSQLSETUP_EXPORTS
)

if(ANSI_VERSION)
  target_compile_definitions(${psqlsetup_library_name} PUBLIC

  )
else()
  target_compile_definitions(${psqlsetup_library_name} PUBLIC
    UNICODE_SUPPORT
  )
endif()

target_link_libraries(${psqlsetup_library_name} PRIVATE
  ${CALL_LIB}
  "winmm.lib"
  "odbc32.lib"
  "legacy_stdio_definitions.lib" # MSVC_VER >= 1900 # https://stackoverflow.com/a/37080275
  "odbccp32.lib"
)

target_link_directories(${psqlsetup_library_name} PRIVATE
  ${PSQLODBC_LIB}
  ${PGENLIST_LIB}
)

set_target_properties(${psqlsetup_library_name} PROPERTIES LINK_FLAGS
  "${DELAY_LOAD_DLLS_OPT}"
)
