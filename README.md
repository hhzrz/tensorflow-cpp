# tensorflow-cpp
Compile TensorFlow to C++ library for CMake project.

**NOTE** :This mannual only test under ubuntu 14.04, so other platforms are not guaranteed.

In general, for compiling tensorflow library as a c++ shared library, we need two other libraries: `protobuf` and `eigen`, and a compiling tool `bazel`.

If you are quite familiar with `docker`, you can go ahead to look into the `Dockerfile` in this repository and just ignore all the following explanation.
## Prerequisites
platform: ubuntu 14.04

## Step1: Install protobuf (version 3.4.0)
1. Install protobuf dependencies
```bash
sudo apt-get update && sudo apt-get install -y \
    autoconf \
    automake \
    ccache \
    curl \
    g++ \
    libtool \
    make \
    unzip \
    wget
```
2: Install protobuf
Be careful, we must install version 3.4.0 as current tensorflow(v1.4.0-rc1) need this version's protobuf.
```bash
git clone https://github.com/google/protobuf \
    && cd protobuf \
    && git checkout v3.4.0 \
    && ./autogen.sh \
    && ./configure --prefix=/usr \
    && sudo make install \
    && cd ..
```

## Step2: Install Bazel
1. Install JDK 8
```bash
sudo apt-get update \
    && sudo apt-get install -y software-properties-common \
    && sudo add-apt-repository -y ppa:webupd8team/java \
    && sudo apt-get update \
    && echo debconf shared/accepted-oracle-license-v1-1 select true | sudo debconf-set-selections \
    && echo debconf shared/accepted-oracle-license-v1-1 seen   true | sudo debconf-set-selections \
    && sudo apt-get install -y oracle-java8-installer
 ```
2. Add Bazel distribution URI as a package source
```bash
echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list \
    && curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
```
3. Install and update Bazel
```bash
sudo apt-get update && sudo apt-get install -y bazel && sudo apt-get upgrade -y bazel
```

## Step3: Install TensorFlow Python dependencies
```bash
sudo apt-get update && sudo apt-get install -y \
    python-numpy \
    python-dev \
    python-pip \
    python-wheel \
    swig
```

## Step4: Install TensorFlow
```bash
git clone --recurse-submodules -b v1.4.0-rc1 https://github.com/tensorflow/tensorflow.git \
    && cd tensorflow \
    && printf '\n\n\n\n\n\n\n\n\n\n\n\n' | ./configure \
    && bazel build //tensorflow:libtensorflow_cc.so \
    && sudo mkdir -p                /usr/local/include/google/tensorflow \
    && sudo cp -r bazel-genfiles/*  /usr/local/include/google/tensorflow/ \
    && sudo cp -r tensorflow        /usr/local/include/google/tensorflow/ \
    && sudo find                    /usr/local/include/google/tensorflow -type f  ! -name "*.h" -delete \
    && sudo cp -r third_party       /usr/local/include/google/tensorflow/ \
    && sudo cp bazel-bin/tensorflow/libtensorflow_cc.so /usr/local/lib \
    && sudo cp bazel-bin/tensorflow/libtensorflow_framework.so /usr/local/lib \
    && cd ..
```

## Step5: Install eigen header files
Execute the file `eigen.sh` in this repository as follow,
Please specify `<tensorflow-root>` as the root directory of the TensorFlow repository which cloned in step4,
and `/usr/local` is the directory to where you wish to install eigen.
```bash
cd tensorflow-cpp
sudo ./eigen.sh install <tensorflow-root> /usr/local
```

## Step6: Install nsync
For my case(under ubuntu 14.04), i found that i also need another library `nsync` after preparing everything.
If you came across some errors similar to the following, it aproves you also need this:
```c++
/usr/local/include/google/tensorflow/tensorflow/core/platform/default/mutex.h:25:22: fatal error: nsync_cv.h: No such file or directory
 #include "nsync_cv.h"
```
So, just install it as following:
```bash
git clone https://github.com/google/nsync.git \
    && cd nsync \
    && git checkout 839fcc53ff9be58218ed55397deb3f8376a1444e \
    && cmake -DCMAKE_INSTALL_PREFIX=/usr \
    && sudo make -j4 install
```

## Testing
After finishing above steps, everything should be ok.
Now let's build a small CMake project which run the official [tensorflow image recognition demo](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/label_image) to validate our work.

1. After clone this repository.
2. Go inside: `cd tensorflow-cpp`, and Download the model data to the `data` directory:
   ```
   curl -L "https://storage.googleapis.com/download.tensorflow.org/models/inception_v3_2016_08_28_frozen.pb.tar.gz" | tar -C ./data -xz
   ```
2. Build this project: `cmake . && make -j4`
3. Run the binary: `./hello_tensorflow`
4. If everything is ok, the result should be similar to the following:
   ```c++
   2017-10-27 08:53:28.165985: I tensorflow/core/platform/cpu_feature_guard.cc:137] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA
   2017-10-27 08:53:28.779356: I /tensorflow-cpp/main.cc:250] military uniform (653): 0.834306
   2017-10-27 08:53:28.779484: I /tensorflow-cpp/main.cc:250] mortarboard (668): 0.0218692
   2017-10-27 08:53:28.779523: I /tensorflow-cpp/main.cc:250] academic gown (401): 0.0103579
   2017-10-27 08:53:28.779551: I /tensorflow-cpp/main.cc:250] pickelhaube (716): 0.00800814
   2017-10-27 08:53:28.779732: I /tensorflow-cpp/main.cc:250] bulletproof vest (466): 0.00535088
   ```

## Conclusion
If you want to integrate tensorflow c++ library to your c++ project, after building tensorflow c++ library successfully according to the above instructions:
1. Copy all cmake mond `tensorflow_framework`.dules(Eigen_VERSION.cmake, FindEigen.cmake, FindTensorFlow.cmake) in this repository to your own project's `cmake/Modules` directory.
2. Add your project's `cmake/Modules` directory to `CMAKE_MODULE_PATH` variable.
3. Link your executable with two more libraries: `tensorflow_cc` and `tensorflow_framework`