cmake_minimum_required(VERSION 3.9)
project(nvmpi VERSION 1.0.0 DESCRIPTION "nvidia multimedia api")

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED on)

set(OPT_FLAGS "-Ofast -march=native")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OPT_FLAGS}")
#set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OPT_FLAGS}")

# Add the specific paths for aarch64 libraries and include files
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wno-unused-value -fPIC -I/usr/include/aarch64-linux-gnu")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra -Wno-unused-value -fPIC -I/usr/include/aarch64-linux-gnu")
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-rpath-link=/usr/lib/aarch64-linux-gnu/nvidia -L/usr/lib/aarch64-linux-gnu")
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-rpath-link=/usr/local/cuda/lib64")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -L/usr/lib/aarch64-linux-gnu")

find_library(LIB_V4L2 nvv4l2 PATHS /usr/lib/aarch64-linux-gnu/nvidia)
find_library(message (STATUS "LIB_NVJPEG: ${LIB_NVJPEG}") nvjpeg PATHS /usr/lib/aarch64-linux-gnu/nvidia)
find_package (Threads)

set(NVMPI_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)
set(NVMPI_PUBLIC_HEADER ${NVMPI_INCLUDE_DIR}/nvmpi.h)
set(CUDA_INCLUDE_DIR /usr/local/cuda/include)
set(JETSON_MULTIMEDIA_API_INCLUDE_DIR /usr/src/jetson_multimedia_api/include)
set(JETSON_MULTIMEDIA_API_SRC_DIR /usr/src/jetson_multimedia_api/samples/common/classes)

set(NVMPI_INCLUDE_DIRS
	${NVMPI_INCLUDE_DIR}
	${CUDA_INCLUDE_DIR}
	${JETSON_MULTIMEDIA_API_INCLUDE_DIR})

set(NVMPI_SRC
    ${CMAKE_CURRENT_SOURCE_DIR}/src/nvmpi_dec.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/nvmpi_enc.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/NVMPI_frameBuf.cpp
    ${JETSON_MULTIMEDIA_API_SRC_DIR}/NvBuffer.cpp
    ${JETSON_MULTIMEDIA_API_SRC_DIR}/NvElement.cpp
    ${JETSON_MULTIMEDIA_API_SRC_DIR}/NvElementProfiler.cpp
    ${JETSON_MULTIMEDIA_API_SRC_DIR}/NvLogging.cpp
    ${JETSON_MULTIMEDIA_API_SRC_DIR}/NvV4l2Element.cpp
    ${JETSON_MULTIMEDIA_API_SRC_DIR}/NvV4l2ElementPlane.cpp
    ${JETSON_MULTIMEDIA_API_SRC_DIR}/NvVideoDecoder.cpp
    ${JETSON_MULTIMEDIA_API_SRC_DIR}/NvVideoEncoder.cpp)
    
set(NVMPI_DEP_LIBS ${CMAKE_THREAD_LIBS_INIT} ${LIB_V4L2} ${LIB_NVJPEG})

#if NvUtils API is present prefer it to nvbuf_utils 
if(EXISTS "/usr/src/jetson_multimedia_api/include/nvbufsurface.h")
	add_definitions(-DWITH_NVUTILS)
	find_library(LIB_NVBUFSURFACE nvbufsurface PATHS /usr/lib/aarch64-linux-gnu/nvidia)
	find_library(LIB_NVBUFSURFTRANSFORM nvbufsurftransform PATHS /usr/lib/aarch64-linux-gnu/nvidia)
	set(NVMPI_SRC ${NVMPI_SRC}
		/usr/src/jetson_multimedia_api/samples/common/classes/NvBufSurface.cpp)
	set(NVMPI_DEP_LIBS ${NVMPI_DEP_LIBS} ${LIB_NVBUFSURFACE} ${LIB_NVBUFSURFTRANSFORM})
endif()

add_library(nvmpi SHARED ${NVMPI_SRC})
add_library(nvmpi_static STATIC ${NVMPI_SRC})

set_target_properties(nvmpi_static PROPERTIES OUTPUT_NAME nvmpi)
set_target_properties(nvmpi PROPERTIES VERSION ${PROJECT_VERSION})
set_target_properties(nvmpi PROPERTIES SOVERSION 1)
set_target_properties(nvmpi nvmpi_static PROPERTIES PUBLIC_HEADER ${NVMPI_PUBLIC_HEADER})
set_target_properties(nvmpi PROPERTIES LINK_FLAGS "-Wl,--no-as-needed")
target_link_libraries(nvmpi PRIVATE ${NVMPI_DEP_LIBS})
target_include_directories(nvmpi PRIVATE ${NVMPI_INCLUDE_DIRS})
target_include_directories(nvmpi_static PRIVATE ${NVMPI_INCLUDE_DIRS})
configure_file(nvmpi.pc.in nvmpi.pc @ONLY)
include(GNUInstallDirs)
install(TARGETS nvmpi nvmpi_static
	    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(FILES ${CMAKE_BINARY_DIR}/nvmpi.pc
	    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig)
install(FILES ${CMAKE_BINARY_DIR}/nvmpi.pc
	    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
