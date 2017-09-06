# Windows Caffe

**这是一个测试版本，我已经将[ssd](https://github.com/weiliu89/caffe/tree/ssd)的Linux版本移植到Windows平台**


This branch of Caffe ports the framework to Windows.

[![Travis Build Status](https://api.travis-ci.org/BVLC/caffe.svg?branch=windows)](https://travis-ci.org/BVLC/caffe) Travis (Linux build)

[![Build status](https://ci.appveyor.com/api/projects/status/ew7cl2k1qfsnyql4/branch/windows?svg=true)](https://ci.appveyor.com/project/BVLC/caffe/branch/windows) AppVeyor (Windows build)

**Update**: 这个分支不是活跃的主分支。 请选择 [this](https://github.com/BVLC/caffe/tree/windows) 得到更多的Windows支持。


## Windows Setup

### 必需

 - Visual Studio 2013 or 2015
 - [CMake](https://cmake.org/) 3.4 or higher (Visual Studio and [Ninja](https://ninja-build.org/) generators are supported)

### 可选择的依赖

 - Python for the pycaffe interface. Anaconda Python 2.7 or 3.5 x64 (or Miniconda)
 - Matlab for the matcaffe interface.
 - CUDA 7.5 or 8.0 (use CUDA 8 if using Visual Studio 2015)
 - cuDNN v5

 We assume that `cmake.exe` and `python.exe` are on your `PATH`.

### 配置和构建caffe

在Windows上开始使用caffe的最快方法是通过在`cmd`提示符下执行以下命令（我们使用`C：\ Projects`作为其余指令的根文件夹,文件夹可以自定义修改）：

```cmd
C:\Projects> https://github.com/runhang/caffe-ssd.git
C:\Projects> cd caffe-ssd
:: Edit any of the options inside build_win.cmd to suit your needs
C:\Projects\caffe> scripts\build_win.cmd
```
The `build_win.cmd` script 将会下载依赖，创建Visual Studio工程文件(或者ninja构建文件)和构建Release配置。默认情况下，所有必需的DLL文件将被复制（或者尽可能硬链接）在二进制文件所在文件夹下。 如果要禁用此选项，可以通过更改命令行选项`-DCOPY_PREREQUISITES = 0`。 预构建库还提供了一个`prependpath.bat`批处理脚本，可以临时修改“PATH”环境变量，以使所需的DLL可用。

下面是一个更完整的一些参与建设咖啡的步骤的描述。

### 安装caffe依赖
默认情况下，CMake将会自动去下载一个依赖库。如果你觉得网络太慢，可以进入scripts文件夹，用编辑器打开download_prebuilt_dependencies.py。然后，你就发现了依赖库的的下载地址，这里我选的是v140版本的py3.5依赖库。

地址链接：

('v120', '2.7'):[点击链接下载](https://github.com/willyd/caffe-builder/releases/download/v1.1.0/libraries_v120_x64_py27_1.1.0.tar.bz2)，
('v140', '2.7'):[点击链接下载](https://github.com/willyd/caffe-builder/releases/download/v1.1.0/libraries_v140_x64_py27_1.1.0.tar.bz2)，
('v140', '3.5'):[点击链接下载](https://github.com/willyd/caffe-builder/releases/download/v1.1.0/libraries_v140_x64_py35_1.1.0.tar.bz2).

下完依赖包，然后在caffe目录下，新建一个名为“build”的文件夹，然后再把我们下好的依赖包解压。解压完后，是一个libraries文件夹，然后把\libraries\bin,\libraries\lib,\libraries\x64\vc14\bin三个的绝对路径添加到环境变量里面。当然也可以直接将压缩包复制到“C:\Users\自己用户名\.caffe\dependencies\download”文件夹下，将“自己的用户名”修改为你自己的用户名即可
### Use cuDNN

要使用cuDNN，最简单的方法是将`cuda`文件夹的内容复制到CUDA工具包安装目录中。 例如，如果您安装了CUDA 8.0并下载了cudnn-8.0-windows10-x64-v5.1.zip，则应将“cuda”目录的内容复制到`C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0`。 或者，您可以定义CUDNN_ROOT缓存变量来指向您解压缩cuDNN文件的位置，例如`C:/Projects/caffe/cudnn-8.0-windows10-x64-v5.1/cuda`。 例如，[scripts/build_win.cmd](scripts/build_win.cmd) 中的命令将成为：

```
cmake -G"!CMAKE_GENERATOR!" ^
      -DBLAS=Open ^
      -DCMAKE_BUILD_TYPE:STRING=%CMAKE_CONFIG% ^
      -DBUILD_SHARED_LIBS:BOOL=%CMAKE_BUILD_SHARED_LIBS% ^
      -DBUILD_python:BOOL=%BUILD_PYTHON% ^
      -DBUILD_python_layer:BOOL=%BUILD_PYTHON_LAYER% ^
      -DBUILD_matlab:BOOL=%BUILD_MATLAB% ^
      -DCPU_ONLY:BOOL=%CPU_ONLY% ^
      -DCUDNN_ROOT=C:/Projects/caffe/cudnn-8.0-windows10-x64-v5.1/cuda ^
      -C "%cd%\libraries\caffe-builder-config.cmake" ^
      "%~dp0\.."
```
另外，你也可以使用`cmake-gui.exe`。

### 构建仅仅使用CPU支持

如果未安装CUDA，Caffe将默认为CPU_ONLY构建。 如果您安装了CUDA，但想要一个仅构建CPU，则可以使用CMake选项`-DCPU_ONLY = 1`。


### Using the Python interface

推荐的Python发行版是Anaconda或Miniconda。 要成功构建python接口，您需要添加以下conda配置：
```
conda config --add channels conda-forge
conda config --add channels willyd
```
安装以下包:
```
conda install --yes cmake ninja numpy scipy protobuf==3.1.0 six scikit-image pyyaml pydotplus graphviz
```
当然，这些配置已经定义在`build_win.cmd`了。
Python安装默认是建立Python接口和Python层的。如果你想禁用Python接口或Python层，可以分别使用cmake选项` - dbuild_python_layer = 0 `和` - dbuild_python = 0 `。

### Using the MATLAB interface

按照上述步骤和使用` - dbuild_matlab=on`。改变你的MATLAB `C:\Projects\caffe\matlab`并运行下面的命令来运行测试：
```
>> caffe.run_tests()
```


## Further Details

Refer to the BVLC/caffe master branch README for all other details such as license, citation, and so on.
