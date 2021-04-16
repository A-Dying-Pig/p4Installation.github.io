# p4 Installation Tutorial

Author: A-Dying-Pig

Last update: 2021.4.16


> This tutorial shows how to install p4 and its dependencies from scratch. `PI` (An implementation framework for a P4Runtime server), `p4c` (P4_16 reference compiler) and `behavior-model` (The reference P4 software switch), which are all the components you need to run p4 programs, will be installed. When finishing installation, you can play with [p4 tutorial](https://github.com/p4lang/tutorials).
>
> The installation is tested on Ubuntu 18.04.5 LTS. The whole installation may take 2 hours depending on network condition. At least 25 Gbytes of free disk space is needed.
>
> It is recommended to install the parts following the order listed below. The version of the packages and repos must follow the instructions. Any mismatch version may cause unsuccessful installation. 
>
> To check latest package and repo versions, visit https://github.com/p4lang/tutorials/blob/master/vm/user-bootstrap.sh and get new versions from the bash file.

## Step1: Install dependencies

1. install basic dependencies needed for p4. 

   ```shell
   sudo apt-get install -y cmake g++ git automake libtool libgc-dev bison flex libfl-dev libgmp-dev libboost-dev libboost-iostreams-dev libboost-graph-dev llvm pkg-config python python-scapy python-ipaddr python-ply tcpdump graphviz golang libpcre3-dev libpcre3 curl mininet lsb-release
   ```
   install doxygen (optional)
   
   ```shell
   sudo apt-get install -y texlive-full doxygen
   ```

2. install dependencies needed for p4 behavior model version 2

   ```shell
   git clone https://github.com/p4lang/behavioral-model.git
   cd behavioral-model
   ./install_deps.sh
   cd ..
   ```

3. install  `protobuf`

   ```shell
   sudo pip install protobuf==3.2.0
   ```

   ```shell
   git clone https://github.com/protocolbuffers/protobuf.git
   cd protobuf
   git checkout v3.2.0
   export CFLAGS="-Os"
   export CXXFLAGS="-Os"
   export LDFLAGS="-Wl,-s"
   ./autogen.sh
   ./configure --prefix=/usr
   make
   sudo make install
   sudo ldconfig
   unset CFLAGS CXXFLAGS LDFLAGS
   cd ..
   ```
   Note: when `./configure`runs with `--prefix=/usr`, `protobuf v3.2` will be installed under `/usr` directory. Otherwise `protobuf v3.2` will be installed under `/usr/local` directory. Programs under `/usr/local` have higher priority than those under `/usr` by default. 
   
   If there are multiple `protobuf`s of different versions installed, please ensure that the `protobuf v3.2` acquires highest priority to meet the requirement from PI,p4c,etc.
   ```shell
   protoc --version
   > libprotoc 3.2.0
   ```
   To delete different version of `protobuf`, delete `libprotoc*`, `protoc` in `bin`,`include`,`share`,`lib` folder.

4. install `grpc`

   ```shell
   sudo pip install grpcio
   sudo pip install psutil
   ```

   ```shell
   git clone https://github.com/google/grpc.git
   cd grpc
   git checkout v1.3.2
   git submodule update --init --recursive
   export LDFLAGS="-Wl,-s"
   make
   ```

   submodule `boringssl` and `boringssl-with-bazel` are different branches of the same repo `boringssl`.
   If the compiling process stops and the following error occurs:

   ```shell
   cc1: warnings being treated as errors
   ```

   Open and edit `Makefile`. In the line that starts with `CPPFLAGS`, remove `-Werror` flag. Save and close `Makefile`. Then run  `make clean`  and `make`. This time the compiling should work well.

   ```shell
   sudo make install
   sudo ldconfig
   unset LDFLAGS
   cd ..
   ```


5. install `sysrepo` and its dependencies `libyang`

   ```shell
   git clone https://github.com/CESNET/libyang.git
   cd libyang
   git checkout v1.0.184
   mkdir build
   cd build
   cmake ..
   make
   sudo make install
   sudo ldconfig
   cd ../..
   ```

   ```shell
   git clone https://github.com/sysrepo/sysrepo.git
   cd sysrepo
   git checkout v1.4.70
   mkdir build
   cd build
   cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_EXAMPLES=Off -DCALL_TARGET_BINS_DIRECTLY=Off ..
   make
   sudo make install
   sudo ldconfig
   cd ../..
   ```

## Step2: Install p4

1. `PI`

   ```shell
   git clone https://github.com/p4lang/PI
   cd PI
   git checkout 41358da0ff32c94fa13179b9cee0ab597c9ccbcc
   git submodule update --init --recursive
   ./autogen.sh
   ./configure --with-proto
   make
   make check
   sudo make install
   sudo ldconfig
   cd ..
   ```

2. `behavioral-model`

   ```shell
   cd behavioral-model
   git checkout b447ac4c0cfd83e5e72a3cc6120251c1e91128ab
   ./autogen.sh
   ./configure --enable-debugger --with-pi
   make
   make check
   sudo make install
   sudo ldconfig
   ```

   ```shell
   cd targets/simple_switch_grpc
   ./autogen.sh
   ./configure --with-thrift
   make
   sudo make install
   sudo ldconfig
   cd ../../..
   ```
   run `simple_switch` to check whether successfully installed `simple_switch`.

3. `p4c`

   ```shell
   git clone https://github.com/p4lang/p4c.git
   cd p4c
   git checkout 69e132d0d663e3408d740aaf8ed534ecefc88810
   git submodule update --init --recursive
   mkdir build
   cd build
   cmake ..
   make
   make check
   sudo make install
   sudo ldconfig
   cd ../..
   ```
   run `p4c --help` to check whether successfully installed `p4c`.

No error? Congratulations! You have successfully installed everything you need for p4.

Have fun with p4! 

## Step3 (optional): run a p4 program

You can now follow [p4 tutorial](https://github.com/p4lang/tutorials) to learn p4. To run `Basic Forwarding` exercise, execute:

  ```shell
  git clone https://github.com/p4lang/tutorials.git
  cd tutorials/exercises/basic
  make clean
  make run
  ```

You should now see a Mininet command prompt. Try to ping between hosts in the topology:

  ```shell
  h1 ping h2
  ```

Type `exit` to leave each xterm and the Mininet command line. Then, to stop mininet:

  ```shell
  make stop
  ```

And to delete all pcaps, build files, and logs:

  ```shell
  make clean
  ```

The ping failed because each switch is programmed according to `basic.p4`, which drops all packets on arrival. Please visit [p4 tutorial](https://github.com/p4lang/tutorials) for more information.
