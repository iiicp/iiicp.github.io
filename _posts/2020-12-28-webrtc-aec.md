---
layout: post
title: 使用cmake在android平台编译webrtc的aec模块
tags: audio process
excerpt: audio  
---   

使用c++开发项目不方便的地方就在于构建，在早期linux平台采用makefile，windows平台是visual studio，iOS平台是xcode，android平台是android.mk之类，windows和iOS相对来说简单点，毕竟有
完善的IDE，导入文件设置头文件和库文件路径，编译生成动态库、静态库都很方便。但是android和linux
平台体验就差了点，还好有cmake工具，通过cmake在android和linux平台可以不用太做修改的构建。

android平台上构建需要指定android的工具链.

``` 
# eg: 当前在build目录下
cmake .. -DANDROID_ABI=arm64_v8a -DCMAKE_TOOLCHAIN_FILE=$NDK_HOME/build/cmake/android.toolchain.cmake  
``` 

下面是我编译aec模块的cmakelists.txt，可以作为一个比较通用的模块.   

```   
CMAKE_MINIMUM_REQUIRED(VERSION 3.10)
project(webrtc_aec)

set(CMAKE_VERBOSE_MAKEFILE ON)
find_library(log-lib log)

#message(STATUS ${log-lib_HEADER})
#message(STATUS ${log-lib_HEADERS})

# 相关宏定义
LIST(APPEND MY_COMPILE_FLAGS
        -D_ANDROID -DPLATFORM_ANDROID
        -DWEBRTC_HAS_NEON -DWEBRTC_POSIX -DWEBRTC_ANDROID -DWEBRTC_APM_DEBUG_DUMP=0 -DWEBRTC_NS_FLOAT -DRTC_DISABLE_CHECK_MSG
        -DNDEBUG
        )

# 相关链接标志
LIST(APPEND MY_LINK_FLAGS
        -llog -lm -lz)

set(TARGET_ABI ${ANDROID_ABI})

# 相关编译参数
set(CMAKE_CXX_FLAGS "-std=c++11 -fvisibility=hidden -fvisibility-inlines-hidden ${CMAKE_CXX_FLAGS}")
set(CMAKE_C_FLAGS "-std=c99 -fvisibility=hidden -fvisibility-inlines-hidden ${CMAKE_C_FLAGS}")

if (ANDROID_ABI STREQUAL armeabi-v7a)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfpu=neon")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mfpu=neon")
    set(ANDROID_ARM_NEON TRUE)
endif ()

#头文件
include_directories(.)

#源文件
set(aecsrc
        webrtc/base/stringutils.cc
        webrtc/base/checks.cc
        webrtc/common_audio/audio_util.cc
        webrtc/common_audio/channel_buffer.cc
        webrtc/common_audio/fft4g.c
        webrtc/common_audio/ring_buffer.c
        webrtc/common_audio/sparse_fir_filter.cc
        webrtc/common_audio/wav_file.cc
        webrtc/common_audio/wav_header.cc
        webrtc/common_audio/signal_processing/auto_corr_to_refl_coef.c
        webrtc/common_audio/signal_processing/auto_correlation.c
        webrtc/common_audio/signal_processing/complex_bit_reverse.c
        webrtc/common_audio/signal_processing/complex_fft.c
        webrtc/common_audio/signal_processing/copy_set_operations.c
        webrtc/common_audio/signal_processing/cross_correlation.c
        webrtc/common_audio/signal_processing/cross_correlation_neon.c
        webrtc/common_audio/signal_processing/division_operations.c
        webrtc/common_audio/signal_processing/dot_product_with_scale.c
        webrtc/common_audio/signal_processing/downsample_fast.c
        webrtc/common_audio/signal_processing/downsample_fast_neon.c
        webrtc/common_audio/signal_processing/energy.c
        webrtc/common_audio/signal_processing/filter_ar.c
        webrtc/common_audio/signal_processing/filter_ar_fast_q12.c
        webrtc/common_audio/signal_processing/filter_ma_fast_q12.c
        webrtc/common_audio/signal_processing/get_hanning_window.c
        webrtc/common_audio/signal_processing/get_scaling_square.c
        webrtc/common_audio/signal_processing/ilbc_specific_functions.c
        webrtc/common_audio/signal_processing/levinson_durbin.c
        webrtc/common_audio/signal_processing/lpc_to_refl_coef.c
        webrtc/common_audio/signal_processing/min_max_operations.c
        webrtc/common_audio/signal_processing/min_max_operations_neon.c
        webrtc/common_audio/signal_processing/randomization_functions.c
        webrtc/common_audio/signal_processing/real_fft.c
        webrtc/common_audio/signal_processing/refl_coef_to_lpc.c
        webrtc/common_audio/signal_processing/resample.c
        webrtc/common_audio/signal_processing/resample_48khz.c
        webrtc/common_audio/signal_processing/resample_by_2.c
        webrtc/common_audio/signal_processing/resample_by_2_internal.c
        webrtc/common_audio/signal_processing/resample_fractional.c
        webrtc/common_audio/signal_processing/spl_init.c
        webrtc/common_audio/signal_processing/spl_inl.c
        webrtc/common_audio/signal_processing/spl_sqrt.c
        webrtc/common_audio/signal_processing/spl_sqrt_floor.c
        webrtc/common_audio/signal_processing/splitting_filter_impl.c
        webrtc/common_audio/signal_processing/sqrt_of_one_minus_x_squared.c
        webrtc/common_audio/signal_processing/vector_scaling_operations.c
        webrtc/common_audio/vad/vad_core.c
        webrtc/common_audio/vad/vad_filterbank.c
        webrtc/common_audio/vad/vad_gmm.c
        webrtc/common_audio/vad/vad_sp.c
        webrtc/common_audio/vad/vad.cc
        webrtc/common_audio/vad/webrtc_vad.c
        webrtc/modules/audio_processing/splitting_filter.cc
        webrtc/modules/audio_processing/three_band_filter_bank.cc
        webrtc/modules/audio_processing/aec/aec_core.cc
        webrtc/modules/audio_processing/aec/aec_core_neon.cc
        webrtc/modules/audio_processing/aec/aec_resampler.cc
        webrtc/modules/audio_processing/aec/echo_cancellation.cc
        webrtc/modules/audio_processing/aec/fftwrap.c
        webrtc/modules/audio_processing/aec/filterbank.c
        webrtc/modules/audio_processing/aec/mdf.c
        webrtc/modules/audio_processing/aec/preprocess.c
        webrtc/modules/audio_processing/aec/smallft.c
        webrtc/modules/audio_processing/agc/legacy/analog_agc.c
        webrtc/modules/audio_processing/agc/legacy/digital_agc.c
        webrtc/modules/audio_processing/logging/apm_data_dumper.cc
        webrtc/modules/audio_processing/utility/block_mean_calculator.cc
        webrtc/modules/audio_processing/utility/delay_estimator.cc
        webrtc/modules/audio_processing/utility/delay_estimator_wrapper.cc
        webrtc/modules/audio_processing/utility/ooura_fft.cc
        webrtc/modules/audio_processing/utility/ooura_fft_neon.cc
        webrtc/modules/audio_processing/ns/noise_suppression.c
        webrtc/modules/audio_processing/ns/noise_suppression_x.c
        webrtc/modules/audio_processing/ns/ns_core.c
        webrtc/modules/audio_processing/ns/nsx_core_neon.c
        webrtc/modules/audio_processing/ns/nsx_core.c
        webrtc/modules/audio_processing/ns/nsx_core_c.c
        webrtc/system_wrappers/source/cpu_features.cc
        gsl-master/err/error.c
        gsl-master/err/message.c
        gsl-master/err/stream.c
        gsl-master/err/strerror.c
        gsl-master/specfunc/cheb_eval.c
        gsl-master/specfunc/expint.c
        )

# 生成库
add_library(${PROJECT_NAME} ${aecsrc})

# 添加宏定义
target_compile_definitions(${PROJECT_NAME} PUBLIC ${MY_COMPILE_FLAGS})

# std11 的特征
target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_11)

# 添加依赖的库
#target_link_libraries(${PROJECT_NAME} PUBLIC log-lib)

# 开始 strip
if (BUILD_SHARED_LIBS AND CMAKE_BUILD_TYPE STREQUAL Release)
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
            #            COMMAND ${CMAKE_STRIP} ${CMAKE_CURRENT_SOURCE_DIR}/build/${CMAKE_SYSTEM_NAME}/${ANDROID_ABI}/lib${FINAL_OUTPUT}.so
            COMMAND ${CMAKE_STRIP} lib${PROJECT_NAME}.so
            )
endif ()
```   
