# Copyright (C) 2007-2011 LuaDist.
# Created by Peter Kapec
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.

# IMPORTANT !!!! Files opcode.h, opcode.c, parse.h, parse.c were manualy generated...

# Project
	PROJECT(sqlite C)
	CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
	INCLUDE(dist.cmake)

# Macros
	MACRO(ADD_PREFIX prefix rootlist)
	  SET(outlist )
	  FOREACH(root ${${rootlist}})
		LIST(APPEND outlist ${prefix}${root})
	  ENDFOREACH(root)
	  SET(${rootlist} ${outlist})
	ENDMACRO(ADD_PREFIX)

# SQLite configuration
	INCLUDE(CheckFunctionExists)

	CHECK_FUNCTION_EXISTS("usleep" HAVE_USLEEP)
	IF(HAVE_USLEEP)
		ADD_DEFINITIONS(-DHAVE_USLEEP=1)
	ENDIF()

	FIND_PACKAGE(Threads)
	IF(Threads_FOUND)
		IF(CMAKE_USE_PTHREADS_INIT)
			ADD_DEFINITIONS(-DTHREADSAFE=1)
			SET(EXTRA_LIBS ${CMAKE_THREAD_LIBS_INIT})
		ENDIF()
	ENDIF()

	#    Leave MEMORY_DEBUG undefined for maximum speed.  Use MEMORY_DEBUG=1
	#    to check for memory leaks.  Use MEMORY_DEBUG=2 to print a log of all
	#    malloc()s and free()s in order to track down memory leaks.
	#    
	# ADD_DEFINITIONS(-DMEMORY_DEBUG=1)
	
	#    SQLite uses some expensive assert() statements in the inner loop.
	#    You can make the library go almost twice as fast if you compile
	#    with -DNDEBUG=1
	ADD_DEFINITIONS(-DNDEBUG=1)

	FIND_PACKAGE(Readline)
	IF(READLINE_FOUND)
		INCLUDE_DIRECTORIES(${READLINE_INCLUDE_DIRS})
	ENDIF()

	#### Should the database engine assume text is coded as UTF-8 or iso8859?
	#
	# ENCODING  = UTF8 or ISO8859
	ADD_DEFINITIONS(-DENCODING=ISO8859)


	INCLUDE(CheckTypeSize)
	CHECK_TYPE_SIZE("char*" SQLITE_PTR_SZ VARIABLE)
	CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h)
	INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR})
		
	SET(VERS "2.8.17")
	SET(ENCODING "ISO8859")
	CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/sqlite.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/sqlite.h)

# SQLite Library
	INCLUDE_DIRECTORIES(. src)

	SET(SQLITE_SRCS 
		attach.c auth.c btree.c btree_rb.c build.c copy.c date.c delete.c 
		expr.c func.c hash.c insert.c 
		main.c opcodes.c os.c pager.c parse.c pragma.c printf.c random.c 
		select.c table.c tokenize.c trigger.c update.c util.c 
		vacuum.c vdbe.c vdbeaux.c where.c 
		)
	ADD_PREFIX("src/" SQLITE_SRCS)


	ADD_LIBRARY(sqlite SHARED ${SQLITE_SRCS}  sqlite.def)
	IF(CMAKE_USE_PTHREADS_INIT)	
		TARGET_LINK_LIBRARIES(sqlite ${EXTRA_LIBS})
	ENDIF()

	INSTALL(TARGETS sqlite RUNTIME DESTINATION ${INSTALL_BIN} LIBRARY DESTINATION ${INSTALL_LIB} ARCHIVE DESTINATION ${INSTALL_LIB})

# SQLite client
	ADD_EXECUTABLE(sqlite-app src/shell.c)
	SET_TARGET_PROPERTIES(sqlite-app PROPERTIES OUTPUT_NAME sqlite CLEAN_DIRECT_OUTPUT 1 )	
	TARGET_LINK_LIBRARIES(sqlite-app sqlite)	
	IF(READLINE_LIBRARIES)
		TARGET_LINK_LIBRARIES(sqlite-app ${READLINE_LIBRARIES})	
	ENDIF()
	INSTALL(TARGETS sqlite-app RUNTIME DESTINATION ${INSTALL_BIN} LIBRARY DESTINATION ${INSTALL_LIB} ARCHIVE DESTINATION ${INSTALL_LIB})
	
# Install Headers
	INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/sqlite.h DESTINATION ${INSTALL_INC})
	
