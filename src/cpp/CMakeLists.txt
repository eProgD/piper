cmake_minimum_required(VERSION 3.13)

project(piper C CXX)

find_package(PkgConfig)
pkg_check_modules(SPDLOG REQUIRED spdlog)

file(READ "${CMAKE_CURRENT_LIST_DIR}/../../VERSION" piper_version)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
#ADD_EXECUTABLE(piper main.cpp piper.cpp)
ADD_LIBRARY(piper STATIC piper.cpp)
#SHARED
string(APPEND CMAKE_CXX_FLAGS "-fPIC -Wall -Wextra -Wl,-rpath,'$ORIGIN'")
string(APPEND CMAKE_C_FLAGS " -Wall -Wextra")

set(PIPER_PHONEMIZE_ROOTDIR ${CMAKE_CURRENT_LIST_DIR}/../../lib/${CMAKE_HOST_SYSTEM_NAME}-${CMAKE_HOST_SYSTEM_PROCESSOR}/piper_phonemize)

target_link_libraries(piper
                      piper_phonemize
                      espeak-ng
                      onnxruntime
                      pthread
                      ${SPDLOG_LIBRARIES})

if(NOT APPLE)
  target_link_libraries(piper -static-libgcc -static-libstdc++)
endif()

target_link_directories(piper PUBLIC
                        ${PIPER_PHONEMIZE_ROOTDIR}/lib)

target_include_directories(piper PUBLIC
                           ${PIPER_PHONEMIZE_ROOTDIR}/include
                           ${SPDLOG_INCLUDE_DIRS})

target_compile_options(piper PUBLIC
                       ${SPDLOG_CFLAGS_OTHER})

target_compile_definitions(piper PUBLIC _PIPER_VERSION=${piper_version})

install(DIRECTORY ${PIPER_PHONEMIZE_ROOTDIR}/lib/espeak-ng-data DESTINATION .)
install(TARGETS piper DESTINATION lib)

install(DIRECTORY 
${PIPER_PHONEMIZE_ROOTDIR}/lib
DESTINATION lib_shared FILES_MATCHING PATTERN "libespeak-ng*")
install(DIRECTORY 
${PIPER_PHONEMIZE_ROOTDIR}/lib
DESTINATION lib_shared FILES_MATCHING PATTERN "libonnxruntime*")
install(FILES 
${PIPER_PHONEMIZE_ROOTDIR}/lib/libpiper_phonemize.so
DESTINATION lib_shared)

install(FILES
piper.hpp
json.hpp
utf8.h
wavfile.hpp
${PIPER_PHONEMIZE_ROOTDIR}/include/cpu_provider_factory.h 
${PIPER_PHONEMIZE_ROOTDIR}/include/onnxruntime_c_api.h 
${PIPER_PHONEMIZE_ROOTDIR}/include/onnxruntime_cxx_api.h 
${PIPER_PHONEMIZE_ROOTDIR}/include/onnxruntime_cxx_inline.h 
${PIPER_PHONEMIZE_ROOTDIR}/include/onnxruntime_run_options_config_keys.h 
${PIPER_PHONEMIZE_ROOTDIR}/include/onnxruntime_session_options_config_keys.h 
${PIPER_PHONEMIZE_ROOTDIR}/include/phoneme_ids.hpp 
${PIPER_PHONEMIZE_ROOTDIR}/include/phonemize.hpp 
${PIPER_PHONEMIZE_ROOTDIR}/include/provider_options.h 
${PIPER_PHONEMIZE_ROOTDIR}/include/tashkeel.hpp
${PIPER_PHONEMIZE_ROOTDIR}/include/espeak-ng/encoding.h 
${PIPER_PHONEMIZE_ROOTDIR}/include/espeak-ng/espeak_ng.h 
${PIPER_PHONEMIZE_ROOTDIR}/include/espeak-ng/speak_lib.h
DESTINATION include)