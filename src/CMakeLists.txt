include(AmxConfig)
include(Plugin)
include(FindSDK)

if(NOT WIN32 AND MYSQLCAPI_LIBRARY_STATIC)
	option(BUILD_STATIC "Build static library" OFF)
endif()

set(SAMP_SDK_DIR "../libs")
find_package(SDK REQUIRED)

set(MYSQL_PLUGIN_CONFIG_INCLUDE_DIR "${CMAKE_CURRENT_BINARY_DIR}/config_headers")
include_directories("${SDK_INCLUDE_DIR}" "${MYSQL_PLUGIN_CONFIG_INCLUDE_DIR}")

set(CMAKE_DEBUG_POSTFIX "_d")

configure_file(
	"../a_mysql.inc.in"
	"a_mysql.inc"
	@ONLY
)

configure_file(
	"version.hpp.in"
	"${MYSQL_PLUGIN_CONFIG_INCLUDE_DIR}/version.hpp"
	@ONLY
)


add_plugin(mysql 
	${SDK_INCLUDE_DIR}/amxplugin.cpp
	${SDK_INCLUDE_DIR}/amxplugin2.cpp
	${SDK_INCLUDE_DIR}/amx/getch.c
	CCallback.cpp
	CConnection.cpp
	CDispatcher.cpp
	CHandle.cpp
	CLog.cpp
	COptions.cpp
	COrm.cpp
	CQuery.cpp
	CResult.cpp
	CScripting.cpp
	main.cpp
	misc.cpp
	plugin.def
)

if(MSVC)
	add_definitions(-D_CRT_SECURE_NO_WARNINGS -DNOMINMAX)
endif()
#--TODO: remove this DLL-interface
add_definitions(-DDLL_EXPORTS) 

if(NOT MSVC) #we have auto-linking in Visual Studio
	target_link_libraries(mysql
		"${Boost_ATOMIC_LIBRARY}"
		"${Boost_CHRONO_LIBRARY}"
		"${Boost_DATE_TIME_LIBRARY}"
		"${Boost_FILESYSTEM_LIBRARY}"
		"${Boost_SYSTEM_LIBRARY}"
		"${Boost_THREAD_LIBRARY}"
	)
endif()

if(BUILD_STATIC)
	target_link_libraries(mysql "${MYSQLCAPI_LIBRARY_STATIC}")
else()
	target_link_libraries(mysql "${MYSQLCAPI_LIBRARY}")
endif()

install(TARGETS mysql DESTINATION "plugins/")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/a_mysql.inc" DESTINATION "pawno/include/")
if(WIN32 AND MYSQLCAPI_LIBRARY_SHARED)
	install(FILES "${MYSQLCAPI_LIBRARY_SHARED}" DESTINATION "./")
endif()

set(CPACK_PACKAGE_VERSION ${MYSQL_PLUGIN_VERSION})
if(WIN32)
	set(CPACK_GENERATOR ZIP)
else()
	set(CPACK_GENERATOR TGZ)
endif()

include(CPack)