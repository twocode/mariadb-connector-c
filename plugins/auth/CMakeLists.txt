INCLUDE_DIRECTORIES(${CMAKE_SOURCE_DIR}/include)

# Dialog plugin
SET(DIALOG_SOURCES dialog.c ${CMAKE_SOURCE_DIR}/libmariadb/get_password.c)
IF(WIN32)
  SET_VERSION_INFO("TARGET:dialog"
                   "FILE_TYPE:VFT_DLL"
                   "SOURCE_FILE:plugins/auth/dialog.c"
                   "ORIGINAL_FILE_NAME:dialog.dll"
                   "FILE_DESCRIPTION:Authentication plugin")
  SET(DIALOG_SOURCES ${DIALOG_SOURCES} ${CMAKE_SOURCE_DIR}/plugins/plugin.def)
ENDIF()
ADD_LIBRARY(dialog SHARED ${dialog_RC} ${DIALOG_SOURCES})
SET_TARGET_PROPERTIES(dialog PROPERTIES PREFIX "")
SIGN_TARGET(dialog)

INSTALL(TARGETS
          dialog
          RUNTIME DESTINATION "${PLUGIN_INSTALL_DIR}"
          LIBRARY DESTINATION "${PLUGIN_INSTALL_DIR}"
          ARCHIVE DESTINATION "${PLUGIN_INSTALL_DIR}")

# Cleartext
# Dialog plugin
SET(CTEXT_SOURCES mariadb_cleartext.c)
IF(WIN32)
  SET_VERSION_INFO("TARGET:mysql_clear_password"
                     "FILE_TYPE:VFT_DLL"
                     "SOURCE_FILE:plugins/auth/mariadb_cleartext.c"
                     "ORIGINAL_FILE_NAME:mysql_clear_password.dll"
                     "FILE_DESCRIPTION:Authentication plugin")
  SET(CTEXT_SOURCES ${CTEXT_SOURCES} ${CMAKE_SOURCE_DIR}/plugins/plugin.def)
ENDIF()
ADD_LIBRARY(mysql_clear_password SHARED ${mysql_clear_password_RC} ${CTEXT_SOURCES})
SET_TARGET_PROPERTIES(mysql_clear_password PROPERTIES PREFIX "")
SIGN_TARGET(mysql_clear_password)

INSTALL(TARGETS
          mysql_clear_password
          RUNTIME DESTINATION "${PLUGIN_INSTALL_DIR}"
          LIBRARY DESTINATION "${PLUGIN_INSTALL_DIR}"
          ARCHIVE DESTINATION "${PLUGIN_INSTALL_DIR}")

IF(WIN32)
  SET(USE_SSPI 1)
ENDIF()

IF(WIN32)
  SET_VERSION_INFO("TARGET:auth_gssapi_client"
                 "FILE_TYPE:VFT_DLL"
                 "SOURCE_FILE:plugins/auth/auth_gssapi_client.c"
                 "ORIGINAL_FILE_NAME:auth_gssapi_client.dll"
                 "FILE_DESCRIPTION:Authentication plugin")
               ADD_DEFINITIONS(-DHAVE_AUTH_GSSAPI_DYNAMIC=1)
               SET(GSSAPI_SOURCES auth_gssapi_client.c sspi_client.c sspi_errmsg.c ${CMAKE_SOURCE_DIR}/plugins/plugin.def ${gssapi_RC})
ELSE()
  IF(GSSAPI_FOUND)
    SET(GSSAPI_SOURCES auth_gssapi_client.c gssapi_client.c gssapi_errmsg.c)
  ENDIF()
ENDIF()
IF(GSSAPI_FOUND OR WIN32)
  ADD_LIBRARY(auth_gssapi_client MODULE ${GSSAPI_SOURCES})
  IF(WIN32)
    TARGET_LINK_LIBRARIES(auth_gssapi_client secur32.lib)
  ELSE()
    TARGET_LINK_LIBRARIES(auth_gssapi_client ${GSSAPI_LIBS})
  ENDIF()
  SET_TARGET_PROPERTIES(auth_gssapi_client PROPERTIES PREFIX "")
  SIGN_TARGET(auth_gssapi_client)
  INSTALL(TARGETS
          auth_gssapi_client
          RUNTIME DESTINATION "${PLUGIN_INSTALL_DIR}"
          LIBRARY DESTINATION "${PLUGIN_INSTALL_DIR}"
          ARCHIVE DESTINATION "${PLUGIN_INSTALL_DIR}")
ENDIF()
