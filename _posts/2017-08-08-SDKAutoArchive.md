---

layout: post

title: SDK自动打包

tags: [SDK, Framework]

Author: Steve.liu

---


背景：
由于iOS Framework打包出来是分CPU指令集的，所以需要每个CPU指令集都打包一个Framework，然后使用`lipo -create`命令将所有CPU指令集合并，才能形成一个兼容所有设备和模拟器的Framework。这样的打包流程复杂而繁琐，对于需要经常打包测试的需求来说明显是不合适的。

### 解决方案
使用xcode `Aggregate`工具中的`Run Script`自动进行不同CPU指令集打包，并自动合并所有CPU指令集

### 处理器指令集介绍

- ARM处理器
  	- armv7
  	- armv7s
 	- arm64 
- Mac处理器
	- i386
	- x86_64

指令集 | 设备|支持设备 
-----| ----| ----
arm64	 | 真机64位|iPhone6s、iphone6s plus、iPhone6、iPhone6 plus、iPhone5S、iPad Air、 iPad mini2
armv7s	 | 真机32位|iPhone5、iPhone5C、iPad4(iPad with Retina Display)
armv7	 | 真机32位|iPhone4、iPhone4S、iPad、iPad2、iPad3(The New iPad)、iPad mini、iPod Touch 3G、iPod Touch4
i386	 | 模拟器32位|针对intel通用微处理器32位处理器
x86_64	 | 模拟器64位|针对x86架构的64位处理器

### 添加 Aggregate

- 使用Xcode打开打包工程
- `File`→`New`→`Target`→`Cross-platform`→`Aggregate`
- 命名为`AvidlyAdsSDKArchive`(自由命名)

### 添加 Run Script

- 打开`AvidlyAdsSDKArchive``Aggregate `
- `BuildPhases`→`New`→`+`→`Cross-platform`→`New Run Script Phase`
- 打开新建的`Run Script`，在shell中键入打包代码

### shell编写打包代码

- 历史版本
 - [2017-08-08更新 适用于Xcode10 and before](#update1)
 - [2019-06-10更新 适用于Xcode11](#update2)
 - [2020-9-15更新 适用于Xcode12](#update3)

### <span id="update1">2017-08-08更新 适用于Xcode10 and before</span>

```
# 工程名称
# 如果工程名称和Framework的Target名称不一样的 话，要自定义FMKNAME
# 例如: FMK_NAME = "MyFramework"
FMK_NAME=${PROJECT_NAME}

# 工程Info.list路径
InfoPlist=${SRCROOT}/${PROJECT_NAME}/Info.plist

# 工程Version
Version=$(/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" $InfoPlist)
Version=${Version//./}

# 工程Build
Build=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" $InfoPlist)

# SDK文件夹名称
SDKFilePath=${SRCROOT}/${PROJECT_NAME}_${Version}_${Build}

# 文件路径是否存在
if [ -d "${SDKFilePath}" ]
then
rm -rf "${SDKFilePath}"
fi
mkdir -p "${SDKFilePath}"

# 当前时间
updataDate=`date +%F`

# 将信息写入文件
updataFileName=${SDKFilePath}/version.md
touch ${updataFileName}
echo 版本:${Version} >> ${updataFileName}
echo 编译:${Build} >> ${updataFileName}
echo 时间:${updataDate} >> ${updataFileName}
echo 更新: >> ${updataFileName}

# SDK目录
INSTALL_DIR=${SDKFilePath}/${FMK_NAME}.framework

# Working dir will be deleted after the framework creation.
WRK_DIR=build
DEVICE_DIR=${WRK_DIR}/Release-iphoneos/${FMK_NAME}.framework
SIMULATOR_DIR=${WRK_DIR}/Release-iphonesimulator/${FMK_NAME}.framework

# -configuration ${CONFIGURATION}
# Clean and Building both architectures.
xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphoneos clean build
xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphonesimulator clean build

# Cleaning the oldest.
if [ -d "${INSTALL_DIR}" ]
then
rm -rf "${INSTALL_DIR}"
fi
mkdir -p "${INSTALL_DIR}"
cp -R "${DEVICE_DIR}/" "${INSTALL_DIR}/"

# Uses the Lipo Tool to merge both binary files (i386 + armv6/armv7) into one Universal final product.
lipo -create "${DEVICE_DIR}/${FMK_NAME}" "${SIMULATOR_DIR}/${FMK_NAME}" -output "${INSTALL_DIR}/${FMK_NAME}"
rm -r "${WRK_DIR}"
open "${SRCROOT}"
```

<br>
### <span id="update2">2019-06-10更新 适用于Xcode11</span>

因Xcode11更新Bulid的目录，将Simulator和真机的编译目录修改为同一个地址，之前的脚本对导致出错，下边是修改后的sh脚本

```

# Sets the target folders and the final framework product.
# 如果工程名称和Framework的Target名称不一样的 话，要自定义FMKNAME
# 例如: FMK_NAME = "MyFramework"
FMK_NAME=${PROJECT_NAME}

PROJECT_PATH=$(cd "$(dirname "$0")";pwd)

# 工程Info.list路径
InfoPlist=${PROJECT_PATH}/${FMK_NAME}/Info.plist

# 工程Version
Version=$(/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" $InfoPlist)
Version=${Version//./}

# 工程Build
Build=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" $InfoPlist)

# SDK文件夹名称
SDKFilePath=${PROJECT_PATH}/${FMK_NAME}_${Version}_${Build}

if [ -d "${SDKFilePath}" ]
then
rm -rf "${SDKFilePath}"
fi
mkdir -p "${SDKFilePath}"

updataDate=`date +%F`
#updataMessage=版本:${Version}\\n编译:${Build}\\n时间:${updataDate}\\n更新:

updataFileName=${SDKFilePath}/version.md
touch ${updataFileName}
echo 版本:${Version} >> ${updataFileName}
echo 编译:${Build} >> ${updataFileName}
echo 时间:${updataDate} >> ${updataFileName}
echo 更新: >> ${updataFileName}

# SDK目录
INSTALL_DIR=${SDKFilePath}/${FMK_NAME}.framework

# Working dir will be deleted after the framework creation.
WRK_DIR=build
WRK_DIR_iphone=build_Release-iphoneos
WRK_DIR_iphonesimulator=build_Release-iphonesimulator

DEVICE_DIR=${WRK_DIR}/Release-iphoneos/${FMK_NAME}.framework
SIMULATOR_DIR=${WRK_DIR}/Release-iphonesimulator/${FMK_NAME}.framework

DEVICE_DIR1=${WRK_DIR_iphone}/Release-iphoneos/${FMK_NAME}.framework
SIMULATOR_DIR1=${WRK_DIR_iphonesimulator}/Release-iphonesimulator/${FMK_NAME}.framework

# -configuration ${CONFIGURATION}
# Clean and Building both architectures.
xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphoneos clean build

# Cleaning the oldest.
if [ -d "${DEVICE_DIR1}" ]
then
rm -rf "${DEVICE_DIR1}"
fi
mkdir -p "${DEVICE_DIR1}"
cp -R "${DEVICE_DIR}/" "${DEVICE_DIR1}/"


xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphonesimulator clean build

# Cleaning the oldest.
if [ -d "${SIMULATOR_DIR1}" ]
then
rm -rf "${SIMULATOR_DIR1}"
fi
mkdir -p "${SIMULATOR_DIR1}"
cp -R "${SIMULATOR_DIR}/" "${SIMULATOR_DIR1}/"

# Cleaning the oldest.
if [ -d "${INSTALL_DIR}" ]
then
rm -rf "${INSTALL_DIR}"
fi
mkdir -p "${INSTALL_DIR}"
cp -R "${DEVICE_DIR1}/" "${INSTALL_DIR}/"

# Uses the Lipo Tool to merge both binary files (i386 + armv6/armv7) into one Universal final product.
lipo -create "${DEVICE_DIR1}/${FMK_NAME}" "${SIMULATOR_DIR1}/${FMK_NAME}" -output "${INSTALL_DIR}/${FMK_NAME}"
rm -r "${WRK_DIR}"
rm -r "${WRK_DIR_iphone}"
rm -r "${WRK_DIR_iphonesimulator}"

open "${SRCROOT}"

```

<br>
### <span id="update2">2020-9-15更新 适用于Xcode12</span>

因Xcode12模拟器Release编译的时候，编译出来的Framework自带arm64架构，和真机编译出来的Framework的arm64架构冲突，合并不了，导致之前的脚本对导致出错。修改方案，是模拟器先移除arm64架构，再合并，下边是修改后的sh脚本

```

# Sets the target folders and the final framework product.
# 如果工程名称和Framework的Target名称不一样的 话，要自定义FMKNAME
# 例如: FMK_NAME = "MyFramework"
FMK_NAME=TraceAnalysisSDK

PROJECT_PATH=$(cd "$(dirname "$0")";pwd)

# 工程Info.list路径
InfoPlist=${PROJECT_PATH}/${FMK_NAME}/Info.plist

# 工程Version
Version=$(/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" $InfoPlist)
Version=${Version//./}

# 工程Build
Build=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" $InfoPlist)

# SDK文件夹名称
SDKFilePath=${PROJECT_PATH}/${FMK_NAME}_${Version}_${Build}

if [ -d "${SDKFilePath}" ]
then
rm -rf "${SDKFilePath}"
fi
mkdir -p "${SDKFilePath}"

updataDate=`date +%F`
#updataMessage=版本:${Version}\\n编译:${Build}\\n时间:${updataDate}\\n更新:

updataFileName=${SDKFilePath}/version.md
touch ${updataFileName}
echo 版本:${Version} >> ${updataFileName}
echo 编译:${Build} >> ${updataFileName}
echo 时间:${updataDate} >> ${updataFileName}
echo 更新: >> ${updataFileName}

# SDK目录
INSTALL_DIR=${SDKFilePath}/${FMK_NAME}.framework

# Working dir will be deleted after the framework creation.
WRK_DIR=build
WRK_DIR_iphone=build_Release-iphoneos
WRK_DIR_iphonesimulator=build_Release-iphonesimulator

DEVICE_DIR=${WRK_DIR}/Release-iphoneos/${FMK_NAME}.framework
SIMULATOR_DIR=${WRK_DIR}/Release-iphonesimulator/${FMK_NAME}.framework

DEVICE_DIR1=${WRK_DIR_iphone}/Release-iphoneos/${FMK_NAME}.framework
SIMULATOR_DIR1=${WRK_DIR_iphonesimulator}/Release-iphonesimulator/${FMK_NAME}.framework

# -configuration ${CONFIGURATION}
# Clean and Building both architectures.
xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphoneos clean build

# Cleaning the oldest.
if [ -d "${DEVICE_DIR1}" ]
then
rm -rf "${DEVICE_DIR1}"
fi
mkdir -p "${DEVICE_DIR1}"
cp -R "${DEVICE_DIR}/" "${DEVICE_DIR1}/"


xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphonesimulator clean build

# Cleaning the oldest.
if [ -d "${SIMULATOR_DIR1}" ]
then
rm -rf "${SIMULATOR_DIR1}"
fi
mkdir -p "${SIMULATOR_DIR1}"
cp -R "${SIMULATOR_DIR}/" "${SIMULATOR_DIR1}/"

# Cleaning the oldest.
if [ -d "${INSTALL_DIR}" ]
then
rm -rf "${INSTALL_DIR}"
fi
mkdir -p "${INSTALL_DIR}"
cp -R "${DEVICE_DIR1}/" "${INSTALL_DIR}/"

# Uses the Lipo Tool to merge both binary files (i386 + armv6/armv7) into one Universal final product.

lipo -remove arm64 "${SIMULATOR_DIR1}/${FMK_NAME}" -output "${WRK_DIR}/${FMK_NAME}"
lipo -create "${DEVICE_DIR1}/${FMK_NAME}" "${WRK_DIR}/${FMK_NAME}" -output "${INSTALL_DIR}/${FMK_NAME}"


#lipo -create "${DEVICE_DIR1}/${FMK_NAME}" "${SIMULATOR_DIR1}/${FMK_NAME}" -output "${INSTALL_DIR}/${FMK_NAME}"
rm -r "${WRK_DIR}"
rm -r "${WRK_DIR_iphone}"
rm -r "${WRK_DIR_iphonesimulator}"

open "${SRCROOT}"

```