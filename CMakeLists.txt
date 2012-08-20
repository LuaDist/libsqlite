# Copyright (C) 2007-2011 LuaDist.
# Created by Peter Kapec
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.

# IMPORTANT !!!! Files opcode.h, opcode.c, parse.h, parse.c were manualy generated...

# Project
project ( sqlite C )
cmake_minimum_required ( VERSION 2.8 )
include ( cmake/dist.cmake )

# Macros
macro ( ADD_PREFIX prefix rootlist )
set ( outlist )
foreach ( root ${${rootlist}} )
list ( APPEND outlist ${prefix}${root} )
endforeach ( root )
set ( ${rootlist} ${outlist} )
endmacro ( ADD_PREFIX )

# SQLite configuration
include ( CheckFunctionExists )

check_function_exists ( "usleep" HAVE_USLEEP )
if ( HAVE_USLEEP )
  add_definitions ( -DHAVE_USLEEP=1 )
endif ( )

find_package ( Threads )
if ( Threads_FOUND )
  if ( CMAKE_USE_PTHREADS_INIT )
    add_definitions ( -DTHREADSAFE=1 )
    set ( EXTRA_LIBS ${CMAKE_THREAD_LIBS_INIT} )
  endif ( )
endif ( )

#    Leave MEMORY_DEBUG undefined for maximum speed.  Use MEMORY_DEBUG=1
#    to check for memory leaks.  Use MEMORY_DEBUG=2 to print a log of all
#    malloc()s and free()s in order to track down memory leaks.
#    
# ADD_DEFINITIONS(-DMEMORY_DEBUG=1)
#    SQLite uses some expensive assert() statements in the inner loop.
#    You can make the library go almost twice as fast if you compile
#    with -DNDEBUG=1
add_definitions ( -DNDEBUG=1 )

find_package ( Readline )
if ( READLINE_FOUND )
  include_directories ( ${READLINE_INCLUDE_DIRS} )
endif ( )

#### Should the database engine assume text is coded as UTF-8 or iso8859?
#
# ENCODING  = UTF8 or ISO8859
add_definitions ( -DENCODING=ISO8859 )

include ( CheckTypeSize )
check_type_size ( "char*" SQLITE_PTR_SZ VARIABLE )
configure_file ( ${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h )
include_directories ( ${CMAKE_CURRENT_BINARY_DIR} )
set ( VERS "2.8.17" )
set ( ENCODING "ISO8859" )
configure_file ( ${CMAKE_CURRENT_SOURCE_DIR}/sqlite.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/sqlite.h )

# SQLite Library
include_directories ( . src )

set ( SQLITE_SRCS attach.c auth.c btree.c btree_rb.c build.c copy.c date.c delete.c 
  expr.c func.c hash.c insert.c main.c opcodes.c os.c pager.c parse.c pragma.c printf.c 
  random.c select.c table.c tokenize.c trigger.c update.c util.c vacuum.c vdbe.c 
  vdbeaux.c where.c )
add_prefix ( "src/" SQLITE_SRCS )

add_library ( sqlite ${SQLITE_SRCS} sqlite.def )
if ( CMAKE_USE_PTHREADS_INIT )
  target_link_libraries ( sqlite ${EXTRA_LIBS} )
endif ( )

install ( TARGETS sqlite RUNTIME DESTINATION ${INSTALL_BIN} LIBRARY DESTINATION ${INSTALL_LIB} 
  ARCHIVE DESTINATION ${INSTALL_LIB} )

# SQLite client
add_executable ( sqlite-app src/shell.c )
set_target_properties ( sqlite-app PROPERTIES OUTPUT_NAME sqlite CLEAN_DIRECT_OUTPUT 
  1 )
target_link_libraries ( sqlite-app sqlite )
if ( READLINE_LIBRARIES )
  target_link_libraries ( sqlite-app ${READLINE_LIBRARIES} )
endif ( )
install_executable ( sqlite-app )
# Install Headers
install_header ( ${CMAKE_CURRENT_BINARY_DIR}/sqlite.h )
