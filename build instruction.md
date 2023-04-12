
## Building MySQL 8.x
Below versions of MySQL are available in respective distributions at the time creation of these build instructions:
*   RHEL (8.4, 8.6, 8.7, 9.0, 9.1) have `8.0.30`
*   Ubuntu 18.04 has `5.7.41`
*   Ubuntu (20.04, 22.04, 22.10) has `8.0.32`

The instructions provided below specify the steps to build [MySQL](https://dev.mysql.com/) version 8.0.32 on Linux on IBM Z for the following distributions:
*   RHEL (7.8, 7.9, 8.4, 8.6, 8.7, 9.0, 9.1)
*   SLES (12 SP5, 15 SP3, 15 SP4)
*   Ubuntu 18.04

_**General Notes:**_
*   _When following the steps below please use a standard permission user unless otherwise specified._
*   _A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writable directory anywhere you'd like to place it._
## Build MySQL
### Step 1: Build using script
If you want to build MySQL using manual steps, go to Step 2.
Use the following commands to build MySQL using the build [script](https://github.com/linux-on-ibm-z/scripts/tree/master/MySQL). Please make sure you have wget installed.
```bash
wget -q https://raw.githubusercontent.com/linux-on-ibm-z/scripts/master/MySQL/8.0.32/build_mysql.sh
# Build MySQL
bash build_mysql.sh   [Provide -t option for executing build with tests]
```
If the build completes successfully, go to STEP 4. In case of error, check `logs` for more details or go to STEP 2 to follow manual build steps.
### Step 2: Install the dependencies
```
export SOURCE_ROOT=/<source_root>/
export PATCH_URL=https://raw.githubusercontent.com/linux-on-ibm-z/scripts/master/MySQL/8.0.32/patch
```
#### 2.1) Install the required dependencies 
*   RHEL (7.8, 7.9)
    ```shell
    sudo yum install -y bison bzip2 gcc gcc-c++ git hostname ncurses-devel pkgconfig tar wget zlib-devel doxygen devtoolset-11-gcc devtoolset-11-gcc-c++ devtoolset-11-binutils net-tools
    sudo yum install -y xz python2 python-yaml     # For Duktape
    export PATH=/opt/rh/devtoolset-11/root/usr/bin:/usr/local/bin:$PATH
    ```
*   RHEL (8.4, 8.6, 8.7)
    ```shell
    sudo yum install -y bison bzip2 gcc gcc-c++ git hostname ncurses-devel openssl openssl-devel pkgconfig tar wget zlib-devel doxygen cmake diffutils rpcgen make libtirpc-devel libarchive gcc-toolset-11-gcc gcc-toolset-11-gcc-c++ gcc-toolset-11-binutils net-tools
    sudo yum install -y xz python2 python2-pyyaml     # For Duktape
    source /opt/rh/gcc-toolset-11/enable
    ```
*   RHEL (9.0, 9.1)
    ```shell
    sudo yum install -y bison bzip2 bzip2-devel gcc gcc-c++ git xz xz-devel hostname ncurses ncurses-devel openssl openssl-devel pkgconfig tar wget zlib-devel doxygen cmake diffutils rpcgen make libtirpc-devel libarchive tk-devel gdb gdbm-devel sqlite-devel readline-devel libdb-devel libffi-devel libuuid-devel libnsl2-devel net-tools
    ```
*   SLES 12 SP5
    ```shell
    sudo zypper install -y cmake bison git ncurses-devel pkg-config gawk doxygen tar gcc11 gcc11-c++ libtirpc-devel 
    sudo zypper install -y python python2-PyYAML      # For Duktape
    sudo update-alternatives --install /usr/bin/cc cc /usr/bin/gcc-11 11
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 11
    sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-11 11
    sudo update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++-11 11
    sudo ln -sf /usr/bin/gcc /usr/bin/s390x-linux-gnu-gcc
    sudo ln -sf /usr/bin/cpp-11 /usr/bin/cpp
    ```
*   SLES (15 SP3, 15 SP4)
    ```
    sudo zypper install -y cmake bison gcc11 gcc11-c++ git hostname ncurses-devel openssl openssl-devel pkg-config gawk doxygen libtirpc-devel rpcgen tar wget net-tools-deprecated
    sudo zypper install -y xz python python2-PyYAML   # For Duktape
    sudo update-alternatives --install /usr/bin/cc cc /usr/bin/gcc-11 11
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 11
    sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-11 11
    sudo update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++-11 11
    sudo ln -sf /usr/bin/gcc /usr/bin/s390x-linux-gnu-gcc
    sudo ln -sf /usr/bin/cpp-11 /usr/bin/cpp
    ```
*   Ubuntu 18.04
    ```
    sudo apt-get update
    sudo apt-get install -y bison cmake gcc-8 g++-8 git hostname libncurses-dev libssl-dev make openssl pkg-config doxygen tar wget net-tools
    sudo apt-get install -y python python-yaml        # For Duktape
    sudo ln -sf /usr/bin/gcc-8 /usr/bin/gcc
    ```
#### 2.2) Install the additional dependencies
* Build Python 2.7.18 (Only on RHEL 9.x)    
  ```
    cd $SOURCE_ROOT
    wget https://www.python.org/ftp/python/2.7.18/Python-2.7.18.tar.xz
    tar -xvf Python-2.7.18.tar.xz
    cd $SOURCE_ROOT/Python-2.7.18
    ./configure --prefix=/usr/local --exec-prefix=/usr/local
    make
    sudo make install
    sudo ln -sf /usr/local/bin/python /usr/bin/python2
    python2 -V
    # Install pip
    curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py 
    sudo python2 get-pip.py
    # Install PyYAML for Duktape
    python2 -m pip install --upgrade pip setuptools --force-reinstall
    python2 -m pip install PyYAML
  ```
* Build OpenSSL (Only on RHEL 7.x and SLES 12 SP5)    
  ```
  cd $SOURCE_ROOT
  wget https://www.openssl.org/source/openssl-1.1.1k.tar.gz --no-check-certificate
  tar -xzf openssl-1.1.1k.tar.gz
  cd openssl-1.1.1k
  ./config --prefix=/usr/local --openssldir=/usr/local
  make
  sudo make install
  sudo mkdir -p /usr/local/etc/openssl
  sudo wget https://curl.se/ca/cacert.pem --no-check-certificate -P /usr/local/etc/openssl
  export LDFLAGS="-L/usr/local/lib/ -L/usr/local/lib64/"
  LD_LIBRARY_PATH=/usr/local/lib/:/usr/local/lib64/${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
  export LD_LIBRARY_PATH
  PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:/usr/local/lib64/pkgconfig${PKG_CONFIG_PATH:+:${PKG_CONFIG_PATH}}
  export PKG_CONFIG_PATH
  LD_RUN_PATH=/usr/local/lib:/usr/local/lib64${LD_RUN_PATH:+:${LD_RUN_PATH}}
  export LD_RUN_PATH
  export CPPFLAGS="-I/usr/local/include/ -I/usr/local/include/openssl"
  export SSL_CERT_FILE=/usr/local/etc/openssl/cacert.pem
  sudo /usr/sbin/ldconfig /usr/local/lib64   # Only on RHEL 7.x
  sudo ldconfig /usr/local/lib64    # Only on SLES 12 SP5
  ```
* Build Cmake (Only on RHEL 7.x)  
  ```
  cd $SOURCE_ROOT
  wget https://github.com/Kitware/CMake/releases/download/v3.20.3/cmake-3.20.3.tar.gz --no-check-certificate
  tar -xvzf cmake-3.20.3.tar.gz
  cd cmake-3.20.3
  ./bootstrap
  make
  sudo make install
  cmake --version
  ``` 
* Build Duktape 
  ```
  cd $SOURCE_ROOT
  wget https://duktape.org/duktape-2.7.0.tar.xz     # Use wget with `--no-check-certificate` on RHEL 7.x and SLES 12 SP5
  tar xfJ duktape-2.7.0.tar.xz
  cd duktape-2.7.0
  # Add s390x related code changes
  sed -i '/\/* Durango (Xbox One)/i \/* s390x *\/\n#if defined(__s390x__)\n#define DUK_F_S390X \n#endif' src/duk_config.h
  sed -i '/#elif defined(DUK_F_SPARC32)/i #elif defined(DUK_F_S390X)\n\/* --- s390x --- *\/\n#define DUK_USE_ARCH_STRING "s390x"\n#define DUK_USE_BYTEORDER 3\n#undef DUK_USE_PACKED_TVAL\n#define DUK_F_PACKED_TVAL_PROVIDED'  src/duk_config.h 
  sed -i 's/duk_memcpy_unsafe((void \*) p, (const void \*) ins, (size_t) (ins_end - ins));/duk_memcpy_unsafe((void *) p, (const void *) ins, (size_t) (ins_end - ins) * 4);/' src-input/duk_api_bytecode.c 
  sed -i 's/p += (size_t) (ins_end - ins);/p += (size_t) (ins_end - ins) * 4;/' src-input/duk_api_bytecode.c
  python2 tools/configure.py --output-directory src-duktape
  ```
### Step 3: Product Build - MySQL
#### 3.1) Download the MySQL source code
```shell
cd $SOURCE_ROOT
git clone https://github.com/mysql/mysql-server.git
cd mysql-server
git checkout mysql-8.0.32
# Copy Duktape files
rm $SOURCE_ROOT/mysql-server/extra/duktape/duktape-2.7.0/src/*
cp $SOURCE_ROOT/duktape-2.7.0/src-duktape/* $SOURCE_ROOT/mysql-server/extra/duktape/duktape-2.7.0/src/
```  
*   <i>This patch `/storage/ndb/include/portlib/mt-asm.h`  fixes the build failure due to rmb() and wmb()</i>

    ```shell
       wget --no-check-certificate $PATCH_URL/mt-asm.h.diff
       git apply mt-asm.h.diff
    ```
*   <i>This patch `/storage/ndb/src/common/portlib/NdbHW.cpp`  fixes the test NdbHW-t failure </i>

    ```shell
       wget --no-check-certificate $PATCH_URL/NdbHW.cpp.diff
       git apply NdbHW.cpp.diff
       mkdir build
       cd build
    ```
#### 3.2) Configure, Build and Install MySQL
*   RHEL 7.x
    ```shell
    wget https://boostorg.jfrog.io/artifactory/main/release/1.77.0/source/boost_1_77_0.tar.bz2
    cmake .. -DDOWNLOAD_BOOST=0 -DWITH_BOOST=$SOURCE_ROOT/mysql-server/build -DWITH_SSL=system -DCMAKE_C_COMPILER=/opt/rh/devtoolset-11/root/bin/gcc -DCMAKE_CXX_COMPILER=/opt/rh/devtoolset-11/root/bin/g++
    make
    sudo make install   
    ```
*   SLES 12 SP5
    ```shell
    cmake .. -DDOWNLOAD_BOOST=1 -DWITH_BOOST=. -DWITH_SSL=system -DCMAKE_C_COMPILER=/usr/bin/gcc -DCMAKE_CXX_COMPILER=/usr/bin/g++
    make
    sudo make install   
    ```
*   RHEL(8.x, 9.x), SLES (15 SP3, 15 SP4) and Ubuntu
    ```
    cmake .. -DDOWNLOAD_BOOST=1 -DWITH_BOOST=. -DWITH_SSL=system
    make
    sudo make install
    ```
    **Note:** For more MySQL source configuration options, please visit their official [guide](https://dev.mysql.com/doc/refman/8.0/en/source-configuration-options.html).
#### 3.3) Run unit tests (Optional)
* Use the below command to run the whole test suite:
```
cd $SOURCE_ROOT/mysql-server/build
export LD_PRELOAD=$SOURCE_ROOT/mysql-server/build/library_output_directory/libprotobuf-lite.so.3.19.4:$LD_PRELOAD   # To fix routertest test failures
make test
```
* Use the below command to run an individual test case (for example `routertest_component_routing_splicer`):
```
ctest -VV -R routertest_component_routing_splicer
```
**Note:** 
  * _Below test cases might fail if IPv6 is not enabled:_
    * _routertest_harness_net_ts_internet_
    * _routertest_component_http_server_
### Step 4: Post installation Setup and Testing (Optional)
Refer to this [guide](https://dev.mysql.com/doc/refman/8.0/en/postinstallation.html) for the Postinstallation Setup and Testing overview.
### References:
- [https://dev.mysql.com/doc/refman/8.0/en/](https://dev.mysql.com/doc/refman/8.0/en/) - MySQL 8.0 Reference Manual  
- [https://dev.mysql.com/doc/refman/8.0/en/source-installation.html](https://dev.mysql.com/doc/refman/8.0/en/source-installation.html) - Installing MySQL from Source
