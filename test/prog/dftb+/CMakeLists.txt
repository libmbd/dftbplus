set(builddir ${CMAKE_CURRENT_BINARY_DIR})
set(srcdir ${CMAKE_CURRENT_SOURCE_DIR})

# execute_process is verbatim, therefore ";" must be replaced with " "
string(REGEX REPLACE ";" " " releasename "${RELEASE}")

list(APPEND FYPP_FLAGS -I${CMAKE_SOURCE_DIR}/prog/dftb+/include -DRELEASE="'${releasename}'")

execute_process(
  COMMAND ${CMAKE_SOURCE_DIR}/utils/test/testlist_to_fypp
  INPUT_FILE ${srcdir}/tests
  OUTPUT_FILE ${builddir}/_dftbplus_tests.fypp)

# Make sure, the line-marker option is not set in fypp
set(fyppflags ${FYPP_FLAGS})
list(REMOVE_ITEM fyppflags "-n")

execute_process(
  COMMAND ${FYPP} ${fyppflags} -DMPI_PROCS=${TEST_MPI_PROCS} -DOMP_THREADS=${TEST_OMP_THREADS}
  INPUT_FILE ${builddir}/_dftbplus_tests.fypp
  OUTPUT_FILE ${builddir}/_dftbplus_tests)

file(STRINGS ${builddir}/_dftbplus_tests dftbplus_tests_raw)
foreach(line IN LISTS dftbplus_tests_raw)
  string(STRIP "${line}" testname)
  if(NOT "${testname}" STREQUAL "")
    list(APPEND tests ${testname})
  endif()
endforeach()

foreach(test IN LISTS tests)
  add_test(
    NAME dftb+_${test}
    COMMAND ${srcdir}/bin/autotest2
        -r ${srcdir} -w ${builddir} -d ${srcdir}/bin/tagdiff
        -P "${TEST_RUNNER}" -p "$<TARGET_FILE:dftb+>"
        -s P,R,C,S ${test})
endforeach()
