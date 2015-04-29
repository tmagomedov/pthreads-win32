cmake_minimum_required(VERSION 2.8.10)

project(pthreads-win32)

option(PTHREADS_BUILD_STATIC "If enabled, static library is built. Default is shared library." OFF)
option(PTHREADS_EXPORT       "Export target for using inside other CMake projects." OFF)

set(PTHREAD_API
	pthread.h
	sched.h
	semaphore.h)

source_group(PTHREAD_API FILES ${PTHREAD_API})

set(PTHREAD_SOURCES pthread.c)

set(PTHREAD_RESOURCES
	version.rc)
set(PTHREAD_HEADERS
	config.h
	implement.h
	need_errno.h)

set(PTHREAD_STATIC_SOURCES pthread.c)

if(PTHREADS_BUILD_STATIC)
	set(SOURCES ${PTHREAD_STATIC_SOURCES})
	set(PTHREADS_LIBTYPE STATIC)
else()
	set(PTHREADS_LIBTYPE SHARED)
endif()

if( CMAKE_SIZEOF_VOID_P EQUAL 8 )
	add_definitions( -DPTW32_ARCH="x64" )
else( CMAKE_SIZEOF_VOID_P EQUAL 8 )
	add_definitions( -DPTW32_ARCH="x86" )
endif( CMAKE_SIZEOF_VOID_P EQUAL 8 )

if(MSVC)
	add_definitions(/DPTW32_RC_MSC)
	set(PTHREADS_COMPILER V)
elseif(CMAKE_COMPILER_IS_GNUCXX)
	set(PTHREADS_COMPILER G)
else()
	message("WARNING: pthreads-win32 doesn't recognize your compiler!")
endif()

if(PTHREADS_BUILD_CPP)
	set(PTHREADS_EXCEPTION_SCHEME CE)
	add_definitions(/__CLEANUP_CXX)
	message("Building pthreads-win32 with C++ exception handling.")
elseif(PTHREADS_BUILD_SEH)
	set(PTHREADS_EXCEPTION_SCHEME SE)
	add_definitions(/__CLEANUP_SEH)
	message("Building pthreads-win32 with Microsoft SEH exception handling.")
else()
	# defaults to C - setjmp/longjmp
	set(PTHREADS_EXCEPTION_SCHEME C)
	message("Building pthreads-win32 with C setjmp/longjmp (default/recommended exception scheme).")
endif()
set(PTHREADS_COMPATIBILITY_VERSION 2)
if(PTHREADS_EXPORT)
	set(PTHREADS_LIBRARY "pthreads") # use single fixed exported target name
else()
	set(PTHREADS_LIBRARY "pthreads${PTHREADS_COMPILER}${PTHREADS_EXCEPTION_SCHEME}${PTHREADS_COMPATIBILITY_VERSION}")
endif()
message("Library name of pthreads-win32: ${PTHREADS_LIBRARY}.")
set(CMAKE_DEBUG_POSTFIX d)

add_definitions(/DHAVE_PTW32_CONFIG_H)

include_directories(.)
add_library(${PTHREADS_LIBRARY} ${PTHREADS_LIBTYPE} ${PTHREAD_SOURCES} ${PTHREAD_API} ${PTHREAD_RESOURCES})
set_target_properties(${PTHREADS_LIBRARY} PROPERTIES PUBLIC_HEADER "${PTHREAD_API}")
if(PTHREADS_EXPORT)
	export(TARGETS ${PTHREADS_LIBRARY} FILE pthreads-win32-exports.cmake)
else()
	set(PTHREADS_INSTALL YES)
	set(PTHREADS_BINDIR bin)
	set(PTHREADS_BINCOMPONENT runtime)
	set(PTHREADS_LIBDIR lib)
	set(PTHREADS_LIBCOMPONENT dev)
	set(PTHREADS_INCLUDEDIR include)
	set(PTHREADS_INCLUDECOMPONENT dev)

	install(TARGETS ${PTHREADS_LIBRARY}
		RUNTIME DESTINATION ${PTHREADS_BINDIR} COMPONENT ${PTHREADS_BINCOMPONENT}
		LIBRARY DESTINATION ${PTHREADS_LIBDIR} COMPONENT ${PTHREADS_LIBCOMPONENT}
		ARCHIVE DESTINATION ${PTHREADS_LIBDIR} COMPONENT ${PTHREADS_LIBCOMPONENT}
		PUBLIC_HEADER DESTINATION ${PTHREADS_INCLUDEDIR} COMPONENT ${PTHREADS_INCLUDECOMPONENT})
endif()
