file(STRINGS "configure.ac" CONFIGURE_AC_LINES)
foreach(line ${CONFIGURE_AC_LINES})
	if(line MATCHES [[^m4_define\(\[(MAJOR_VERSION|MINOR_VERSION|MICRO_VERSION)\],[ \t]*([0-9]+)\)$]])
		set(LIBXML_${CMAKE_MATCH_1} ${CMAKE_MATCH_2})
	elseif(line MATCHES "^(LIBXML_MAJOR_VERSION|LIBXML_MINOR_VERSION|LIBXML_MICRO_VERSION)=([0-9]+)$")
		set(${CMAKE_MATCH_1} ${CMAKE_MATCH_2})
	endif()
endforeach()
set(VERSION "${LIBXML_MAJOR_VERSION}.${LIBXML_MINOR_VERSION}.${LIBXML_MICRO_VERSION}")

include(CheckCSourceCompiles)
include(CheckFunctionExists)
include(CheckIncludeFiles)
include(CheckLibraryExists)
include(CheckStructHasMember)
include(CheckSymbolExists)
include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

option(BUILD_SHARED_LIBS "Build shared libraries" ON)
set(LIBXML2_WITH_AUTOMATA ON)
option(LIBXML2_WITH_C14N "Add the Canonicalization support" ON)
option(LIBXML2_WITH_CATALOG "Add the Catalog support" ON)
option(LIBXML2_WITH_DEBUG "Add the debugging module" ON)
option(LIBXML2_WITH_DOCB "Add Docbook SGML support" ON)
set(LIBXML2_WITH_EXPR ON)
option(LIBXML2_WITH_FTP "Add the FTP support" OFF)
option(LIBXML2_WITH_HTML "Add the HTML support" ON)
option(LIBXML2_WITH_HTTP "Add the HTTP support" OFF)
option(LIBXML2_WITH_ICONV "Add ICONV support" OFF)
option(LIBXML2_WITH_ICU "Add ICU support" OFF)
option(LIBXML2_WITH_ISO8859X "Add ISO8859X support if no iconv" ON)
option(LIBXML2_WITH_LEGACY "Add deprecated APIs for compatibility" ON)
option(LIBXML2_WITH_LZMA "Use liblzma" OFF)
option(LIBXML2_WITH_MEM_DEBUG "Add the memory debugging module" OFF)
option(LIBXML2_WITH_MODULES "Add the dynamic modules support" ON)
option(LIBXML2_WITH_OUTPUT "Add the serialization support" ON)
option(LIBXML2_WITH_PATTERN "Add the xmlPattern selection interface" ON)
option(LIBXML2_WITH_PROGRAMS "Build programs" OFF)
option(LIBXML2_WITH_PUSH "Add the PUSH parser interfaces" ON)
option(LIBXML2_WITH_PYTHON "Build Python bindings" OFF)
option(LIBXML2_WITH_READER "Add the xmlReader parsing interface" ON)
option(LIBXML2_WITH_REGEXPS "Add Regular Expressions support" ON)
option(LIBXML2_WITH_RUN_DEBUG "Add the runtime debugging module" OFF)
option(LIBXML2_WITH_SAX1 "Add the older SAX1 interface" ON)
option(LIBXML2_WITH_SCHEMAS "Add Relax-NG and Schemas support" ON)
option(LIBXML2_WITH_SCHEMATRON "Add Schematron support" ON)
option(LIBXML2_WITH_TESTS "Build tests" OFF)
option(LIBXML2_WITH_THREADS "Add multithread support" ON)
option(LIBXML2_WITH_THREAD_ALLOC "Add per-thread memory" OFF)
option(LIBXML2_WITH_TREE "Add the DOM like tree manipulation APIs" ON)
set(LIBXML2_WITH_TRIO OFF)
set(LIBXML2_WITH_UNICODE ON)
option(LIBXML2_WITH_VALID "Add the DTD validation support" ON)
option(LIBXML2_WITH_WRITER "Add the xmlWriter saving interface" ON)
option(LIBXML2_WITH_XINCLUDE "Add the XInclude support" ON)
option(LIBXML2_WITH_XPATH "Add the XPATH support" ON)
option(LIBXML2_WITH_XPTR "Add the XPointer support" ON)
option(LIBXML2_WITH_ZLIB "Use libz" ON)
set(LIBXML2_XMLCONF_WORKING_DIR ${CMAKE_CURRENT_BINARY_DIR} CACHE PATH "Working directory for XML Conformance Test Suite")

if(LIBXML2_WITH_ICONV)
	find_package(Iconv REQUIRED)
endif()

if(LIBXML2_WITH_ICU)
	find_package(ICU REQUIRED COMPONENTS data i18n uc)
endif()

if(LIBXML2_WITH_LZMA)
	find_package(LibLZMA REQUIRED)
endif()

if(LIBXML2_WITH_PYTHON)
	check_include_files(unistd.h HAVE_UNISTD_H)
	check_symbol_exists(F_GETFL fcntl.h HAVE_F_GETFL)
	if(HAVE_UNISTD_H AND HAVE_F_GETFL)
		find_package(Python COMPONENTS Interpreter Development REQUIRED)
	else()
		find_package(Python2 COMPONENTS Interpreter Development REQUIRED)
		add_library(Python::Python ALIAS Python2::Python)
		set(Python_EXECUTABLE ${Python2_EXECUTABLE})
		set(Python_SITEARCH ${Python2_SITEARCH})
	endif()
	set(LIBXML2_PYTHON_INSTALL_DIR ${Python_SITEARCH} CACHE PATH "Python bindings install directory")
endif()

if(LIBXML2_WITH_THREADS)
	find_package(Threads REQUIRED)
endif()

set(DEFS)
if(LIBXML2_WITH_ZLIB)
	__cc_find(zlib_static)
	__assert_target(zlib_static)
	list(APPEND CMAKE_REQUIRED_LIBRARIES zlib_static)
endif()

foreach(VARIABLE IN ITEMS WITH_AUTOMATA WITH_C14N WITH_CATALOG WITH_DEBUG WITH_DOCB WITH_EXPR WITH_FTP WITH_HTML WITH_HTTP WITH_ICONV WITH_ICU WITH_ISO8859X WITH_LEGACY WITH_LZMA WITH_MEM_DEBUG WITH_MODULES WITH_OUTPUT WITH_PATTERN WITH_PUSH WITH_READER WITH_REGEXPS WITH_RUN_DEBUG WITH_SAX1 WITH_SCHEMAS WITH_SCHEMATRON WITH_THREADS WITH_THREAD_ALLOC WITH_TREE WITH_TRIO WITH_UNICODE WITH_VALID WITH_WRITER WITH_XINCLUDE WITH_XPATH WITH_XPTR WITH_ZLIB)
	if(LIBXML2_${VARIABLE})
		set(${VARIABLE} 1)
	else()
		set(${VARIABLE} 0)
	endif()
endforeach()

set(LIBXML_VERSION ${LIBXML_MAJOR_VERSION}0${LIBXML_MINOR_VERSION}0${LIBXML_MICRO_VERSION})
set(LIBXML_VERSION_STRING "${LIBXML_VERSION}")
set(LIBXML_VERSION_EXTRA "")
set(LIBXML_VERSION_NUMBER ${LIBXML_VERSION})

set(MODULE_EXTENSION "${CMAKE_SHARED_LIBRARY_SUFFIX}")

if(LIBLZMA_FOUND)
	list(APPEND CMAKE_REQUIRED_LIBRARIES LibLZMA::LibLZMA)
endif()

if(Threads_FOUND)
	list(APPEND CMAKE_REQUIRED_LIBRARIES Threads::Threads)
endif()

if(MSVC)
	configure_file(include/win32config.h config.h COPYONLY)
else()
	check_c_source_compiles("
		void __attribute__((destructor))
		f(void) {}
		int main(void) { return 0; }
	" ATTRIBUTE_DESTRUCTOR)
	check_c_source_compiles("
		#include <netdb.h>
		int main() { (void) gethostbyname((const char*) \"\"); return 0; }
	" GETHOSTBYNAME_ARG_CAST_CONST)
	if(NOT GETHOSTBYNAME_ARG_CAST_CONST)
		set(GETHOSTBYNAME_ARG_CAST "(char *)")
	else()
		set(GETHOSTBYNAME_ARG_CAST "/**/")
	endif()
	check_include_files(arpa/inet.h HAVE_ARPA_INET_H)
	check_include_files(arpa/nameser.h HAVE_ARPA_NAMESER_H)
	check_struct_has_member("struct sockaddr_storage" ss_family "sys/socket.h;sys/types.h" HAVE_SS_FAMILY)
	check_struct_has_member("struct sockaddr_storage" __ss_family "sys/socket.h;sys/types.h" HAVE_BROKEN_SS_FAMILY)
	if(HAVE_BROKEN_SS_FAMILY)
		set(ss_family __ss_family)
	endif()
	check_function_exists(class HAVE_CLASS)
	check_include_files(ctype.h HAVE_CTYPE_H)
	check_include_files(dirent.h HAVE_DIRENT_H)
	check_include_files(dlfcn.h HAVE_DLFCN_H)
	check_library_exists(dl dlopen "" HAVE_DLOPEN)
	check_include_files(dl.h HAVE_DL_H)
	check_include_files(errno.h HAVE_ERRNO_H)
	check_include_files(fcntl.h HAVE_FCNTL_H)
	check_function_exists(finite HAVE_FINITE)
	check_include_files(float.h HAVE_FLOAT_H)
	check_function_exists(fpclass HAVE_FPCLASS)
	check_function_exists(fprintf HAVE_FPRINTF)
	check_function_exists(fp_class HAVE_FP_CLASS)
	check_function_exists(ftime HAVE_FTIME)
	check_function_exists(getaddrinfo HAVE_GETADDRINFO)
	check_function_exists(gettimeofday HAVE_GETTIMEOFDAY)
	check_include_files(inttypes.h HAVE_INTTYPES_H)
	check_function_exists(isascii HAVE_ISASCII)
	check_function_exists(isinf HAVE_ISINF)
	check_function_exists(isnan HAVE_ISNAN)
	check_function_exists(isnand HAVE_ISNAND)
	check_library_exists(history append_history "" HAVE_LIBHISTORY)
	check_library_exists(lzma lzma_code "" HAVE_LIBLZMA)
	check_library_exists(pthread pthread_join "" HAVE_LIBPTHREAD)
	check_library_exists(readline readline "" HAVE_LIBREADLINE)
	check_library_exists(z gzread "" HAVE_LIBZ)
	check_include_files(limits.h HAVE_LIMITS_H)
	check_function_exists(localtime HAVE_LOCALTIME)
	check_include_files(lzma.h HAVE_LZMA_H)
	check_include_files(malloc.h HAVE_MALLOC_H)
	check_include_files(math.h HAVE_MATH_H)
	check_include_files(memory.h HAVE_MEMORY_H)
	check_function_exists(mmap HAVE_MMAP)
	check_function_exists(munmap HAVE_MUNMAP)
	check_symbol_exists(DIR ndir.h HAVE_NDIR_H)
	check_include_files(netdb.h HAVE_NETDB_H)
	check_include_files(netinet/in.h HAVE_NETINET_IN_H)
	check_include_files(poll.h HAVE_POLL_H)
	check_function_exists(printf HAVE_PRINTF)
	check_include_files(pthread.h HAVE_PTHREAD_H)
	check_function_exists(putenv HAVE_PUTENV)
	check_function_exists(rand HAVE_RAND)
	check_function_exists(rand_r HAVE_RAND_R)
	check_include_files(resolv.h HAVE_RESOLV_H)
	check_library_exists(dld shl_load "" HAVE_SHLLOAD)
	check_function_exists(signal HAVE_SIGNAL)
	check_include_files(signal.h HAVE_SIGNAL_H)
	check_function_exists(snprintf HAVE_SNPRINTF)
	check_function_exists(sprintf HAVE_SPRINTF)
	check_function_exists(srand HAVE_SRAND)
	check_function_exists(sscanf HAVE_SSCANF)
	check_function_exists(stat HAVE_STAT)
	check_include_files(stdarg.h HAVE_STDARG_H)
	check_include_files(stdint.h HAVE_STDINT_H)
	check_include_files(stdlib.h HAVE_STDLIB_H)
	check_function_exists(strftime HAVE_STRFTIME)
	check_include_files(strings.h HAVE_STRINGS_H)
	check_include_files(string.h HAVE_STRING_H)
	check_symbol_exists(DIR sys/dir.h HAVE_SYS_DIR_H)
	check_include_files(sys/mman.h HAVE_SYS_MMAN_H)
	check_symbol_exists(DIR sys/ndir.h HAVE_SYS_NDIR_H)
	check_include_files(sys/select.h HAVE_SYS_SELECT_H)
	check_include_files(sys/socket.h HAVE_SYS_SOCKET_H)
	check_include_files(sys/stat.h HAVE_SYS_STAT_H)
	check_include_files(sys/timeb.h HAVE_SYS_TIMEB_H)
	check_include_files(sys/time.h HAVE_SYS_TIME_H)
	check_include_files(sys/types.h HAVE_SYS_TYPES_H)
	check_function_exists(time HAVE_TIME)
	check_include_files(time.h HAVE_TIME_H)
	check_include_files(unistd.h HAVE_UNISTD_H)
	check_function_exists(va_copy HAVE_VA_COPY)
	check_function_exists(vfprintf HAVE_VFPRINTF)
	check_function_exists(vsnprintf HAVE_VSNPRINTF)
	check_function_exists(vsprintf HAVE_VSPRINTF)
	check_function_exists(__va_copy HAVE___VA_COPY)
	check_c_source_compiles("
		#include <stdlib.h>
		#include <iconv.h>
		extern
		#ifdef __cplusplus
		\"C\"
		#endif
		#if defined(__STDC__) || defined(__cplusplus)
		size_t iconv(iconv_t cd, char** inbuf, size_t* inbytesleft, char** outbuf, size_t* outbytesleft);
		#else
		size_t iconv();
		#endif
		int main() { return 0; }
	" ICONV_CONST_TEST)
	if(NOT ICONV_CONST_TEST)
		set(ICONV_CONST "const")
	endif()
	set(LT_OBJDIR ".libs/")
	check_c_source_compiles("
		#include <sys/socket.h>
		#include <sys/types.h>
		int main() { (void) send(1, (const char*) \"\", 1, 1); return 0; }
	" SEND_ARG2_CAST_CONST)
	if(NOT SEND_ARG2_CAST_CONST)
		set(SEND_ARG2_CAST "(char *)")
	else()
		set(SEND_ARG2_CAST "/**/")
	endif()
	check_include_files("float.h;stdarg.h;stdlib.h;string.h" STDC_HEADERS)
	check_c_source_compiles("
		#include <stdarg.h>
		void a(va_list* ap) {};
		int main() { va_list ap1, ap2; a(&ap1); ap2 = (va_list) ap1; return 0; }
	" VA_LIST_IS_ARRAY_TEST)
	if(VA_LIST_IS_ARRAY_TEST)
		set(VA_LIST_IS_ARRAY FALSE)
	else()
		set(VA_LIST_IS_ARRAY TRUE)
	endif()
	check_c_source_compiles("
		#include <stddef.h>
		#include <sys/socket.h>
		#include <sys/types.h>
		int main() { (void) getsockopt(1, 1, 1, NULL, (socklen_t*) NULL); return 0; }
	" XML_SOCKLEN_T_SOCKLEN_T)
	if(XML_SOCKLEN_T_SOCKLEN_T)
		set(XML_SOCKLEN_T socklen_t)
	else()
		check_c_source_compiles("
			#include <stddef.h>
			#include <sys/socket.h>
			#include <sys/types.h>
			int main() { (void) getsockopt(1, 1, 1, NULL, (size_t*) NULL); return 0; }
		" XML_SOCKLEN_T_SIZE_T)
		if(XML_SOCKLEN_T_SIZE_T)
			set(XML_SOCKLEN_T size_t)
		else()
			check_c_source_compiles("
				#include <stddef.h>
				#include <sys/socket.h>
				#include <sys/types.h>
				int main() { (void) getsockopt (1, 1, 1, NULL, (int*) NULL); return 0; }
			" XML_SOCKLEN_T_INT)
			set(XML_SOCKLEN_T int)
		endif()
	endif()
	configure_file(config.h.cmake.in config.h)
endif()

set(
	LIBXML2_HDRS
	include/libxml/c14n.h
	include/libxml/catalog.h
	include/libxml/chvalid.h
	include/libxml/debugXML.h
	include/libxml/dict.h
	include/libxml/DOCBparser.h
	include/libxml/encoding.h
	include/libxml/entities.h
	include/libxml/globals.h
	include/libxml/hash.h
	include/libxml/HTMLparser.h
	include/libxml/HTMLtree.h
	include/libxml/list.h
	include/libxml/nanoftp.h
	include/libxml/nanohttp.h
	include/libxml/parser.h
	include/libxml/parserInternals.h
	include/libxml/pattern.h
	include/libxml/relaxng.h
	include/libxml/SAX.h
	include/libxml/SAX2.h
	include/libxml/schemasInternals.h
	include/libxml/schematron.h
	include/libxml/threads.h
	include/libxml/tree.h
	include/libxml/uri.h
	include/libxml/valid.h
	include/libxml/xinclude.h
	include/libxml/xlink.h
	include/libxml/xmlIO.h
	include/libxml/xmlautomata.h
	include/libxml/xmlerror.h
	include/libxml/xmlexports.h
	include/libxml/xmlmemory.h
	include/libxml/xmlmodule.h
	include/libxml/xmlreader.h
	include/libxml/xmlregexp.h
	include/libxml/xmlsave.h
	include/libxml/xmlschemas.h
	include/libxml/xmlschemastypes.h
	include/libxml/xmlstring.h
	include/libxml/xmlunicode.h
	include/libxml/xmlwriter.h
	include/libxml/xpath.h
	include/libxml/xpathInternals.h
	include/libxml/xpointer.h
)

set(
	LIBXML2_SRCS
	buf.c
	c14n.c
	catalog.c
	chvalid.c
	debugXML.c
	dict.c
	encoding.c
	entities.c
	error.c
	globals.c
	hash.c
	HTMLparser.c
	HTMLtree.c
	legacy.c
	list.c
	nanoftp.c
	nanohttp.c
	parser.c
	parserInternals.c
	pattern.c
	relaxng.c
	SAX.c
	SAX2.c
	schematron.c
	threads.c
	tree.c
	uri.c
	valid.c
	xinclude.c
	xlink.c
	xmlIO.c
	xmlmemory.c
	xmlmodule.c
	xmlreader.c
	xmlregexp.c
	xmlsave.c
	xmlschemas.c
	xmlschemastypes.c
	xmlstring.c
	xmlunicode.c
	xmlwriter.c
	xpath.c
	xpointer.c
	xzlib.c
)

if(WIN32)
	list(APPEND LIBXML2_SRCS win32/libxml2.rc)
	file(
		WRITE
		${CMAKE_CURRENT_BINARY_DIR}/rcVersion.h
		"#define LIBXML_MAJOR_VERSION ${LIBXML_MAJOR_VERSION}\n"
		"#define LIBXML_MINOR_VERSION ${LIBXML_MINOR_VERSION}\n"
		"#define LIBXML_MICRO_VERSION ${LIBXML_MICRO_VERSION}\n"
		"#define LIBXML_DOTTED_VERSION \"${VERSION}\"\n"
	)
endif()

if(LIBXML2_WITH_SAX1)
	list(APPEND LIBXML2_SRCS DOCBparser.c)
endif()

if(LIBXML2_WITH_TRIO)
	list(APPEND LIBXML2_SRCS trio.c triostr.c)
endif()

__add_real_target(libxml_static lib SOURCE ${LIBXML2_HDRS} ${LIBXML2_SRCS}
							  LIB ${CMAKE_REQUIRED_LIBRARIES}
							  DEF ${DEFS} LIBXML_STATIC
							  INTERFACE_DEF LIBXML_STATIC
							  INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include/
							  ${CMAKE_CURRENT_BINARY_DIR}
							  )

if(NOT BUILD_SHARED_LIBS)
	target_compile_definitions(libxml_static INTERFACE LIBXML_STATIC)
	set(XML_CFLAGS "-DLIBXML_STATIC")
endif()

if(LIBXML2_WITH_THREADS)
	target_compile_definitions(libxml_static PRIVATE _REENTRANT)
	if(WIN32)
		target_compile_definitions(libxml_static PRIVATE HAVE_WIN32_THREADS)
	endif()
endif()

target_include_directories(
	libxml_static
	PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}/libxml2>
)

if(HAVE_DLOPEN)
	target_link_libraries(libxml_static PRIVATE dl)
	set(MODULE_PLATFORM_LIBS "-ldl")
endif()

if(HAVE_SHLLOAD)
	target_link_libraries(libxml_static PRIVATE dld)
	set(MODULE_PLATFORM_LIBS "-ldld")
endif()

if(UNIX)
	target_link_libraries(libxml_static PRIVATE m)
	set(M_LIBS "-lm")
endif()

if(WIN32)
	target_link_libraries(libxml_static PRIVATE ws2_32)
	set(WIN32_EXTRA_LIBADD "-lws2_32")
endif()

if(LIBXML2_WITH_THREADS)
	target_link_libraries(libxml_static PRIVATE Threads::Threads)
	set(THREAD_LIBS ${CMAKE_THREAD_LIBS_INIT})
endif()

configure_package_config_file(
	libxml2-config.cmake.cmake.in libxml2-config.cmake
	INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libxml2-${PROJECT_VERSION}
)

configure_file(include/libxml/xmlversion.h.in libxml/xmlversion.h)
if(MSVC)
	configure_file(include/libxml/xmlwin32version.h.in libxml/xmlwin32version.h)
endif()


if(CC_INSTALL_LIBXML_STATIC)
	__install_directory_specif(${CMAKE_CURRENT_SOURCE_DIR}/include/libxml libxml_static)
	INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/libxml/xmlversion.h DESTINATION include/libxml/)
endif()