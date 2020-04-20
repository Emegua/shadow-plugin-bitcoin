## v0.16.0 version Bitcoind shadow plugin setup from clean slate 
Shadow-Plugin-bitcoin repository holds a Shadow plug-in that runs the Bitcoin Satoshi reference client. It can be used to run private Bitcoin networks of clients and servers on a single machine using the Shadow discrete-event network simulator. Since the current version of **Shadow-Plugin-bitcoin** has unresolved segmentation fault errors on recent versions of Ubuntu, for example 16.04 and 18.04 LTS, the repository is already archived by the owners or the repo. 
The steps provided below will give you some guidance to setup Bitcoind plugin without any segfault in ubunutu 18.04. 

### 0. Make sure that the Shadow simulator is sucessfully installed. 
If you haven't done that already, you may check [Shadow'a installation manual](https://github.com/shadow/shadow/blob/master/docs/1.1-Shadow.md) 
### 1. Clone the bitcoin plugin repository, setup basic settings. 
```
cd ~/
git clone https://github.com/shadow/shadow-plugin-bitcoin
cd shadow-plugin-bitcoin
mkdir build;cd build
sudo apt-get install -y autoconf libtool libboost-all-dev libssl-dev libevent-dev
```
### 2. Install older version openssl
The unresolved segmentation faults are found to be coused by using the recent versions of Openssl. Since using openssl-1.1.0h do not cause the segmentation fault, we need to install the older version openssl. 
```
cd ~/
wget https://www.openssl.org/source/openssl-1.1.0h.tar.gz
tar xaf openssl-1.1.0h.tar.gz
cd openssl-1.1.0h
```
The following command depends on the location where you want to install. If you install it in a different location than /home/${USER}/.shadow, make sure you change the value for prefix accordingly. 
```
./config --prefix=/home/${USER}/.shadow shared threads enable-ec_nistp_64_gcc_128 -fPIC
make depend
make
make install_sw
cd ..``
```
This installs the older verison of openssl inside the directory where we installed shadow. 
### 3. Change CMakeLists.txt in shadow-plugin-bitcoin directory.
You may use any text editor to make the change. 
240: SET(PIE_FLAGS "-shared -fPIC")
261: SET(BITCOIN_LDFLAGS "-pthread -Wl,-z,relro -Wl,-z,now")
### 4. Based on the Shadow directory, modify SHADOW_ROOT in CMakeLists.txt
ex) 23: set(SHADOW_ROOT "$ENV{HOME}/blockchain-sim/Install")
### 5. Set bitcoind with corresponding openssl library
```
cd ~/shadow-plugin-bitcoin/build
git clone https://github.com/bitcoin/bitcoin.git
cd bitcoin
git checkout v0.16.0
./autogen.sh
```
The following command depends on the location where you installed Shadow and openssl. If you install it in a different location than /home/${USER}/.shadow, make sure you some changes accordingly.  
```
PKG_CONFIG_PATH=/home/${USER}/.shadow/lib/pkgconfig LDFLAGS=-L/home/${USER}/.shadow/lib CFLAGS=-I/home/${USER}/.shadow/include ./configure --prefix=/home/${USER}/.shadow --disable-wallet
```
```
make -C src obj/build.h
make -C src/secp256k1 src/ecmult_static_context.h
git apply ../../DisableSanityCheck.patch
cd ..
```
### 6. Compile - build the actual Shadow plug-in using cmake.
```
cmake ../
make -j8
make install
cd ..
```
### 7. Remove line 23: in ~/shadow-plugin-bitcoin/resource/example.xml, since asn should be a positive value in the latest Shadow.
remove line 23: <data key="d6">0</data>
### 8. Running an experiment
```
mkdir run
cd run
mkdir -p data/bcdnode1
mkdir -p data/bcdnode2
shadow -d datadir ../resource/example.xml
```
### Reference

https://github.com/shadow/shadow-plugin-bitcoin

https://github.com/shadow/shadow-plugin-bitcoin/wiki#openssl

https://github.com/shadow/shadow-plugin-bitcoin/wiki#bitcoin
