---
layout: post
title: "在 mac osx 上安装 caffe"
date: 2014-12-18
---

[Caffe](https://github.com/BVLC/caffe): a fast framework for deep learning.

## 修改配置文件 `Makefile.config`
{% highlight diff %}
diff --git a/Makefile.config.example b/Makefile.config.example
index e11db51..f688112 100644
--- a/Makefile.config.example
+++ b/Makefile.config.example
@@ -5,7 +5,7 @@
# USE_CUDNN := 1

# CPU-only switch (uncomment to build without GPU support).
-# CPU_ONLY := 1
+CPU_ONLY := 1

# To customize your choice of compiler, uncomment and set the following.
# N.B. the default for Linux is g++ and the default for OSX is clang++
@@ -31,7 +31,7 @@ CUDA_ARCH := -gencode arch=compute_20,code=sm_20 \
# atlas for ATLAS (default)
# mkl for MKL
# open for OpenBlas
-BLAS := atlas
+BLAS := open
# Custom (MKL/ATLAS/OpenBLAS) include and lib directories.
# Leave commented to accept the defaults for your choice of BLAS
# (which should work)!
@@ -41,12 +41,13 @@ BLAS := atlas
# This is required only if you will compile the matlab interface.
# MATLAB directory should contain the mex binary in /bin.
# MATLAB_DIR := /usr/local
-# MATLAB_DIR := /Applications/MATLAB_R2012b.app
+MATLAB_DIR := /Applications/MATLAB_R2014b.app

# NOTE: this is required only if you will compile the python interface.
# We need to be able to find Python.h and numpy/arrayobject.h.
-PYTHON_INCLUDE := /usr/include/python2.7 \
-		/usr/lib/python2.7/dist-packages/numpy/core/include
+PYTHON_INCLUDE := /usr/local/include/python2.7 \
+		/usr/local/lib/python2.7/dist-packages/numpy/core/include \
+		/usr/local/Cellar/python/2.7.9/Frameworks/Python.framework/Headers
# Anaconda Python distribution is quite popular. Include path:
# Verify anaconda location, sometimes it's in root.
# ANACONDA_HOME := $(HOME)/anaconda
@@ -55,12 +56,12 @@ PYTHON_INCLUDE := /usr/include/python2.7 \
# $(ANACONDA_HOME)/lib/python2.7/site-packages/numpy/core/include \

# We need to be able to find libpythonX.X.so or .dylib.
-PYTHON_LIB := /usr/lib
-# PYTHON_LIB := $(ANACONDA_HOME)/lib
+PYTHON_LIB := /usr/local/Cellar/python/2.7.9/Frameworks/Python.framework/Versions/2.7/lib/
+# PYTHON_LIB := $(HOME)/anaconda/lib

# Whatever else you find you need goes here.
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include
-LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib
+LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib $(MATLAB_DIR)/bin/maci64 /usr/lib

# Uncomment to use `pkg-config` to specify OpenCV library paths.
# (Usually not necessary -- OpenCV libraries are normally installed in one of the above $LIBRARY_DIRS.)
{% endhighlight %}

在编译过程中遇见链接 `python` 错了，链接到了系统的 `python`, 经过 Google 和修改才解决，这就是个坑。

## `make` 时发现 gtest 报错了
{% highlight diff %}
diff --git a/src/gtest/gtest.h b/src/gtest/gtest.h
index 3143bd6..3dcc36c 100644
--- a/src/gtest/gtest.h
+++ b/src/gtest/gtest.h
@@ -50,6 +50,7 @@

#ifndef GTEST_INCLUDE_GTEST_GTEST_H_
#define GTEST_INCLUDE_GTEST_GTEST_H_
+#define GTEST_USE_OWN_TR1_TUPLE 1

#include <limits>
#include <vector>
{% endhighlight %}

## 最后
{% highlight bash %}
make
make pycaffe
make runtest
{% endhighlight %}

到这里就算完成了。
