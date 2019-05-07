必须在linux下编译，且gcc,g++，make，python等环境已经安装好
使用android studio集成工具里面的sdk下载好，不要在android studio里面选择集成的ndk，如果下载ndk，因为下载的是最新的ndk版本，我使用ndk19编译gmssl，结果sm2加密代码运行就崩溃，最后使用ndk-r10e-linux-x86_64
编译出来的libcrypto.a，运行无问题。
解压ndk-r10e-linux-x86_64.zip, 并在android studio 下载好的sdk目录里面建好目录ndk-bundle, 将解压好的ndk-r10e-linux-x86_64里面的文件拷贝到ndk-bundle里面，
进入ndk-bundle/build/tools该目录，生成交叉工具链，如下：
./make-standalone-toolchain.sh --arch=arm --platform=android-22 --ndk-dir=/home/younger/android-sdk/ndk-bundle --system=linux-x86_64
./make-standalone-toolchain.sh --arch=arm64 --platform=android-22 --ndk-dir=/home/younger/android-sdk/ndk-bundle --system=linux-x86_64
./make-standalone-toolchain.sh --arch=x86 --platform=android-22 --ndk-dir=/home/younger/android-sdk/ndk-bundle --system=linux-x86_64
./make-standalone-toolchain.sh --arch=x86-64 --platform=android-22 --ndk-dir=/home/younger/android-sdk/ndk-bundle --system=linux-x86_64
该make-standalone-toolchain.sh依赖于python，一定要安装python才能正常运行，可指定--verbose来检查运行问题。
我生成了4种目标平台要使用的交叉工具链，交叉工具链生成后会在/tmp 里面生成arm-linux-androideabi-4.8.tar.bz2 这种文件，将其解压缩后拷入android-sdk/android-toolchain-arm
arm64的拷入android-sdk/aarch64-linux-android , x86的拷入android-sdk/x86 , x86-64的拷入android-sdk/x86-64。
4种平台的交叉工具链都拷贝完成后，就可以编译gmssl的android库了。
使用git clone https://github.com/guanzhi/GmSSL, 进入该目录，输入以下：
////////////////////////////////////////////////////////////////////////////
ANDROID_PATH=/home/younger/android-sdk

PLATFORM_VERSION=22

MAKE_TOOLCHAIN=$ANDROID_PATH/ndk-bundle/build/tools/make-standalone-toolchain.sh
export TOOLCHAIN_PATH=$ANDROID_PATH/android-toolchain-arm
$MAKE_TOOLCHAIN --arch=arm --platform=android-$PLATFORM_VERSION --install-dir=$TOOLCHAIN_PATH

export MACHINE=armv7
export SYSTEM=android
export ARCH=arm
export CROSS_SYSROOT=$TOOLCHAIN_PATH/sysroot
export TOOL_BASENAME=$TOOLCHAIN_PATH/bin/arm-linux-androideabi
export CC=$TOOL_BASENAME-gcc
export CXX=$TOOL_BASENAME-g++
export LD=$TOOL_BASENAME-ld
export LINK=$CXX
export AR=$TOOL_BASENAME-ar
export RANLIB=$TOOL_BASENAME-ranlib
export STRIP=$TOOL_BASENAME-strip

./config
make
编译armv7版本的libcrypto.a 和libssl.a ,编译完成后make clean 后编译arm64版本和x86、x86-64版本

arm64的如下：
////////////////////////////////////////////////////////////////////////////
ANDROID_PATH=/home/younger/android-sdk

PLATFORM_VERSION=22

MAKE_TOOLCHAIN=$ANDROID_PATH/ndk-bundle/build/tools/make-standalone-toolchain.sh
export TOOLCHAIN_PATH=$ANDROID_PATH/aarch64-linux-android
$MAKE_TOOLCHAIN --arch=aarch64 --platform=android-$PLATFORM_VERSION --install-dir=$TOOLCHAIN_PATH

export MACHINE=armv8
export SYSTEM=android64-aarch64
export ARCH=aarch64
export CROSS_SYSROOT=$TOOLCHAIN_PATH/sysroot
export TOOL_BASENAME=$TOOLCHAIN_PATH/bin/aarch64-linux-android
export CC=$TOOL_BASENAME-gcc
export CXX=$TOOL_BASENAME-g++
export LD=$TOOL_BASENAME-ld
export LINK=$CXX
export AR=$TOOL_BASENAME-ar
export RANLIB=$TOOL_BASENAME-ranlib
export STRIP=$TOOL_BASENAME-strip

./config
make

x86
////////////////////////////////////////////////////////////////////////////
ANDROID_PATH=/home/younger/android-sdk

PLATFORM_VERSION=22

MAKE_TOOLCHAIN=$ANDROID_PATH/ndk-bundle/build/tools/make-standalone-toolchain.sh
export TOOLCHAIN_PATH=$ANDROID_PATH/x86
$MAKE_TOOLCHAIN --arch=x86 --platform=android-$PLATFORM_VERSION --install-dir=$TOOLCHAIN_PATH

export MACHINE=x86
export SYSTEM=android-x86
export ARCH=x86
export CROSS_SYSROOT=$TOOLCHAIN_PATH/sysroot
export TOOL_BASENAME=$TOOLCHAIN_PATH/bin/i686-linux-android
export CC=$TOOL_BASENAME-gcc
export CXX=$TOOL_BASENAME-g++
export LD=$TOOL_BASENAME-ld
export LINK=$CXX
export AR=$TOOL_BASENAME-ar
export RANLIB=$TOOL_BASENAME-ranlib
export STRIP=$TOOL_BASENAME-strip

./config
make

x86-64如下：
////////////////////////////////////////////////////////////////////////////
ANDROID_PATH=/home/younger/android-sdk

PLATFORM_VERSION=22

MAKE_TOOLCHAIN=$ANDROID_PATH/ndk-bundle/build/tools/make-standalone-toolchain.sh
export TOOLCHAIN_PATH=$ANDROID_PATH/x86_64
$MAKE_TOOLCHAIN --arch=x86_64 --platform=android-$PLATFORM_VERSION --install-dir=$TOOLCHAIN_PATH

export MACHINE=x86_64
export SYSTEM=android64
export ARCH=x86_64
export CROSS_SYSROOT=$TOOLCHAIN_PATH/sysroot
export TOOL_BASENAME=$TOOLCHAIN_PATH/bin/x86_64-linux-android
export CC=$TOOL_BASENAME-gcc
export CXX=$TOOL_BASENAME-g++
export LD=$TOOL_BASENAME-ld
export LINK=$CXX
export AR=$TOOL_BASENAME-ar
export RANLIB=$TOOL_BASENAME-ranlib
export STRIP=$TOOL_BASENAME-strip

./config
make