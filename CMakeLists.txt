cmake_minimum_required(VERSION 3.11)
project(dechamps_ASIOUtil)

find_package(dechamps_cpputil CONFIG REQUIRED)

include(FetchContent)
set(FETCHCONTENT_QUIET FALSE)
FetchContent_Declare(asiosdk
	URL "https://download.steinberg.net/sdk_downloads/asiosdk_2.3.3_2019-06-14.zip"
	URL_HASH SHA1=4d0097725bcf1015c91fb84f89e1a888141bd131
	USES_TERMINAL_DOWNLOAD TRUE  # https://gitlab.kitware.com/cmake/cmake/issues/18238#note_440475
)
FetchContent_GetProperties(asiosdk)
if(NOT asiosdk_POPULATED)
  FetchContent_Populate(asiosdk)
endif()

set(CMAKE_CXX_STANDARD 17)
add_definitions(-DDECHAMPS_ASIOUTIL_BUILD)

add_library(asiosdk_ginclude INTERFACE)
target_include_directories(asiosdk_ginclude INTERFACE "$<INSTALL_INTERFACE:include>")

add_library(asiosdk_asioh INTERFACE)
target_include_directories(asiosdk_asioh INTERFACE "$<INSTALL_INTERFACE:include>")

add_library(asiosdk_asiosys INTERFACE)
target_include_directories(asiosdk_asiosys INTERFACE "$<INSTALL_INTERFACE:include>")

add_library(asiosdk_iasiodrv INTERFACE)
target_include_directories(asiosdk_iasiodrv INTERFACE "$<INSTALL_INTERFACE:include>")

add_library(asiosdk_asiolist STATIC "${asiosdk_SOURCE_DIR}/host/pc/asiolist.cpp")
target_include_directories(asiosdk_asiolist
	PUBLIC "$<BUILD_INTERFACE:${asiosdk_SOURCE_DIR}/host/pc>"
	PRIVATE "$<BUILD_INTERFACE:${asiosdk_SOURCE_DIR}/common>"
	INTERFACE "$<INSTALL_INTERFACE:include>"
)

add_library(asiosdk_asiodrivers STATIC "${asiosdk_SOURCE_DIR}/host/asiodrivers.cpp")
target_link_libraries(asiosdk_asiodrivers PRIVATE asiosdk_asiolist)
target_include_directories(asiosdk_asiodrivers
	PRIVATE "$<BUILD_INTERFACE:${asiosdk_SOURCE_DIR}/common>"
	INTERFACE "$<INSTALL_INTERFACE:include>"
)

add_library(asiosdk_asio STATIC "${asiosdk_SOURCE_DIR}/common/asio.cpp")
target_link_libraries(asiosdk_asio PRIVATE asiosdk_asiodrivers)
target_include_directories(asiosdk_asio
	PRIVATE "$<BUILD_INTERFACE:${asiosdk_SOURCE_DIR}/host>"
	PRIVATE "$<BUILD_INTERFACE:${asiosdk_SOURCE_DIR}/host/pc>"
	INTERFACE "$<INSTALL_INTERFACE:include>"
)

add_library(asio STATIC asio.cpp)
target_link_libraries(asio
	PUBLIC asiosdk_asioh
	PUBLIC asiosdk_asiosys
	PRIVATE dechamps_cpputil::endian
	PRIVATE dechamps_cpputil::string
)
target_include_directories(asio
	PUBLIC "$<BUILD_INTERFACE:${asiosdk_SOURCE_DIR}>"
	PUBLIC "$<INSTALL_INTERFACE:include>"
)
target_compile_options(asio PRIVATE /WX /W4 /permissive- /analyze)

install(FILES "${asiosdk_SOURCE_DIR}/host/ginclude.h" "${asiosdk_SOURCE_DIR}/common/asio.h" "${asiosdk_SOURCE_DIR}/common/asiosys.h" "${asiosdk_SOURCE_DIR}/common/iasiodrv.h" DESTINATION include/dechamps_ASIOUtil/asiosdk)
install(TARGETS asiosdk_ginclude asiosdk_asioh asiosdk_asiosys asiosdk_iasiodrv asiosdk_asiolist asiosdk_asiodrivers asiosdk_asio EXPORT dechamps_ASIOUtil-config ARCHIVE DESTINATION lib/dechamps_ASIOUtil)

install(FILES asio.h DESTINATION include/dechamps_ASIOUtil)
install(TARGETS asio EXPORT dechamps_ASIOUtil-config ARCHIVE DESTINATION lib/dechamps_ASIOUtil)

install(EXPORT dechamps_ASIOUtil-config NAMESPACE dechamps_ASIOUtil:: DESTINATION lib/cmake/dechamps_ASIOUtil)
