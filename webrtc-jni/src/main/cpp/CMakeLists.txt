cmake_minimum_required(VERSION 3.13)

if(NOT WEBRTC_TOOLCHAIN_FILE STREQUAL "")
    message(STATUS "Toolchain file: ${WEBRTC_TOOLCHAIN_FILE}")
    set(CMAKE_TOOLCHAIN_FILE "${WEBRTC_TOOLCHAIN_FILE}")
endif()

project(webrtc-java)

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_ANDROID_LD "lld")
set(CMAKE_BUILD_TYPE Release)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
option(MOBILEGL_FORCE_RELEASE_OPT "Enable Release optimization flags in Debug build" ON)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3 -pipe -integrated-as -mllvm -polly -mllvm -polly-vectorizer=stripmine -mllvm -polly-invariant-load-hoisting -mllvm -polly-run-inliner -mllvm -polly-run-dce -mllvm -polly-invariant-load-hoisting -flto -mllvm -polly-run-inliner -mllvm -polly-run-dce -fdata-sections -ffunction-sections -mllvm -polly-detect-keep-going -mllvm -polly-ast-use-context -march=armv8-a+simd -fvisibility=hidden -Wall -Wextra -ferror-limit=0 -Wno-return-type -Wno-unused-parameter -Wno-c++98-compat -Wno-c++98-compat-pedantic -DGLM_FORCE_INTRINSICS -Wno-unicode-whitespace -Wno-format -Wno-unused-function -fmerge-all-constants -D_FILE_OFFSET_BITS=64 -D__USE_BSD -D_ALLBSD_SOURCE -mllvm -inlinehint-threshold=100000 -D_GNU_SOURCE -U__USE_GNU")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -pipe -integrated-as -mllvm -polly -mllvm -polly-vectorizer=stripmine -mllvm -polly-invariant-load-hoisting -mllvm -polly-run-inliner -mllvm -polly-run-dce -mllvm -polly-invariant-load-hoisting -flto -mllvm -polly-run-inliner -mllvm -polly-run-dce -fdata-sections -ffunction-sections -mllvm -polly-detect-keep-going -mllvm -polly-ast-use-context -march=armv8-a+simd -fvisibility=hidden -Wall -Wextra -ferror-limit=0 -Wno-return-type -Wno-unused-parameter -Wno-c++98-compat -Wno-c++98-compat-pedantic -DGLM_FORCE_INTRINSICS -Wno-unicode-whitespace -Wno-format -Wno-unused-function -Wno-vla-cxx-extension -frtti -fmerge-all-constants -D_FILE_OFFSET_BITS=64 -D__USE_BSD -D_ALLBSD_SOURCE -mllvm -inlinehint-threshold=100000 -D_GNU_SOURCE -U__USE_GNU")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fuse-ld=lld -flto -O3 -Wl,--strip-all -fvisibility=hidden -Wl,-O3 -Wl,--gc-sections -Wl,--as-needed -static-libstdc++ -Wl,--exclude-libs,ALL -Wl,--lto-O3")

if(UNIX AND NOT APPLE)
    # Apply the sysroot configuration for the actual build
    if(DEFERRED_SYSROOT)
        # Configure sysroot for all subsequent operations
        set(CMAKE_SYSROOT ${DEFERRED_SYSROOT})
        set(CMAKE_FIND_ROOT_PATH ${DEFERRED_SYSROOT})

        # Set pkg-config environment
        set(ENV{PKG_CONFIG_SYSROOT_DIR} "${DEFERRED_SYSROOT}")
        set(ENV{PKG_CONFIG_PATH} "${DEFERRED_SYSROOT}/usr/lib/pkgconfig:${DEFERRED_SYSROOT}/usr/share/pkgconfig")

        # Restrict library and header searches to sysroot only
        set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
        set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
        set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
    endif()

    set(LINUX TRUE)
endif()

if(APPLE)
    set(CMAKE_CXX_VISIBILITY_PRESET hidden)
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -s")
    set(SOURCE_TARGET macos)
elseif(LINUX)
    set(CMAKE_POSITION_INDEPENDENT_CODE ON)
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -s")
    set(SOURCE_TARGET linux)
elseif(WIN32)
    set(CMAKE_CXX_FLAGS_RELEASE "/MT")
    set(CMAKE_CXX_FLAGS_DEBUG "/MTd")
    set(SOURCE_TARGET windows)
endif()

if (CMAKE_CXX_COMPILER_ID STREQUAL Clang AND CMAKE_CXX_COMPILER_FRONTEND_VARIANT STREQUAL "MSVC")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /showFilenames")
endif()

add_subdirectory(dependencies/webrtc)
add_subdirectory(dependencies/jni-voithos)

file(GLOB SOURCES_ROOT "src/*.cpp")
file(GLOB SOURCES_API "src/api/*.cpp")
file(GLOB SOURCES_API_AUDIO "src/api/audio/*.cpp")
file(GLOB SOURCES_MEDIA "src/media/*.cpp")
file(GLOB SOURCES_MEDIA_AUDIO "src/media/audio/*.cpp")
file(GLOB SOURCES_MEDIA_AUDIO_OS "src/media/audio/${SOURCE_TARGET}/*.cpp")
file(GLOB SOURCES_MEDIA_VIDEO "src/media/video/*.cpp")
file(GLOB SOURCES_MEDIA_VIDEO_DESKTOP "src/media/video/desktop/*.cpp")
file(GLOB SOURCES_MEDIA_VIDEO_DESKTOP_OS "src/media/video/desktop/${SOURCE_TARGET}/*.cpp")
file(GLOB SOURCES_MEDIA_VIDEO_OS "src/media/video/${SOURCE_TARGET}/*.cpp")
file(GLOB SOURCES_PLATFORM "src/platform/${SOURCE_TARGET}/*.cpp")
file(GLOB SOURCES_RTC "src/rtc/*.cpp")

list(APPEND SOURCES
    ${SOURCES_ROOT}
    ${SOURCES_API}
    ${SOURCES_API_AUDIO}
    ${SOURCES_MEDIA}
    ${SOURCES_MEDIA_AUDIO}
    ${SOURCES_MEDIA_AUDIO_OS}
    ${SOURCES_MEDIA_VIDEO}
    ${SOURCES_MEDIA_VIDEO_DESKTOP}
    ${SOURCES_MEDIA_VIDEO_DESKTOP_OS}
    ${SOURCES_MEDIA_VIDEO_OS}
    ${SOURCES_PLATFORM}
    ${SOURCES_RTC}
)

add_library(${PROJECT_NAME} SHARED ${SOURCES})

# Introduced since WebRTC branch 7204 and Clang 20.
target_compile_options(${PROJECT_NAME} PRIVATE -Wno-nullability-completeness)

target_include_directories(${PROJECT_NAME}
    PRIVATE
        include
        include/api
        include/media
        include/rtc
)

set_target_properties(${PROJECT_NAME} PROPERTIES
    OUTPUT_NAME "${PROJECT_NAME}-${OUTPUT_NAME_SUFFIX}"
    CXX_STANDARD 20
    CXX_STANDARD_REQUIRED ON
    CXX_EXTENSIONS OFF
)

target_link_libraries(${PROJECT_NAME} jni-voithos)
target_link_libraries(${PROJECT_NAME} webrtc)

if(APPLE)
    set_source_files_properties(${SOURCES} PROPERTIES COMPILE_FLAGS "-x objective-c++")
    target_link_options(${PROJECT_NAME} PRIVATE "-ObjC")
    target_link_libraries(${PROJECT_NAME} "-framework Foundation" "-framework AVFoundation" "-framework CoreMedia" "-framework CoreAudio" "-framework IOKit" "-framework CoreVideo" "-framework VideoToolbox" "-framework QuartzCore")
elseif(WIN32)
    target_link_libraries(${PROJECT_NAME} dwmapi.lib mf.lib mfreadwrite.lib mfplat.lib mfuuid.lib shcore.lib)
endif()

install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX} COMPONENT Runtime
    LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX} COMPONENT Runtime
)
