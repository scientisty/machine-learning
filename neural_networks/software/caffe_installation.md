# Install Caffe on Jardines

## 8/2016

Install Cafee using home-brew python, follow directions on caffe site to the T.

Aside, this is how you

	otool -L /usr/local/Cellar/boost/1.61.0_1/lib/libboost_system.dylib
	otool -L /usr/local/Cellar/boost-python/1.61.0/lib/libboost_python.dylib

etc

    PYTHON_INCLUDE := 
    /usr/local/lib/python2.7/site-packages/numpy/core/include/ 
    /usr/local/Cellar/python/2.7.12/Frameworks/Python.framework/Versions/2.7/include/python2.7

    PYTHON_LIB := 
    /usr/local/Cellar/python/2.7.12/Frameworks/Python.framework/Versions/2.7/lib



* 10.9 has vecLib (-framework vecLib)
* 10.10 has accelerate (-framework accelerate) which has vecLib inside
* 10.11 (El Capitan)

---

## 12/2016

cblas.h is here:

    /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/Headers/cblas.h
    
    /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/Headers/cblas.h

actual path:

    /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/System/Library/Frameworks/Accelerate.framework/Versions/Current/Frameworks/vecLib.framework/Versions/Current/Headers

Compiled by setting BLAS_INCLUDE in Makefile, since sdk version was set wrong:
    
    /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.sdk/System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/Headers

but it appears that the problem has been fixed upstream and correct version is found like:

    ls /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/ | sort | tail -1
