cmake_minimum_required(VERSION 3.15 FATAL_ERROR)

# Build options
option(BUILD_SHARED_LIBS "Build as a DLL." ON)
option(BUILD_TESTING "Build the testing tree." ON)

# Subsystem options
option(SDL_ENABLE_LIBC "Use the system C library" ON)
option(SDL_ENABLE_AUDIO "Enable the audio subsystem" ON)
option(SDL_ENABLE_VIDEO "Enable the video subsystem" ON)
option(SDL_ENABLE_EVENTS "Enable the events subsystem" ON)
option(SDL_ENABLE_JOYSTICK "Enable the joystick subsystem" ON)
option(SDL_ENABLE_CDROM "Enable the cdrom subsystem" ON)
option(SDL_ENABLE_THREADS "Enable the threading subsystem" ON)
option(SDL_ENABLE_TIMERS "Enable the timer subsystem" ON)
option(SDL_ENABLE_FILE "Enable the file subsystem" ON)
option(SDL_ENABLE_LOADSO "Enable the shared object loading subsystem" ON)
option(SDL_ENABLE_CPUINFO "Enable the cpuinfo subsystem" ON)
option(SDL_ENABLE_ASSEMBLY "Enable assembly routines" ON)

# Audio driver options
option(SDL_ENABLE_DISKAUDIO "support the disk writer audio driver" ON)
option(SDL_ENABLE_DUMMYAUDIO "support the dummy audio driver" ON)

# Video driver options
option(SDL_ENABLE_DUMMYVIDEO "use dummy video driver" ON)
option(SDL_ENABLE_VIDEO_OPENGL "include OpenGL context creation" ON)
option(SDL_ENABLE_DIRECTX "use DirectX for Win32 audio/video" ON)

# Assembly options
option(SDL_ENABLE_NASM "use nasm assembly blitters on x86" ON)

# Platform options
option(SDL_ENABLE_STDIO_REDIRECT "Redirect STDIO to files on Win32" ON)

# Misc options
option(SDL_ENABLE_SCREENSAVER "enable screensaver by default while any SDL application is running" OFF)

include(CheckIncludeFile)
include(CheckLanguage)
include(CheckStructHasMember)
include(CheckSymbolExists)
include(CheckTypeSize)

set(SDL_STANDALONE OFF)
if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
    set(SDL_STANDALONE ON)
    set_property(GLOBAL PROPERTY USE_FOLDERS ON)

    project(SDL
        VERSION 1.2.16
        DESCRIPTION "Simple Directmedia Layer"
        HOMEPAGE_URL https://www.libsdl.org/
    )

    include(GNUInstallDirs)
endif()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_C_VISIBILITY_PRESET hidden)
set(CMAKE_VISIBILITY_INLINES_HIDDEN ON)
if(MSVC)
    add_compile_options(/W3)
else()
    add_compile_options(-Wall)
endif()

if(CMAKE_C_BYTE_ORDER STREQUAL "BIG_ENDIAN")
    set(SDL_BYTEORDER 4321)
elseif(CMAKE_C_BYTE_ORDER STREQUAL "LITTLE_ENDIAN")
    set(SDL_BYTEORDER 1234)
else()
    include(TestBigEndian)
    test_big_endian(BIG_ENDIAN)
    if(BIG_ENDIAN)
        set(SDL_BYTEORDER 4321)
    else()
        set(SDL_BYTEORDER 1234)
    endif()
endif()

if(SDL_ENABLE_LIBC)
    set(HAVE_LIBC ON)

    # Skip testing for platforms with known configurations
    if(MSVC)
        set(HAVE_SYS_TYPES_H ON)
        set(HAVE_STDIO_H ON)
        set(HAVE_STDLIB_H ON)
        set(HAVE_MALLOC_H ON)
        set(HAVE_MEMORY_H ON)
        set(HAVE_STRING_H ON)
        set(HAVE_INTTYPES_H ON)
        set(HAVE_CTYPE_H ON)
        set(HAVE_MATH_H ON)
        set(HAVE_SIGNAL_H ON)
        set(STDC_HEADERS ON)

        set(HAVE_MALLOC ON)
        set(HAVE_CALLOC ON)
        set(HAVE_REALLOC ON)
        set(HAVE_FREE ON)
        set(HAVE_GETENV ON)
        set(HAVE_PUTENV ON)
        set(HAVE_QSORT ON)
        set(HAVE_ABS ON)
        set(HAVE_MEMSET ON)
        set(HAVE_MEMCPY ON)
        set(HAVE_MEMMOVE ON)
        set(HAVE_MEMCMP ON)
        set(HAVE_STRLEN ON)
        set(HAVE_STRDUP ON)
        set(HAVE__STRREV ON)
        set(HAVE__STRUPR ON)
        set(HAVE__STRLWR ON)
        set(HAVE_STRCHR ON)
        set(HAVE_STRRCHR ON)
        set(HAVE_STRSTR ON)
        set(HAVE_ITOA ON)
        set(HAVE__LTOA ON)
        set(HAVE__ULTOA ON)
        set(HAVE_STRTOL ON)
        set(HAVE_STRTOUL ON)
        set(HAVE__I64TOA ON)
        set(HAVE__UI64TOA ON)
        set(HAVE_STRTOLL ON)
        set(HAVE_STRTOULL ON)
        set(HAVE__STRTOI64 ON)
        set(HAVE__STRTOUI64 ON)
        set(HAVE_STRTOD ON)
        set(HAVE_ATOI ON)
        set(HAVE_ATOF ON)
        set(HAVE_STRCMP ON)
        set(HAVE_STRNCMP ON)
        set(HAVE__STRICMP ON)
        set(HAVE__STRNICMP ON)
        set(HAVE_SSCANF ON)
        set(HAVE_SNPRINTF ON)
        set(HAVE_VSNPRINTF ON)
        set(HAVE_SETJMP ON)

        set(SDL_HAS_64BIT_TYPE ON)
    else()
        set(HEADERS sys/types.h stdio.h stdlib.h malloc.h memory.h string.h
            strings.h inttypes.h ctype.h math.h iconv.h signal.h alloca.h
            sys/mman.h
        )
        foreach(header ${HEADERS})
            string(TOUPPER ${header} var)
            string(REGEX REPLACE [./] _ var ${var})
            string(PREPEND var HAVE_)
            check_include_file(${header} ${var})
        endforeach()

        if(HAVE_STDLIB_H AND HAVE_STRING_H)
            set(STDC_HEADERS ON)
        endif()

        if(HAVE_INTTYPES_H OR HAVE_STDINT_H)
            check_type_size(int64_t SIZEOF_INT64_T)
            if(HAVE_SIZEOF_INT64_T)
                set(SDL_HAS_64BIT_TYPE ON)
            endif()
        endif()

        set(FUNCTIONS alloca:alloca.h memcmp:string.h strtod:stdlib.h
            memprotect:sys/types.h,sys/mman.h malloc:stdlib.h calloc:stdlib.h
            realloc:stdlib.h free:stdlib.h getenv:stdlib.h putenv:stdlib.h
            unsetenv:stdlib.h qsort:stdlib.h abs:stdlib.h bcopy:strings.h
            memset:string.h memcpy:string.h memmove:string.h strlen:string.h
            strlcpy:string.h strlcat:string.h strdup:string.h _strrev:string.h
            _strupr:string.h _strlwr:string.h strchr:string.h strrchr:string.h
            strstr:string.h itoa:stdlib.h _ltoa:stdlib.h _uitoa:stdlib.h
            _ultoa:stdlib.h strtol:stdlib.h strtoul:stdlib.h _i64toa:stdlib.h
            _ui64toa:stdlib.h strtoll:stdlib.h strtoull:stdlib.h atoi:stdlib.h
            atof:stdlib.h strcmp:string.h strncmp:string.h _stricmp:string.h
            strcasecmp:strings.h _strnicmp:string.h strncasecmp:strings.h
            sscanf:stdio.h snprintf:stdio.h vsnprintf:stdio.h iconv:iconv.h
            sigaction:signal.h setjmp:setjmp.h nanosleep:time.h
            getauxval:sys/auxv.h elf_aux_info:sys/auxv.h _strtoi64:stdlib.h
            _strtoui64:stdlib.h
        )
        foreach(function ${FUNCTIONS})
            if(NOT function MATCHES "(.*):(.*)")
                message(FATAL_ERROR "Bad function list entry: %{function}")
            endif()
            set(function ${CMAKE_MATCH_1})
            set(headers ${CMAKE_MATCH_2})
            string(REPLACE "," ";" headers ${headers})
            set(found_headers true)
            foreach(header ${headers})
                string(TOUPPER ${header} var)
                string(REGEX REPLACE [./] _ var ${var})
                string(PREPEND var HAVE_)
                check_include_file(${header} ${var})
                if(NOT ${var})
                    set(found_headers false)
                    break()
                endif()
            endforeach()
            if(found_headers)
                string(TOUPPER ${function} var)
                string(PREPEND var HAVE_)
                check_symbol_exists(${function} "${headers}" ${var})
            endif()
        endforeach()

        find_library(LIBICONV iconv)
        find_library(LIBM m)

        check_struct_has_member(sigaction sa_sigaction signal.h HAVE_SA_SIGACTION)
    endif()
endif()

set(SUBSYSTEMS AUDIO VIDEO EVENTS JOYSTICK CDROM THREADS TIMERS FILE LOADSO
    CPUINFO ASSEMBLY
)
foreach(subsystem ${SUBSYSTEMS})
    if(NOT ENABLE_${subsystem})
        set(SDL_${subsystem}_DISABLED ON)
    endif()
endforeach()

if(SDL_ENABLE_AUDIO)
    if(SDL_ENABLE_DISKAUDIO)
        set(SDL_AUDIO_DRIVER_DISK ON)
    endif()
    if(SDL_ENABLE_DUMMYAUDIO)
        set(SDL_AUDIO_DRIVER_DUMMY ON)
        set(HAVE_AUDIO ON)
    endif()
    if(WIN32)
        set(SDL_AUDIO_DRIVER_WAVEOUT ON)
        if(SDL_ENABLE_DIRECTX)
            set(SDL_AUDIO_DRIVER_DSOUND ON)
        endif()
    endif()
endif()

if(SDL_ENABLE_VIDEO)
    if(SDL_ENABLE_DUMMYVIDEO)
        set(SDL_VIDEO_DRIVER_DUMMY ON)
        set(HAVE_VIDEO ON)
    endif()
    if(WIN32)
        set(SDL_VIDEO_DRIVER_WINDIB ON)
        if(SDL_ENABLE_VIDEO_OPENGL)
            set(SDL_VIDEO_OPENGL ON)
            set(SDL_VIDEO_OPENGL_WGL ON)
            set(HAVE_VIDEO ON)
        endif()
        if(SDL_ENABLE_DIRECTX)
            set(SDL_VIDEO_DRIVER_DDRAW ON)
            set(HAVE_VIDEO ON)
        endif()
    endif()
    if(SDL_ENABLE_ASSEMBLY AND SDL_ENABLE_NASM)
        check_type_size(void* SIZEOF_VOID_P BUILTIN_TYPES_ONLY)
        if(SIZEOF_VOID_P EQUAL 4)
            check_language(ASM_NASM)
            if(CMAKE_ASM_NASM_COMPILER)
                enable_language(ASM_NASM)
                set(SDL_HERMES_BLITTERS ON)
                if(NOT EXISTS ${CMAKE_CURRENT_BINARY_DIR}/detect/src/nasmvis.asm)
                    file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/detect/src/nasmvis.asm
[[GLOBAL _bar:function hidden
_bar:
]])
                endif()
                set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
                try_compile(NASM_HAS_HIDDEN_VISIBILITY ${CMAKE_CURRENT_BINARY_DIR}/detect/build
                    SOURCES ${CMAKE_CURRENT_BINARY_DIR}/detect/src/nasmvis.asm)
                if(NASM_HAS_HIDDEN_VISIBILITY)
                    set(CMAKE_ASM_NASM_FLAGS -DHIDDEN_VISIBILITY)
                endif()
            endif()
        endif()
    endif()
endif()

if(SDL_ENABLE_JOYSTICK AND WIN32)
    set(SDL_JOYSTICK_WINMM ON)
    set(HAVE_JOYSTICK ON)
endif()

if(SDL_ENABLE_CDROM AND WIN32)
    set(SDL_CDROM_WIN32 ON)
    set(HAVE_CDROM ON)
endif()

if(SDL_ENABLE_THREADS AND WIN32)
    set(SDL_THREAD_WIN32 ON)
    set(HAVE_THREADS ON)
endif()

if(SDL_ENABLE_TIMERS AND WIN32)
    set(SDL_TIMER_WIN32 ON)
    set(HAVE_TIMERS ON)
endif()

if(SDL_ENABLE_LOADSO AND WIN32)
    set(SDL_LOADSO_WIN32 ON)
    set(HAVE_LOADSO ON)
endif()

if(NOT SDL_ENABLE_SCREENSAVER)
    set(SDL_VIDEO_DISABLE_SCREENSAVER ON)
endif()

if(SDL_ENABLE_JOYSTICK AND NOT HAVE_JOYSTICK)
    set(SDL_JOYSTICK_DUMMY ON)
endif()

if(SDL_ENABLE_CDROM AND NOT HAVE_CDROM)
    set(SDL_CDROM_DISABLED ON)
endif()

if(SDL_ENABLE_THREADS AND NOT HAVE_THREADS)
    set(SDL_THREADS_DISABLED ON)
endif()

if(SDL_ENABLE_TIMERS AND NOT HAVE_TIMERS)
    set(SDL_TIMERS_DISABLED ON)
endif()

if(SDL_ENABLE_LOADSO AND NOT HAVE_LOADSO)
    set(SDL_LOADSO_DISABLED ON)
endif()

configure_file(include/SDL_config_cmake.h.in generated/include/SDL_config.h)

file(GLOB SOURCES
    src/*.c src/*.h
    src/audio/*.c src/audio/*.h
    src/cdrom/*.c src/cdrom/*.h
    src/cpuinfo/*.c src/cpuinfo/*.h
    src/events/*.c src/events/*.h
    src/file/*.c src/file/*.h
    src/stdlib/*.c src/stdlib/*.h
    src/thread/*.c src/thread/*.h
    src/timer/*.c src/timer/*.h
    src/video/*.c src/video/*.h
)
file(GLOB_RECURSE GENERATED_SOURCES ${CMAKE_CURRENT_BINARY_DIR}/generated/src/*)
file(GLOB_RECURSE GENERATED_HEADERS ${CMAKE_CURRENT_BINARY_DIR}/generated/include/*)
file(GLOB HEADERS include/*.h)
file(GLOB REMOVE_HEADERS include/SDL_config*.h)
list(REMOVE_ITEM HEADERS ${REMOVE_HEADERS} ${CMAKE_CURRENT_SOURCE_DIR}/include/SDL_copying.h)

if(SDL_ENABLE_AUDIO)
    if(SDL_ENABLE_DISKAUDIO)
        file(GLOB DISKAUDIO_SOURCES src/audio/disk/*.c src/audio/disk/*.h)
        list(APPEND SOURCES ${DISKAUDIO_SOURCES})
    endif()
    if(SDL_ENABLE_DUMMYAUDIO)
        file(GLOB DUMMYAUDIO_SOURCES src/audio/dummy/*.c src/audio/dummy/*.h)
        list(APPEND SOURCES ${DUMMYAUDIO_SOURCES})
    endif()
    if(WIN32)
        file(GLOB WAVEOUT_SOURCES src/audio/windib/*.c src/audio/windib/*.h)
        list(APPEND SOURCES ${WAVEOUT_SOURCES})
        if(SDL_ENABLE_DIRECTX)
            file(GLOB DSOUND_SOURCES src/audio/windx5/*.c src/audio/windx5/*.h)
            list(APPEND SOURCES ${DSOUND_SOURCES})
        endif()
    endif()
endif()

if(SDL_ENABLE_VIDEO)
    if(SDL_ENABLE_DUMMYVIDEO)
        file(GLOB DUMMYVIDEO_SOURCES src/video/dummy/*.c src/video/dummy/*.h)
        list(APPEND SOURCES ${DUMMYVIDEO_SOURCES})
    endif()
    if(WIN32)
        file(GLOB WINDIB_SOURCES src/video/wincommon/*.c src/video/wincommon/*.h src/video/windib/*.c src/video/windib/*.h)
        list(APPEND SOURCES ${WINDIB_SOURCES})
        if(SDL_ENABLE_DIRECTX)
            file(GLOB DDRAW_SOURCES src/video/windx5/*.c src/video/windx5/*.h)
            list(APPEND SOURCES ${DDRAW_SOURCES})
        endif()
    endif()
endif()

if(SDL_ENABLE_JOYSTICK)
    file(GLOB JOYSTICK_SOURCES src/joystick/*.c src/joystick/*.h)
    list(APPEND SOURCES ${JOYSTICK_SOURCES})
    if(WIN32)
        file(GLOB WINMM_JOYSTICK_SOURCES src/joystick/win32/*.c src/joystick/win32/*.h)
        list(APPEND SOURCES ${WINMM_JOYSTICK_SOURCES})
    endif()
endif()

if(SDL_ENABLE_CDROM AND WIN32)
    file(GLOB WIN32_CDROM_SOURCES src/cdrom/win32/*.c src/cdrom/win32/*.h)
    list(APPEND SOURCES ${WIN32_CDROM_SOURCES})
endif()

if(SDL_ENABLE_THREADS AND WIN32)
    file(GLOB WIN32_THREAD_SOURCES src/thread/win32/SDL_*.c src/thread/win32/SDL_*.h)
    list(APPEND SOURCES ${WIN32_THREAD_SOURCES} src/thread/generic/SDL_syscond.c)
endif()

if(SDL_ENABLE_TIMERS AND WIN32)
    file(GLOB WIN32_TIMER_SOURCES src/timer/win32/*.c src/timer/win32/*.h)
    list(APPEND SOURCES ${WIN32_TIMER_SOURCES})
endif()

if(SDL_ENABLE_LOADSO AND WIN32)
    file(GLOB WIN32_LOADSO_SOURCES src/loadso/win32/*.c src/loadso/win32/*.h)
    list(APPEND SOURCES ${WIN32_LOADSO_SOURCES})
endif()

if(WIN32 AND BUILD_SHARED_LIBS)
    file(GLOB RESOURCES src/main/win32/*.rc)
    list(APPEND SOURCES ${RESOURCES})
endif()

if(SDL_ENABLE_JOYSTICK AND NOT HAVE_JOYSTICK)
    file(GLOB JOYSTICK_SOURCES src/joystick/dummy/*.c src/joystick/dummy/*.h)
    list(APPEND SOURCES ${JOYSTICK_SOURCES})
endif()

if(SDL_ENABLE_CDROM AND NOT HAVE_CDROM)
    file(GLOB CDROM_SOURCES src/cdrom/dummy/*.c src/cdrom/dummy/*.h)
    list(APPEND SOURCES ${CDROM_SOURCES})
endif()

if(SDL_ENABLE_THREADS AND NOT HAVE_THREADS)
    file(GLOB THREAD_SOURCES src/thread/generic/*.c src/thread/generic/*.h)
    list(APPEND SOURCES ${THREAD_SOURCES})
endif()

if(SDL_ENABLE_TIMERS AND NOT HAVE_TIMERS)
    file(GLOB TIMER_SOURCES src/timer/dummy/*.c src/timer/dummy/*.h)
    list(APPEND SOURCES ${TIMER_SOURCES})
endif()

if(SDL_ENABLE_LOADSO AND NOT HAVE_LOADSO)
    file(GLOB LOADSO_SOURCES src/loadso/dummy/*.c src/loadso/dummy/*.h)
    list(APPEND SOURCES ${LOADSO_SOURCES})
endif()

add_library(SDL ${SOURCES} ${HEADERS} ${GENERATED_SOURCES} ${GENERATED_HEADERS})
target_include_directories(SDL PUBLIC ${CMAKE_CURRENT_BINARY_DIR}/generated/include include)
set_target_properties(SDL PROPERTIES PUBLIC_HEADER "${HEADERS};${GENERATED_HEADERS}")
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${SOURCES} ${HEADERS})
source_group(TREE ${CMAKE_CURRENT_BINARY_DIR}/generated FILES ${GENERATED_SOURCES} ${GENERATED_HEADERS})

if(WIN32)
    target_link_libraries(SDL PUBLIC SDLmain PRIVATE User32 Gdi32 WinMM)
    if(SDL_ENABLE_DIRECTX)
        target_link_libraries(SDL PRIVATE dxguid)
    endif()
    target_compile_definitions(SDL PRIVATE main=SDL_main _CRT_SECURE_NO_WARNINGS _CRT_NONSTDC_NO_WARNINGS)
endif()

if(WIN32)
    file(GLOB SOURCES src/main/win32/*.c src/main/win32/*.h)
    add_library(SDLmain STATIC ${SOURCES})
    target_include_directories(SDLmain PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/generated/include include)
    target_compile_definitions(SDLmain PRIVATE _CRT_SECURE_NO_WARNINGS)
    if(NOT SDL_ENABLE_STDIO_REDIRECT)
        target_compile_definitions(SDL_MAIN PRIVATE NO_STDIO_REDIRECT)
    endif()
endif()

if(SDL_STANDALONE)
    install(TARGETS SDL
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/SDL
    )
    if(WIN32)
        install(TARGETS SDLmain)
    endif()
    if(BUILD_TESTING)
        set(CMAKE_FOLDER Tests)
        add_subdirectory(test)
    endif()
else()
    add_library(SDL::SDL ALIAS SDL)
endif()
