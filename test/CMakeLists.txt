cmake_minimum_required(VERSION 3.15 FATAL_ERROR)

if(MSVC)
    add_compile_definitions(_CRT_SECURE_NO_WARNINGS _CRT_NONSTDC_NO_WARNINGS)
endif()

set(TESTS checkkeys graywin loopwave testalpha testbitmap testblitspeed
    testcdrom testcursor testdyngl testerror testfile testgamma testgl
    testhread testiconv testjoystick testkeys testlock testoverlay2
    testoverlay testpalette testplatform testsem testsprite testtimer
    testver testvidinfo testwin testwm threadwin torturethread testloadso
)
foreach(test ${TESTS})
    add_executable(${test} ${test}.c)
    source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${test}.c)
    target_link_libraries(${test} PRIVATE SDL)
endforeach()
