#*******************************************************************************
#  Copyright (c) 2015, 2020 logi.cals GmbH and others
#
#  All rights reserved. This program and the accompanying materials
#  are made available under the terms of the Eclipse Public License v2.0
#  and Eclipse Distribution License v1.0 which accompany this distribution.
#
#  The Eclipse Public License is available at
#     https://www.eclipse.org/legal/epl-2.0/
#  and the Eclipse Distribution License is available at
#    http://www.eclipse.org/org/documents/edl-v10.php.
#
#  Contributors:
#     Rainer Poisel - initial version
#     Ian Craggs (IBM Corp.) - merge master
#     Ian Craggs - update for MQTTV5 support
#*******************************************************************************/

# Note: on OS X you should install XCode and the associated command-line tools

## compilation/linkage settings

CONFIGURE_FILE(VersionInfo.h.in
    ${PROJECT_BINARY_DIR}/VersionInfo.h
    @ONLY
    )

SET(common_src
  MQTTTime.c
  MQTTProtocolClient.c
  Clients.c
  utf-8.c
  MQTTPacket.c
  MQTTPacketOut.c
  Messages.c
  Tree.c
  Socket.c
  Log.c
  MQTTPersistence.c
  Thread.c
  MQTTProtocolOut.c
  MQTTPersistenceDefault.c
  SocketBuffer.c
  LinkedList.c
  MQTTProperties.c
  MQTTReasonCodes.c
  Base64.c
  SHA1.c
  WebSocket.c
  Proxy.c
)

IF (NOT PAHO_HIGH_PERFORMANCE)
  SET(common_src ${common_src}
    StackTrace.c
    Heap.c
    )
ENDIF()

SET(mqtt_version_src MQTTVersion.c)
IF (WIN32)
    INCLUDE(Windows-MyExeVersion)
    generate_product_exe_version(PAHO_MQTTVERSION_VERSION_FILES
        NAME "MQTTVersion - print the version strings of an MQTT client library"
        BUNDLE "${PROJECT_DESCRIPTION}"
        ICON ${PROJECT_SOURCE_DIR}/mqtt.ico
        VERSION_MAJOR ${PAHO_VERSION_MAJOR}
        VERSION_MINOR ${PAHO_VERSION_MINOR}
        VERSION_PATCH ${PAHO_VERSION_PATCH}
        VERSION_REVISION ${MY_REVISION}
        ORIGINAL_FILENAME "MQTTVersion.exe")
    SET(mqtt_version_src ${mqtt_version_src} ${PAHO_MQTTVERSION_VERSION_FILES})
    SET(LIBS_SYSTEM ws2_32 crypt32 RpcRT4 advapi32)
ELSEIF (UNIX)
    IF(CMAKE_SYSTEM_NAME MATCHES "Linux")
        SET(LIBS_SYSTEM c dl pthread rt)
        # anl is only available with glibc so check if it is found before using
        # it or build will fail on uclibc or musl
        FIND_LIBRARY(LIB_ANL anl)
        IF(LIB_ANL)
            SET(LIBS_SYSTEM "${LIBS_SYSTEM}" anl)
        ENDIF()
        ADD_DEFINITIONS(-D_GNU_SOURCE -fvisibility=hidden)
    ELSEIF (CMAKE_SYSTEM_NAME MATCHES "Android")
        SET(LIBS_SYSTEM c dl)
    ELSEIF (CMAKE_SYSTEM_NAME MATCHES "FreeBSD")
        SET(LIBS_SYSTEM compat pthread)
    ELSE()
        SET(LIBS_SYSTEM c pthread)
    ENDIF()
ENDIF()

IF (PAHO_BUILD_SHARED)
# common compilation for libpaho-mqtt3c and libpaho-mqtt3a
ADD_LIBRARY(common_obj OBJECT ${common_src})
SET_TARGET_PROPERTIES(common_obj PROPERTIES
    POSITION_INDEPENDENT_CODE ON
    COMPILE_DEFINITIONS "PAHO_MQTT_EXPORTS=1")
ADD_EXECUTABLE(MQTTVersion ${mqtt_version_src})
SET_TARGET_PROPERTIES(MQTTVersion PROPERTIES
    POSITION_INDEPENDENT_CODE ON
    COMPILE_DEFINITIONS "PAHO_MQTT_IMPORTS=1")
ENDIF()

IF (PAHO_BUILD_STATIC)    
ADD_LIBRARY(common_obj_static OBJECT ${common_src})
SET_TARGET_PROPERTIES(common_obj_static PROPERTIES
    POSITION_INDEPENDENT_CODE ON
    COMPILE_DEFINITIONS "PAHO_MQTT_STATIC=1")
ENDIF()

SET(mqtt_mqtt3c_src MQTTClient.c)
SET(mqtt_mqtt3a_src MQTTAsync.c MQTTAsyncUtils.c)
SET(mqtt_mqtt3cs_src MQTTClient.c SSLSocket.c)
SET(mqtt_mqtt3as_src MQTTAsync.c MQTTAsyncUtils.c SSLSocket.c)
IF(WIN32)
    INCLUDE(Windows-MyDllVersion)
    generate_product_dll_version(PAHO_MQTT3C_VERSION_FILES
        NAME "Eclipse Paho Synchronous MQTT C Client Library"
        BUNDLE "${PROJECT_DESCRIPTION}"
        VERSION_MAJOR ${PAHO_VERSION_MAJOR}
        VERSION_MINOR ${PAHO_VERSION_MINOR}
        VERSION_PATCH ${PAHO_VERSION_PATCH}
        VERSION_REVISION ${MY_REVISION}
        ORIGINAL_FILENAME "libpaho-mqtt3c.dll"
    )
    SET(mqtt_mqtt3c_src ${mqtt_mqtt3c_src} ${PAHO_MQTT3C_VERSION_FILES})
    generate_product_dll_version(PAHO_MQTT3A_VERSION_FILES
        NAME "Eclipse Paho Asynchronous MQTT C Client Library"
        BUNDLE "${PROJECT_DESCRIPTION}"
        VERSION_MAJOR ${PAHO_VERSION_MAJOR}
        VERSION_MINOR ${PAHO_VERSION_MINOR}
        VERSION_PATCH ${PAHO_VERSION_PATCH}
        VERSION_REVISION ${MY_REVISION}
        ORIGINAL_FILENAME "libpaho-mqtt3a.dll"
    )
    SET(mqtt_mqtt3a_src ${mqtt_mqtt3a_src} ${PAHO_MQTT3A_VERSION_FILES})
    generate_product_dll_version(PAHO_MQTT3CS_VERSION_FILES
        NAME "Eclipse Paho Synchronous MQTT C Client Library with SSL"
        BUNDLE "${PROJECT_DESCRIPTION}"
        VERSION_MAJOR ${PAHO_VERSION_MAJOR}
        VERSION_MINOR ${PAHO_VERSION_MINOR}
        VERSION_PATCH ${PAHO_VERSION_PATCH}
        VERSION_REVISION ${MY_REVISION}
        ORIGINAL_FILENAME "libpaho-mqtt3cs.dll"
    )
    SET(mqtt_mqtt3cs_src ${mqtt_mqtt3cs_src} ${PAHO_MQTT3CS_VERSION_FILES})
    generate_product_dll_version(PAHO_MQTT3AS_VERSION_FILES
        NAME "Eclipse Paho Asynchronous MQTT C Client Library with SSL"
        BUNDLE "${PROJECT_DESCRIPTION}"
        VERSION_MAJOR ${PAHO_VERSION_MAJOR}
        VERSION_MINOR ${PAHO_VERSION_MINOR}
        VERSION_PATCH ${PAHO_VERSION_PATCH}
        VERSION_REVISION ${MY_REVISION}
        ORIGINAL_FILENAME "libpaho-mqtt3as.dll"
    )
    SET(mqtt_mqtt3as_src ${mqtt_mqtt3as_src} ${PAHO_MQTT3AS_VERSION_FILES})
endif()

IF (PAHO_BUILD_SHARED)
    ADD_LIBRARY(paho-mqtt3c SHARED $<TARGET_OBJECTS:common_obj> ${mqtt_mqtt3c_src})
    ADD_LIBRARY(paho-mqtt3a SHARED $<TARGET_OBJECTS:common_obj> ${mqtt_mqtt3a_src})
    
    TARGET_LINK_LIBRARIES(paho-mqtt3c ${LIBS_SYSTEM})
    TARGET_LINK_LIBRARIES(paho-mqtt3a ${LIBS_SYSTEM})
    TARGET_LINK_LIBRARIES(MQTTVersion paho-mqtt3a paho-mqtt3c ${LIBS_SYSTEM})
    
    SET_TARGET_PROPERTIES(
        paho-mqtt3c paho-mqtt3a PROPERTIES
        VERSION ${CLIENT_VERSION}
        SOVERSION ${PAHO_VERSION_MAJOR}
        COMPILE_DEFINITIONS "PAHO_MQTT_EXPORTS=1")

    IF(${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
		SET(MQTTCLIENT_ENTRY_POINT _MQTTClient_init)
		SET(MQTTASYNC_ENTRY_POINT _MQTTAsync_init)
	ELSEIF (NOT WIN32)
		SET(MQTTCLIENT_ENTRY_POINT MQTTClient_init)
		SET(MQTTASYNC_ENTRY_POINT MQTTAsync_init)
	ENDIF()

    IF (NOT WIN32)
        SET_TARGET_PROPERTIES(
            paho-mqtt3c PROPERTIES
            LINK_FLAGS "-Wl,-init,${MQTTCLIENT_ENTRY_POINT}")
        SET_TARGET_PROPERTIES(
            paho-mqtt3a PROPERTIES
            LINK_FLAGS "-Wl,-init,${MQTTASYNC_ENTRY_POINT}")
    ENDIF()

    FOREACH(TARGET paho-mqtt3c paho-mqtt3a)
        TARGET_INCLUDE_DIRECTORIES(${TARGET}
            PUBLIC
                $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            PRIVATE
                ${PROJECT_BINARY_DIR})
    ENDFOREACH()
    
    IF (NOT PAHO_WITHOUT_INSTALL_LIBRARIES AND NOT PAHO_WITHOUT_INSTALL_ALL)
        INSTALL(TARGETS paho-mqtt3c paho-mqtt3a
            EXPORT eclipse-paho-mqtt-cTargets
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
    ENDIF()
    IF (NOT PAHO_WITHOUT_INSTALL_FILES AND NOT PAHO_WITHOUT_INSTALL_ALL)
        INSTALL(TARGETS MQTTVersion
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
    ENDIF()
ENDIF()

IF (PAHO_BUILD_STATIC)
    ADD_LIBRARY(paho-mqtt3c-static STATIC $<TARGET_OBJECTS:common_obj_static> ${mqtt_mqtt3c_src})
    ADD_LIBRARY(paho-mqtt3a-static STATIC $<TARGET_OBJECTS:common_obj_static> ${mqtt_mqtt3a_src})

    TARGET_LINK_LIBRARIES(paho-mqtt3c-static ${LIBS_SYSTEM})
    TARGET_LINK_LIBRARIES(paho-mqtt3a-static ${LIBS_SYSTEM})
    IF (NOT WIN32)
        SET_TARGET_PROPERTIES(paho-mqtt3c-static PROPERTIES OUTPUT_NAME paho-mqtt3c)
        SET_TARGET_PROPERTIES(paho-mqtt3a-static PROPERTIES OUTPUT_NAME paho-mqtt3a)
    ENDIF()       
    SET_TARGET_PROPERTIES(
        paho-mqtt3c-static paho-mqtt3a-static PROPERTIES
        VERSION ${CLIENT_VERSION}
        SOVERSION ${PAHO_VERSION_MAJOR}
        COMPILE_DEFINITIONS "PAHO_MQTT_STATIC=1")

    FOREACH(TARGET paho-mqtt3c-static paho-mqtt3a-static)
        TARGET_INCLUDE_DIRECTORIES(${TARGET}
            PUBLIC
                $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
            PRIVATE
                ${PROJECT_BINARY_DIR})
    ENDFOREACH()

    IF (NOT PAHO_WITHOUT_INSTALL_LIBRARIES AND NOT PAHO_WITHOUT_INSTALL_ALL)
        IF (NOT PAHO_BUILD_SHARED)
            INSTALL(TARGETS paho-mqtt3c-static paho-mqtt3a-static
                EXPORT eclipse-paho-mqtt-cTargets
                ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
                LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
                RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
        ELSE()
            INSTALL(TARGETS paho-mqtt3c-static paho-mqtt3a-static
                ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
        ENDIF()
    ENDIF()
ENDIF()

IF (NOT PAHO_WITHOUT_INSTALL_HEADERS AND NOT PAHO_WITHOUT_INSTALL_ALL)
    INSTALL(FILES MQTTAsync.h MQTTClient.h MQTTClientPersistence.h MQTTProperties.h MQTTReasonCodes.h MQTTSubscribeOpts.h MQTTExportDeclarations.h
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
ENDIF()

IF (PAHO_WITH_SSL)
    SET(OPENSSL_ROOT_DIR "" CACHE PATH "Directory containing OpenSSL libraries and includes")
    find_package(SiblingOpenSSL)
    IF (NOT OPENSSL_FOUND)
        find_package(OpenSSL REQUIRED)
    ENDIF()

    IF (PAHO_BUILD_SHARED)
        ## common compilation for libpaho-mqtt3cs and libpaho-mqtt3as
    	## Note: SSL libraries must be recompiled due ifdefs
    	ADD_LIBRARY(common_ssl_obj OBJECT ${common_src})
    	TARGET_INCLUDE_DIRECTORIES(common_ssl_obj PUBLIC ${OPENSSL_INCLUDE_DIR} ${OPENSSL_INCLUDE_DIRS})
    	SET_PROPERTY(TARGET common_ssl_obj PROPERTY	POSITION_INDEPENDENT_CODE ON)
    	SET_PROPERTY(TARGET common_ssl_obj PROPERTY COMPILE_DEFINITIONS "OPENSSL=1;PAHO_MQTT_EXPORTS=1")
    
        ADD_LIBRARY(paho-mqtt3cs SHARED $<TARGET_OBJECTS:common_ssl_obj> ${mqtt_mqtt3cs_src})
        ADD_LIBRARY(paho-mqtt3as SHARED $<TARGET_OBJECTS:common_ssl_obj> ${mqtt_mqtt3as_src})
    
        SET_TARGET_PROPERTIES(
            paho-mqtt3cs paho-mqtt3as PROPERTIES
            VERSION ${CLIENT_VERSION}
            SOVERSION ${PAHO_VERSION_MAJOR}
            COMPILE_DEFINITIONS "OPENSSL=1;PAHO_MQTT_EXPORTS=1")

        IF(${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
		    SET(MQTTCLIENT_ENTRY_POINT _MQTTClient_init)
		    SET(MQTTASYNC_ENTRY_POINT _MQTTAsync_init)
        ELSEIF (NOT WIN32)
            SET(MQTTCLIENT_ENTRY_POINT MQTTClient_init)
		    SET(MQTTASYNC_ENTRY_POINT MQTTAsync_init)
        ENDIF()

        IF (NOT WIN32)
            SET_TARGET_PROPERTIES(
	           paho-mqtt3cs PROPERTIES
	           LINK_FLAGS "-Wl,-init,${MQTTCLIENT_ENTRY_POINT}")
	        SET_TARGET_PROPERTIES(
	           paho-mqtt3as PROPERTIES
	           LINK_FLAGS "-Wl,-init,${MQTTASYNC_ENTRY_POINT}")
        ENDIF()

        FOREACH(TARGET paho-mqtt3cs paho-mqtt3as)
            TARGET_INCLUDE_DIRECTORIES(${TARGET}
                PUBLIC
                    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
                    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                PRIVATE
                    ${PROJECT_BINARY_DIR})
            TARGET_LINK_LIBRARIES(${TARGET}
                PUBLIC
                    OpenSSL::SSL OpenSSL::Crypto ${LIBS_SYSTEM})
        ENDFOREACH()
        IF (NOT PAHO_WITHOUT_INSTALL_LIBRARIES AND NOT PAHO_WITHOUT_INSTALL_ALL)
            INSTALL(TARGETS paho-mqtt3cs paho-mqtt3as
                #EXPORT eclipse-paho-mqtt-cTargets
                ARCHIVE DESTINATION  ${CMAKE_INSTALL_LIBDIR}
                LIBRARY DESTINATION  ${CMAKE_INSTALL_LIBDIR}
                RUNTIME DESTINATION  ${CMAKE_INSTALL_BINDIR})
        ENDIF()
    ENDIF()

    IF (PAHO_BUILD_STATIC)
        ## common compilation for libpaho-mqtt3cs and libpaho-mqtt3as
    	## Note: SSL libraries must be recompiled due ifdefs
    	ADD_LIBRARY(common_ssl_obj_static OBJECT ${common_src})
    	TARGET_INCLUDE_DIRECTORIES(common_ssl_obj_static PUBLIC ${OPENSSL_INCLUDE_DIR} ${OPENSSL_INCLUDE_DIRS})
    	SET_PROPERTY(TARGET common_ssl_obj_static PROPERTY POSITION_INDEPENDENT_CODE ON)
    	SET_PROPERTY(TARGET common_ssl_obj_static PROPERTY COMPILE_DEFINITIONS "OPENSSL=1;PAHO_MQTT_STATIC=1")
    
        ADD_LIBRARY(paho-mqtt3cs-static STATIC $<TARGET_OBJECTS:common_ssl_obj_static> ${mqtt_mqtt3cs_src})
        ADD_LIBRARY(paho-mqtt3as-static STATIC $<TARGET_OBJECTS:common_ssl_obj_static> ${mqtt_mqtt3as_src})

        SET_TARGET_PROPERTIES(
            paho-mqtt3cs-static paho-mqtt3as-static PROPERTIES
            VERSION ${CLIENT_VERSION}
            SOVERSION ${PAHO_VERSION_MAJOR}
            COMPILE_DEFINITIONS "OPENSSL=1;PAHO_MQTT_STATIC=1")
        IF (NOT WIN32)
            SET_TARGET_PROPERTIES(paho-mqtt3cs-static PROPERTIES OUTPUT_NAME paho-mqtt3cs)
            SET_TARGET_PROPERTIES(paho-mqtt3as-static PROPERTIES OUTPUT_NAME paho-mqtt3as)
        ENDIF()

	    IF(${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
			SET(MQTTCLIENT_ENTRY_POINT _MQTTClient_init)
			SET(MQTTASYNC_ENTRY_POINT _MQTTAsync_init)
		ELSEIF (NOT WIN32)
			SET(MQTTCLIENT_ENTRY_POINT MQTTClient_init)
			SET(MQTTASYNC_ENTRY_POINT MQTTAsync_init)
		ENDIF()

		IF (NOT WIN32)
		    SET_TARGET_PROPERTIES(
	            paho-mqtt3cs-static PROPERTIES
	            LINK_FLAGS "-Wl,-init,${MQTTCLIENT_ENTRY_POINT}")
	        SET_TARGET_PROPERTIES(
	            paho-mqtt3as-static PROPERTIES
	            LINK_FLAGS "-Wl,-init,${MQTTASYNC_ENTRY_POINT}")
	    ENDIF()

        IF (NOT PAHO_WITHOUT_INSTALL_LIBRARIES AND NOT PAHO_WITHOUT_INSTALL_ALL)
            IF (NOT PAHO_BUILD_SHARED)
                INSTALL(TARGETS paho-mqtt3cs-static paho-mqtt3as-static
                    #EXPORT eclipse-paho-mqtt-cTargets
                    ARCHIVE DESTINATION  ${CMAKE_INSTALL_LIBDIR}
                    LIBRARY DESTINATION  ${CMAKE_INSTALL_LIBDIR}
                    RUNTIME DESTINATION  ${CMAKE_INSTALL_BINDIR})
            ELSE()
                INSTALL(TARGETS paho-mqtt3cs-static paho-mqtt3as-static
                    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
            ENDIF()
        ENDIF()
        FOREACH(TARGET paho-mqtt3cs-static paho-mqtt3as-static)
            TARGET_INCLUDE_DIRECTORIES(${TARGET}
                PUBLIC
                    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
                    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                PRIVATE
                    ${PROJECT_BINARY_DIR})
            TARGET_LINK_LIBRARIES(${TARGET}
                PUBLIC
                    OpenSSL::SSL OpenSSL::Crypto ${LIBS_SYSTEM})
        ENDFOREACH()
    ENDIF()
ENDIF()

IF (NOT PAHO_WITHOUT_INSTALL_FILES AND NOT PAHO_WITHOUT_INSTALL_LIBRARIES AND NOT PAHO_WITHOUT_INSTALL_ALL)
    INSTALL(EXPORT eclipse-paho-mqtt-cTargets
        FILE eclipse-paho-mqtt-cConfig.cmake
        NAMESPACE eclipse-paho-mqtt-c::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/eclipse-paho-mqtt-c)
ENDIF()

INCLUDE(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE("eclipse-paho-mqtt-cConfigVersion.cmake"
    VERSION ${CLIENT_VERSION}
    COMPATIBILITY SameMajorVersion)
IF (NOT PAHO_WITHOUT_INSTALL_FILES AND NOT PAHO_WITHOUT_INSTALL_ALL)
    INSTALL(FILES
        "${CMAKE_CURRENT_BINARY_DIR}/eclipse-paho-mqtt-cConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/eclipse-paho-mqtt-c)
ENDIF()

# Base64 test
ADD_EXECUTABLE( Base64Test EXCLUDE_FROM_ALL Base64.c Base64.h )
TARGET_COMPILE_DEFINITIONS( Base64Test PUBLIC "-DBASE64_TEST" )
IF (PAHO_WITH_SSL)
	ADD_EXECUTABLE( Base64TestOpenSSL EXCLUDE_FROM_ALL Base64.c Base64.h )
	TARGET_LINK_LIBRARIES( Base64TestOpenSSL OpenSSL::SSL OpenSSL::Crypto)
	TARGET_COMPILE_DEFINITIONS( Base64TestOpenSSL PUBLIC "-DBASE64_TEST -DOPENSSL=1" )
ENDIF (PAHO_WITH_SSL)

# SHA1 test
ADD_EXECUTABLE( Sha1Test EXCLUDE_FROM_ALL SHA1.c SHA1.h )
TARGET_COMPILE_DEFINITIONS( Sha1Test PUBLIC "-DSHA1_TEST" )
IF (PAHO_WITH_SSL)
	ADD_EXECUTABLE( Sha1TestOpenSSL EXCLUDE_FROM_ALL SHA1.c SHA1.h )
	TARGET_LINK_LIBRARIES( Sha1TestOpenSSL OpenSSL::SSL OpenSSL::Crypto)
	TARGET_COMPILE_DEFINITIONS( Sha1TestOpenSSL PUBLIC "-DSHA1_TEST -DOPENSSL=1" )
ENDIF (PAHO_WITH_SSL)
