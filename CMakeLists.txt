
cmake_minimum_required(VERSION 3.7)

project(unicorn C)

set(UNICORN_ARCHS x86 arm m68k)
set(ARCHS x86_64 arm armeb m68k)

set(QEMU_DIR ${CMAKE_SOURCE_DIR}/qemu)


set(QEMU_ARCH "X86_64")
configure_file(${CMAKE_SOURCE_DIR}/config-host.h.in ${CMAKE_CURRENT_BINARY_DIR}/config-host.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

#[[
add_custom_target(qapi-types.h  
	COMMAND python ${QEMU_DIR}/scripts/qapi-types.py -h -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	#COMMAND python ${QEMU_DIR}/scripts/qapi-types.py -c -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	#COMMAND python ${QEMU_DIR}/scripts/qapi-visit.py -h -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	#COMMAND python ${QEMU_DIR}/scripts/qapi-visit.py -c -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	)

add_custom_target(qapi-types.c  
	#COMMAND python ${QEMU_DIR}/scripts/qapi-types.py -h -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	COMMAND python ${QEMU_DIR}/scripts/qapi-types.py -c -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	#COMMAND python ${QEMU_DIR}/scripts/qapi-visit.py -h -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	#COMMAND python ${QEMU_DIR}/scripts/qapi-visit.py -c -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	)

add_custom_target(qapi-visit.h  
	#COMMAND python ${QEMU_DIR}/scripts/qapi-types.py -h -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	#COMMAND python ${QEMU_DIR}/scripts/qapi-types.py -c -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	COMMAND python ${QEMU_DIR}/scripts/qapi-visit.py -h -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	#COMMAND python ${QEMU_DIR}/scripts/qapi-visit.py -c -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	)

add_custom_target(qapi-visit.c  
	#COMMAND python ${QEMU_DIR}/scripts/qapi-types.py -h -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	#COMMAND python ${QEMU_DIR}/scripts/qapi-types.py -c -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	#COMMAND python ${QEMU_DIR}/scripts/qapi-visit.py -h -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	COMMAND python ${QEMU_DIR}/scripts/qapi-visit.py -c -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	)
]]

add_custom_command(
	OUTPUT ${CMAKE_BINARY_DIR}/qapi-types.h
	COMMAND python ${QEMU_DIR}/scripts/qapi-types.py -h -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	DEPENDS ${QEMU_DIR}/scripts/qapi-types.py ${QEMU_DIR}/qapi-schema.json)
add_custom_command(
	OUTPUT ${CMAKE_BINARY_DIR}/qapi-visit.h
	COMMAND python ${QEMU_DIR}/scripts/qapi-visit.py -h -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	DEPENDS ${QEMU_DIR}/scripts/qapi-types.py ${QEMU_DIR}/qapi-schema.json)
add_custom_command(
	OUTPUT ${CMAKE_BINARY_DIR}/qapi-types.c
	COMMAND python ${QEMU_DIR}/scripts/qapi-types.py -c -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	DEPENDS ${QEMU_DIR}/scripts/qapi-types.py ${QEMU_DIR}/qapi-schema.json)
add_custom_command(
	OUTPUT ${CMAKE_BINARY_DIR}/qapi-visit.c
	COMMAND python ${QEMU_DIR}/scripts/qapi-visit.py -c -o "." -b -i ${QEMU_DIR}/qapi-schema.json
	DEPENDS ${QEMU_DIR}/scripts/qapi-types.py ${QEMU_DIR}/qapi-schema.json)

add_custom_target(gen_code ALL 
	DEPENDS ${CMAKE_BINARY_DIR}/qapi-types.h ${CMAKE_BINARY_DIR}/qapi-types.c ${CMAKE_BINARY_DIR}/qapi-visit.h ${CMAKE_BINARY_DIR}/qapi-visit.c)

#[[
execute_process(
	OUTPUT_FILE ${CMAKE_BINARY_DIR}/qapi-types.h
	COMMAND python ${QEMU_DIR}/scripts/qapi-types.py -h -o "." -b -i ${QEMU_DIR}/qapi-schema.json)

execute_process(
	OUTPUT_FILE ${CMAKE_BINARY_DIR}/qapi-visit.h
	COMMAND python ${QEMU_DIR}/scripts/qapi-visit.py -h -o "." -b -i ${QEMU_DIR}/qapi-schema.json )

execute_process(
	OUTPUT_FILE ${CMAKE_BINARY_DIR}/qapi-types.c
	COMMAND python ${QEMU_DIR}/scripts/qapi-types.py -c -o "." -b -i ${QEMU_DIR}/qapi-schema.json)

execute_process(
	OUTPUT_FILE ${CMAKE_BINARY_DIR}/qapi-visit.c
	COMMAND python ${QEMU_DIR}/scripts/qapi-visit.py -c -o "." -b -i ${QEMU_DIR}/qapi-schema.json )
]]

include_directories(
	${CMAKE_SOURCE_DIR}/include 
	${QEMU_DIR}
	${QEMU_DIR}/include
	${QEMU_DIR}/tcg
	${QEMU_DIR}/tcg/i386
)

# generate target header files
foreach(ARCH ${ARCHS})
	message(STATUS "arch: ${ARCH}")
	execute_process(COMMAND python ${QEMU_DIR}/header_gen.py ${ARCH} 
		OUTPUT_FILE ${CMAKE_BINARY_DIR}/${ARCH}.h)

	add_custom_command(
		OUTPUT ${CMAKE_BINARY_DIR}/${ARCH}.h
		COMMAND python ${QEMU_DIR}/header_gen.py ${ARCH} > ${CMAKE_BINARY_DIR}/${ARCH}.h
		COMMENT "Building ${ARCH}.h")
endforeach()

if("arm" IN_LIST UNICORN_ARCHS)
	add_definitions(-DUNICORN_HAS_ARM)
	add_subdirectory(${QEMU_DIR}/target-arm)
endif()



set(UNICORN_SOURCES
	#${CMAKE_BINARY_DIR}/qapi-types.c
	#${CMAKE_BINARY_DIR}/qapi-types.h
	#${CMAKE_BINARY_DIR}/qapi-visit.c
	#${CMAKE_BINARY_DIR}/qapi-visit.h
	
	${QEMU_DIR}/../uc.c
	${QEMU_DIR}/../list.c
	${QEMU_DIR}/glib_compat.c
	${QEMU_DIR}/accel.c
	${QEMU_DIR}/vl.c                   
	${QEMU_DIR}/qemu-timer.c              
	${QEMU_DIR}/qemu-log.c                
	${QEMU_DIR}/tcg-runtime.c             
	${QEMU_DIR}/hw/core/qdev.c            
	${QEMU_DIR}/hw/core/machine.c         
	${QEMU_DIR}/qom/object.c              
	${QEMU_DIR}/qom/container.c           
	${QEMU_DIR}/qom/qom-qobject.c         
	${QEMU_DIR}/qom/cpu.c
	${QEMU_DIR}/qapi/qapi-visit-core.c    
	${QEMU_DIR}/qapi/qapi-dealloc-visitor.c
	${QEMU_DIR}/qapi/qmp-input-visitor.c  
	${QEMU_DIR}/qapi/qmp-output-visitor.c 
	${QEMU_DIR}/qapi/string-input-visitor.c
	${QEMU_DIR}/qobject/qint.c            
	${QEMU_DIR}/qobject/qstring.c         
	${QEMU_DIR}/qobject/qdict.c           
	${QEMU_DIR}/qobject/qlist.c           
	${QEMU_DIR}/qobject/qfloat.c          
	${QEMU_DIR}/qobject/qbool.c           
	${QEMU_DIR}/qobject/qerror.c          
	${QEMU_DIR}/util/qemu-timer-common.c  
	${QEMU_DIR}/util/cutils.c             
	${QEMU_DIR}/util/oslib-posix.c        
	${QEMU_DIR}/util/qemu-thread-posix.c  
	${QEMU_DIR}/util/module.c             
	${QEMU_DIR}/util/bitmap.c             
	${QEMU_DIR}/util/bitops.c             
	${QEMU_DIR}/util/error.c              
	${QEMU_DIR}/util/aes.c                
	${QEMU_DIR}/util/crc32c.c             
	${QEMU_DIR}/util/host-utils.c         
	${QEMU_DIR}/util/getauxval.c
	$<TARGET_OBJECTS:arm-softmmu>
	$<TARGET_OBJECTS:armeb-softmmu>
	)


add_library(unicorn SHARED ${UNICORN_SOURCES})
add_dependencies(unicorn gen_code)

set_target_properties(unicorn PROPERTIES POSITION_INDEPENDENT_CODE ON)


add_subdirectory(samples)