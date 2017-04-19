
project(arm-softmmu C)

configure_file(${CMAKE_SOURCE_DIR}/config-target.h.in ${CMAKE_CURRENT_BINARY_DIR}/config-target.h)

include_directories(${QEMU_DIR}/target-arm ${CMAKE_CURRENT_BINARY_DIR})

set(ARM_SOURCES

	#${CMAKE_BINARY_DIR}/qapi-types.h
	#${CMAKE_BINARY_DIR}/qapi-visit.h

	${QEMU_DIR}/exec.c                    
	${QEMU_DIR}/translate-all.c           
	${QEMU_DIR}/cpu-exec.c                
	${QEMU_DIR}/tcg/tcg.c                 
	${QEMU_DIR}/tcg/optimize.c            
	${QEMU_DIR}/fpu/softfloat.c           
	${QEMU_DIR}/cpus.c                    
	${QEMU_DIR}/ioport.c                  
	${QEMU_DIR}/memory.c                  
	${QEMU_DIR}/cputlb.c                  
	${QEMU_DIR}/memory_mapping.c          
	${QEMU_DIR}/hw/arm/tosa.c             
	${QEMU_DIR}/hw/arm/virt.c             
	${QEMU_DIR}/target-arm/translate.c    
	${QEMU_DIR}/target-arm/op_helper.c    
	${QEMU_DIR}/target-arm/helper.c       
	${QEMU_DIR}/target-arm/cpu.c          
	${QEMU_DIR}/target-arm/neon_helper.c  
	${QEMU_DIR}/target-arm/iwmmxt_helper.c
	${QEMU_DIR}/target-arm/psci.c         
	${QEMU_DIR}/target-arm/unicorn_arm.c  
	${QEMU_DIR}/target-arm/crypto_helper.c

	${CMAKE_BINARY_DIR}/arm.h
	)

add_library(arm-softmmu OBJECT ${ARM_SOURCES})

set_target_properties(arm-softmmu PROPERTIES POSITION_INDEPENDENT_CODE ON)
target_compile_definitions(arm-softmmu PRIVATE NEED_CPU_H)
target_compile_options(arm-softmmu PRIVATE -include ${CMAKE_BINARY_DIR}/arm.h)

add_dependencies(arm-softmmu gen_code)


add_library(armeb-softmmu OBJECT ${ARM_SOURCES})

set_target_properties(armeb-softmmu PROPERTIES POSITION_INDEPENDENT_CODE ON)
target_compile_definitions(armeb-softmmu PRIVATE NEED_CPU_H TARGET_WORDS_BIGENDIAN)
target_compile_options(armeb-softmmu PRIVATE -include ${CMAKE_BINARY_DIR}/armeb.h)
