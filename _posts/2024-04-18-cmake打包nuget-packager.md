---
layout: post
tags: [config]
date: 2024-04-18
categories: [C#]
---
## 需求
把.NET平台的SDK的工程打包成nuget分发

## 方法
先预设您已经有了CMAKE的基础操作的能力

首先保证，代码编译文件输出目录结构是`./lib/${netversion}`这种形式
```cmake
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib/net472")
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_BINARY_DIR}/lib/net472")
```
其次在根目录的`CMakeLists.txt`的最后加上`CPackConfig.cmake`，主要的打包内容都将在`CPackConfig.cmake`中实现
```cmake
include(CPackConfig.cmake)
```
创建`CPackConfig.cmake`文件，内容如下
```cmake
# Variables for the package
set(CPACK_PACKAGE_NAME "xxxxxx")
set(CPACK_PACKAGE_VERSION_MAJOR "1")
set(CPACK_PACKAGE_VERSION_MINOR "1")
set(CPACK_PACKAGE_VERSION_PATCH "2")
set(CPACK_PACKAGE_CONTACT "someone <someone@somehost.xyz>")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "nuget package")
set(CPACK_PACKAGE_DESCRIPTION "nuget package for C# applications.")

set(CPACK_PACKAGING_INSTALL_PREFIX "/")
set(CPACK_SOURCE_IGNORE_FILES "${CPACK_SOURCE_IGNORE_FILES};somefolders;CMakeFiles;Sources;lib/xxxx;x64;bin")
list(APPEND CPACK_SOURCE_IGNORE_FILES "/somesolutions.sln")
include(CPack)


set(CPACK_GENERATOR "NuGet")
# Now define the NuGet package specifics
set(CPACK_NUGET_PACKAGE_AUTHORS "author") # 
set(CPACK_NUGET_PACKAGE_LICENSE_TYPE "GPL") # The GPL license does not permit closed-source software.
set(CPACK_NUGET_PACKAGE_PROJECT_URL "https://somehost.xyz") # Set this to your project URL
# set(CPACK_NUGET_PACKAGE_ICON "resources/icon.png") # Only if you have an icon
set(CPACK_NUGET_PACKAGE_TAGS "CMake;C#")
set(CPACK_NUGET_PACKAGE_REQUIRE_LICENSE_ACCEPTANCE false) # Set true if you want the user to accept the license
# set(CPACK_NUGET_PACKAGE_RELEASE_NOTES_FILE "${CMAKE_CURRENT_SOURCE_DIR}/RELEASE_NOTES.md") # Optional release notes file

set(CPACK_INSTALLED_DIRECTORIES "${CMAKE_BINARY_DIR};.")

include(CPack)
```

使用方法：
```cmd
cmake CmakeLIST_directory -DOPTION1=ON -DOPTION2=OFF
cmake --build . --config Release
cpack -C Release -G NuGet
```