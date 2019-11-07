set(LEN 10 CACHE STRING "Maximum length of JSON document")
set(KEYLEN 10 CACHE STRING "Maximum length of JSON key to search for")

set(unwind "${LEN}")

set(proof_output_dir "${CMAKE_CURRENT_BINARY_DIR}/value")

add_executable(
    value_proof_0010_initial_link
    "${CMAKE_CURRENT_SOURCE_DIR}/Value_harness.c"
    "${CMAKE_SOURCE_DIR}/jsonparser.c"
)

set_target_properties(
    value_proof_0010_initial_link PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${proof_output_dir}"
    INCLUDE_DIRECTORIES      "${CMAKE_SOURCE_DIR};${CMAKE_CURRENT_SOURCE_DIR}"
    COMPILE_DEFINITIONS      "CBMC;SKIP_COLON_STUB;SKIP_COMMA_STUB;SKIP_WHITESPACE_STUB;SKIP_STRING_STUB;SKIP_INTEGER_STUB;SKIP_HASH_STUB;SKIP_LIST_STUB;LEN=${LEN};KEYLEN=${KEYLEN}"
)

set(removed_functions_file "${proof_output_dir}/value_proof_0020_remove_functions")

add_custom_command(
    OUTPUT "${removed_functions_file}"
    DEPENDS value_proof_0010_initial_link
    COMMENT "Removing function body from value_proof"
    COMMAND "goto-instrument"
        "--remove-function-body" "skip_colon"
        "--remove-function-body" "skip_comma"
        "--remove-function-body" "skip_whitespace"
        "--remove-function-body" "skip_string"
        "--remove-function-body" "skip_integer"
        "--remove-function-body" "skip_hash"
        "--remove-function-body" "skip_list"
        "$<TARGET_FILE:value_proof_0010_initial_link>"
        "${removed_functions_file}"
)

add_custom_command(
    OUTPUT  "${proof_output_dir}/value_proof_0030_final"
    DEPENDS "${removed_functions_file}"
            "${CMAKE_CURRENT_SOURCE_DIR}/stub.c"
    COMMAND "goto-cc"
        "-o" "${proof_output_dir}/value_proof_0030_final"
        "-I${CMAKE_SOURCE_DIR}"
        "-I${CMAKE_CURRENT_SOURCE_DIR}"
        "-DCBMC"
        "-DSKIP_COLON_STUB"
        "-DSKIP_COMMA_STUB"
        "-DSKIP_WHITESPACE_STUB"
        "-DSKIP_STRING_STUB"
        "-DSKIP_INTEGER_STUB"
        "-DSKIP_HASH_STUB"
        "-DSKIP_LIST_STUB"
        "-DLEN=${LEN}"
        "-DKEYLEN=${KEYLEN}"
        "${removed_functions_file}"
        "${CMAKE_CURRENT_SOURCE_DIR}/stub.c"
)

set(final_binary "${proof_output_dir}/value_proof")
add_custom_target(
  final_proof ALL
  COMMAND "${CMAKE_COMMAND}" -E copy
          "${proof_output_dir}/value_proof_0030_final"
          "${final_binary}"
  DEPENDS "${proof_output_dir}/value_proof_0030_final"
)

set(_cbmc_flags
    "--bounds-check"
    "--pointer-check"
    "--div-by-zero-check"
    "--float-overflow-check"
    "--nan-check"
    "--pointer-overflow-check"
    "--undefined-shift-check"
    "--signed-overflow-check"
    "--unsigned-overflow-check"
    "--nondet-static"
    "--unwind" "${unwind}"
)
list(JOIN _cbmc_flags " " cbmc_flags)

set(test_labels
  "cbmc"
  "json-parser"
)

add_test(
  value_cbmc_trace
  COMMAND
    "/bin/sh" "-c"
    "cbmc --unwinding-assertions --trace ${cbmc_flags} ${final_binary} 2>&1 > ${proof_output_dir}/cbmc.txt"
)
set_tests_properties(value_cbmc_trace PROPERTIES LABELS test_labels)

add_test(
  value_cbmc_property
  COMMAND
    "/bin/sh" "-c"
    "cbmc --unwinding-assertions --show-properties ${cbmc_flags} ${final_binary} 2>&1 > ${proof_output_dir}/properties.xml"
)
set_tests_properties(value_cbmc_property PROPERTIES LABELS test_labels)

add_test(
  value_cbmc_coverage
  COMMAND
    "/bin/sh" "-c"
    "cbmc --cover location ${cbmc_flags} ${final_binary} 2>&1 > ${proof_output_dir}/properties.xml"
)
set_tests_properties(value_cbmc_coverage PROPERTIES LABELS test_labels)
