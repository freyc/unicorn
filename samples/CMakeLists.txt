
add_executable(sample_arm sample_arm.c)
target_link_libraries(sample_arm unicorn pthread)

add_executable(sample_armeb sample_armeb.c)
target_link_libraries(sample_armeb unicorn pthread)
