# Installing Caffe


My first install on Jardines used Homebrew python 2 and Accelerate, opencv2?

Then attempt to use anaconda with python 3, openblas, opencv3.


## General Notes


### As of 4/2017 the Caffe site outlines these steps:

    brew install -vd snappy leveldb gflags glog szip lmdb

    brew tap homebrew/science
    brew install hdf5 opencv

If using Anaconda Python:

    brew edit opencv
    
and change the lines that look like the two lines below to exactly the two lines below.

    -DPYTHON_LIBRARY=#{py_prefix}/lib/libpython2.7.dylib
    -DPYTHON_INCLUDE_DIR=#{py_prefix}/include/python2.7

Note: no need to modify opencv formulat, at least not with with-python3 and without-numpy

If using Anaconda Python, HDF5 is bundled and the hdf5 formula can be skipped.

    # with Python pycaffe needs dependencies built from source
    brew install --build-from-source --with-python -vd protobuf
    brew install --build-from-source -vd boost boost-python

Note:

* current protobuf (3.2.0) does not have `--with-python` flag, only `--with-python3` and `--without-python`!
* current opencv (2.4.13.2 and 3.2.0) have `--without-numpy` (seems like what I want, but effect unclear from looking at formula) and `--without-python` option and opencv3 has additional `--with-python3` and `--with-contrib` (possibly good). Opencv does not seem able to build for python2 and python3 simultaneously (but other formula may be).


### And I did...

Install Caffe. Other dependencies already installed, and brew hdf5 not required since Anaconda provides it.

    brew install --build-from-source opencv3 --without-python --with-python3 --without-numpy

Note: Strange, but seemed to need `--without-python` to not complain about building for both versions. `--without-numpy` needed to prevent Hombrew from installing its own numpy.

This builds `/usr/local/opt/opencv3/lib/python3.5/site-packages/cv2.cpython-35m-darwin.so` (actually `/usr/local/Cellar/opencv3/3.2.0/...`). Some suggest renaming to cv2.so

    brew install --build-from-source --with-python3 protobuf
    brew install --build-from-source boost # don't think source is needed
    brew install --build-from-source --with-python3 boost-python

Note: `--with-python3` implies `--build-from-source`

Note: Anaconda does have protobuf, maybe we can use it but Caffe site shows installing with Homebrew.

Build caffe:

    make all -j4
    make test
    mke runtest

Note: builds go to build directory (actually symlink to .build_release)

but the following barf:

    make runtest # barfs
    .build_release/tools/test_all.testbin # barfs
    .build_release/tools/caffe # barfs

With the error:

    dyld: Library not loaded: @rpath/libhdf5_hl.10.dylib
      Referenced from: /Users/smathews/opt/caffe/.build_release/tools/caffe
      Reason: image not found

Note `DYLD_LIBRARY_PATH` and `DYLD_FALLBACK_LIBRARY_PATH` don't work in newer OS X (El Capitan+) because of System Integrity Protection (SIP). So setting `DYLD_FALLBACK_LIBRARY_PATH` as in Caffe instructions does not work. The problem has something to do with how Anaconda configures its libraries (`id`?) or with the Caffe Makefile (should set rpath during build, perhaps similar to how it does that with opencv)?

Checking linkage with:

    otool -L .build_release/tools/caffe

shows that:

    @rpath/libhdf5_hl.10.dylib (compatibility version 12.0.0, current version 12.0.0)
	@rpath/libhdf5.10.dylib (compatibility version 13.0.0, current version 13.0.0)

**The following fixed the problem!!!**

    install_name_tool -add_rpath ~/anaconda3/lib .build_release/tools/caffe
    install_name_tool -add_rpath ~/anaconda3/lib .build_release/test/test_all.testbin

then `make runtest` works, as does `.build_release/test/test_all.testbin` when run directly. Note that since we didn't add the correct rpaths to the individual test binaries they can't be run directly.

The solution was from this suggestion:

    install_name_tool -add_rpath '/usr/local/anaconda/lib' /usr/local/caffe/.build_release/tools/caffe

ref [#2720](https://github.com/BVLC/caffe/issues/2720)

Other suggestion, but it seems more elegant to add the necessary rpath instead of hard coding the library path but this would likely work, too:

    install_name_tool -change @rpath/./libhdf5_hl.10.dylib ~/anaconda/lib/libhdf5_hl.10.dylib .build_release/tools/caffe
    install_name_tool -change @rpath/./libhdf5.10.dylib ~/anaconda/lib/libhdf5.10.dylib .build_release/tools/caffe

ref [Compiling Caffe under Mac OS X with Anaconda dependencies](http://akmetiuk.com/posts/2016-03-29-compiling-caffe.html)

##### PyCaffe

Now with everything working:

    make pycaffe # builds to caffe/python
    make distribute # packages everything into distribute dir


May instructions suggest setting `PYTHONPATH`, but I added `~/opt/caffe/distribute/python` to my `homebrew.pth` (in `.local/...`)

Still I get the following error on `import caffe`:

    ImportError: dlopen(/Users/smathews/opt/caffe/distribute/python/caffe/_caffe.so, 2): Library not loaded: @rpath/libcaffe.so.1.0.0
    Referenced from: /Users/smathews/opt/caffe/distribute/python/caffe/_caffe.so
    Reason: image not found

So I had to

    install_name_tool -add_rpath ~/opt/caffe/lib caffe/distribute/python/_caffe.so
    
and then it worked.





## Helpful Tips

Show what libraries and executable or library link to (useuful for making sure python extensions are linked to right place):

    otool -L <file> # print the shared libraries used
    otool -l <file> # print the load commands (incl. rpaths as LC_RPATH)
    
    otool -D <shared library> # print shared library id name
    
    otool -hv <file> # show Mach header symbolically (shows filetype, etc.)

See `install_name_tool`, `libtool`

And maybe also `dlopen`

remember..`env`/`printenv`/`set`



---

# Caffe Installation

- [Installing Caffe the right way](http://installing-caffe-the-right-way.wikidot.com/)

- [Deepdream Installation](https://gist.github.com/robertsdionne/f58a5fc6e5d1d5d2f798) script


## Install Caffe on Jardines

This previous install used brew python and Accelerate I think.

### 8/2016

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


### 12/2016

cblas.h is here:

    /System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/Headers/cblas.h
    
    /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/Headers/cblas.h

actual path:

    /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/System/Library/Frameworks/Accelerate.framework/Versions/Current/Frameworks/vecLib.framework/Versions/Current/Headers

Compiled by setting BLAS_INCLUDE in Makefile, since sdk version was set wrong:
    
    /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.sdk/System/Library/Frameworks/Accelerate.framework/Versions/A/Frameworks/vecLib.framework/Versions/A/Headers

but it appears that the problem has been fixed upstream and correct version is found like:

    ls /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/ | sort | tail -1
