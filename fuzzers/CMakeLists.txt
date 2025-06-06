# fuzzers: libFuzzer and standalone fuzzing utilities

if(BUILD_FUZZERS AND NOT USE_STANDALONE_FUZZERS)
	set(CMAKE_REQUIRED_FLAGS "-fsanitize=fuzzer-no-link")
	add_c_flag(-fsanitize=fuzzer-no-link)
	unset(CMAKE_REQUIRED_FLAGS)
endif()

file(GLOB SRC_FUZZERS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *_fuzzer.c)
foreach(fuzz_target_src ${SRC_FUZZERS})
	string(REPLACE ".c" "" fuzz_target_name ${fuzz_target_src})
	string(REPLACE "_fuzzer" "" fuzz_name ${fuzz_target_name})

	set(${fuzz_target_name}_SOURCES
		${fuzz_target_src} "fuzzer_utils.c" ${LIBGIT2_OBJECTS})

	if(USE_STANDALONE_FUZZERS)
		list(APPEND ${fuzz_target_name}_SOURCES "standalone_driver.c")
	endif()

	add_executable(${fuzz_target_name} ${${fuzz_target_name}_SOURCES})
	target_include_directories(${fuzz_target_name} PRIVATE ${LIBGIT2_INCLUDES} ${LIBGIT2_DEPENDENCY_INCLUDES})
	target_include_directories(${fuzz_target_name} SYSTEM PRIVATE ${LIBGIT2_SYSTEM_INCLUDES})

	target_link_libraries(${fuzz_target_name} ${LIBGIT2_SYSTEM_LIBS})
	if(NOT USE_STANDALONE_FUZZERS)
		target_link_options(${fuzz_target_name} PRIVATE "-fsanitize=fuzzer")
	endif()

	add_test(${fuzz_target_name} "${CMAKE_CURRENT_BINARY_DIR}/${fuzz_target_name}" "${CMAKE_CURRENT_SOURCE_DIR}/corpora/${fuzz_name}")
endforeach()
