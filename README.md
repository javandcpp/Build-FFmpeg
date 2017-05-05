#Build FFmpeg with x264


FFmpeg    git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg <br/>
X264      git clone http://git.videolan.org/git/x264.git



###1,build_x264.sh  编译X264 <br/>
###2,build_android.sh 编译FFmpeg Android动态与静态库 <br/>
###3,build_android_with_x264.sh  编译FFmpeg集成X264编码 <br/>


注意：clone x264项目后copy到ffmpeg目录下 <br/>
build_android_with_x264.sh 脚本

    #bin/bash 
    NDK=/Applications/android-ndk-r13 
    SYSROOT=$NDK/platforms/android-14/arch-arm/  
    TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64 
    # 加入x264编译库
    EXTRA_CFLAGS="-I./android-lib/include" #x264生成库目录
    EXTRA_LDFLAGS="-L./android-lib/lib"    #x264生成库目录
    export TMPDIR=/Users/loneswordman/Desktop/ffmpegtemp  #unable to create temporary file in
    #NDK的路径，根据自己的安装位置进行设置
    NDK=/Applications/android-ndk-r13 
    #编译针对的平台，可以根据自己的需求进行设置，这里选择最低支持android-14, arm架构，生成的so库是    放在，libs/armeabi文件夹下的，若针对x86架构，要选择arch-x86
    PLATFORM=$NDK/platforms/android-14/arch-arm
    #工具链的路径，根据编译的平台不同而不同，arm-linux-androideabi-4.9与上面设置的PLATFORM对应，4.9为工具的版本号，根据自己安装的NDK版本来确定，一般使用最新的版本
    TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64



    function build_x264
    {
     cd x264                  #cd X264目录
	./configure \
    --prefix=$PREFIX \
    --enable-static \
    --disable-shared \
    --enable-pic \
    --disable-asm \
    --disable-cli \
    --host=arm-linux \
    --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
    --sysroot=$PLATFORM
    
    make -j8
	make install
    }


    PREFIX=../android-lib     #X264编译生成目录
    build_x264                #编译x264
    cd ..                     #进入上层目录


    function build_one
    {
    ./configure \
    --prefix=$PREFIX \
    --target-os=linux \
    --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
    --arch=arm \
    --sysroot=$PLATFORM \
    --cc=$TOOLCHAIN/bin/arm-linux-androideabi-gcc \
    --nm=$TOOLCHAIN/bin/arm-linux-androideabi-nm \
    --enable-static \    #编译静态库
    --disable-shared \   #编译动态库
    --enable-version3
    --enable-runtime-cpudetect \
    --enable-gpl \
    --enable-small \
    --enable-cross-compile \
    --disable-encoders \
    --enable-libx264 \       
    --enable-encoder=libx264 \   #启用x264
    --disable-muxers \
    --enable-muxer=mov \
    --enable-muxer=ipod \
    --enable-muxer=psp \
    --enable-muxer=mp4 \
    --enable-muxer=avi \
    --disable-decoders \
    --enable-decoder=aac \
    --enable-decoder=aac_latm \
    --enable-decoder=h264 \
    --enable-decoder=mpeg4 \
    --disable-demuxers \
    --enable-demuxer=h264 \
    --enable-demuxer=mov \
    --disable-parsers \
    --enable-parser=aac \
    --enable-parser=ac3 \
    --enable-parser=h264 \
    --disable-protocols \
    --enable-protocol=file \
    --enable-protocol=rtmp \
    --disable-bsfs \
    --enable-bsf=aac_adtstoasc \
    --enable-bsf=h264_mp4toannexb \
    --disable-debug \
    --enable-postproc \
    --disable-doc \
    --disable-asm \
    --disable-ffmpeg \
    --disable-ffplay \
    --disable-ffprobe \
    --disable-ffserver \
    --disable-symver \
    --disable-stripping \
    --extra-cflags=$EXTRA_CFLAGS \
    --extra-ldflags=$EXTRA_LDFLAGS


    $ADDITIONAL_CONFIGURE_FLAG
    sed -i '' 's/HAVE_LRINT 0/HAVE_LRINT 1/g' config.h
    sed -i '' 's/HAVE_LRINTF 0/HAVE_LRINTF 1/g' config.h
    sed -i '' 's/HAVE_ROUND 0/HAVE_ROUND 1/g' config.h
    sed -i '' 's/HAVE_ROUNDF 0/HAVE_ROUNDF 1/g' config.h
    sed -i '' 's/HAVE_TRUNC 0/HAVE_TRUNC 1/g' config.h
    sed -i '' 's/HAVE_TRUNCF 0/HAVE_TRUNCF 1/g' config.h
    sed -i '' 's/HAVE_CBRT 0/HAVE_CBRT 1/g' config.h
    sed -i '' 's/HAVE_RINT 0/HAVE_RINT 1/g' config.h
    make clean
    make -j8
    make install
	
	#以上将静态库合成为动态库（编译过程中没有生成libpostproc.a，没找到原因，单独编译了一下FFmpeg拷贝到android-lib/lib目录，然后再执行的此脚本）
	
    $TOOLCHAIN/bin/arm-linux-androideabi-ld -rpath-link=$PLATFORM/usr/lib -L$PLATFORM/usr/lib -L$PREFIX/lib -soname libffmpeg.so -shared -nostdlib -Bsymbolic --whole-archive --no-undefined -o $PREFIX/libffmpeg.so \
    $PREFIX/lib/libx264.a \
    $PREFIX/lib/libavcodec.a \
    $PREFIX/lib/libavfilter.a \
    $PREFIX/lib/libswresample.a \
    $PREFIX/lib/libavformat.a \
    $PREFIX/lib/libavutil.a \
    $PREFIX/lib/libswscale.a \
    $PREFIX/lib/libpostproc.a \
    $PREFIX/lib/libavdevice.a \
    -lc -lm -lz -ldl -llog --dynamic-linker=/system/bin/linker $TOOLCHAIN/lib/gcc/arm-linux-androideabi/4.9.x/libgcc.a  
    #此路径需要注意NDK toolchains $TOOLCHAIN/lib/gcc/arm-linux-androideabi/4.9.x/libgcc.a 否则会找不到文件报错(注意4.9.x此文件名)
    }

    CPU=armv7-a
    OPTIMIZE_CFLAGS="-mfloat-abi=softfp -mfpu=vfp -marm -march=$CPU "
    PREFIX=./android-lib   
    ADDITIONAL_CONFIGURE_FLAG=
    build_one
    
    
    


