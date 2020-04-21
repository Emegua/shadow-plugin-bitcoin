# V0.16.0 version Bitcoind shadow plugin setup from clean slate 
[**Shadow-Plugin-bitcoin**](https://github.com/shadow/shadow-plugin-bitcoin) repository holds a Shadow plug-in that runs the Bitcoin Satoshi reference client. It can be used to run private Bitcoin networks of clients and servers on a single machine using the Shadow discrete-event network simulator. Since the current version of **Shadow-Plugin-bitcoin** has unresolved segmentation fault errors on recent versions of Ubuntu, for example 16.04 and 18.04 LTS, the repository is already archived by the owners or the repo. 

**The steps provided below will give you some guidance to setup Bitcoind plugin without any segfault in ubunutu 18.04.**

### 0. Make sure that the Shadow simulator is sucessfully installed. 
If you haven't done that already, you may check [Shadow's installation manual](https://github.com/shadow/shadow/blob/master/docs/1.1-Shadow.md) 
### 1. Clone the bitcoin plugin repository, setup basic settings. 
```
cd ~/
git clone https://github.com/shadow/shadow-plugin-bitcoin
cd shadow-plugin-bitcoin
mkdir build;cd build
sudo apt-get install -y autoconf libtool libboost-all-dev libssl-dev libevent-dev
```
### 2. Install older version openssl
The unresolved segmentation faults are found to be caused by using the recent versions of Openssl. Since using openssl-1.1.0h do not cause the segmentation fault, we need to install the older version openssl. 
```
cd ~/
wget https://www.openssl.org/source/openssl-1.1.0h.tar.gz
tar xaf openssl-1.1.0h.tar.gz
cd openssl-1.1.0h
```
The following command depends on the location where you want to install openssl-1.1.0h. If you install it in a different location than /home/${USER}/.shadow, make sure you change the value for prefix accordingly. 
```
./config --prefix=/home/${USER}/.shadow shared threads enable-ec_nistp_64_gcc_128 -fPIC
make depend
make
make install_sw
cd ..
```
This installs the older verison of openssl inside the directory where you installed shadow. 
### 3. Change CMakeLists.txt in shadow-plugin-bitcoin directory, and based on the shadow direcotry, modify SHADOW_ROOT in CMakeLists.txt
This can be made by applying a patch as shown below.
#### 3.1. If your shadow is installed in the $HOME/Install directory: 
```
cd ~/shadow-plugin-bitcoin
curl https://raw.githubusercontent.com/Emegua/shadow-plugin-bitcoin/master/update_CMakeLists.patch | git apply
```
#### 3.2. If your shadow is installed in the $HOME/.shadow directory:
```
cd ~/shadow-plugin-bitcoin
curl https://raw.githubusercontent.com/Emegua/shadow-plugin-bitcoin/master/update_CMakeLists_shadow.patch | git apply
```
#### 3.3. If your shadow is installed in any other directory, you may use step 3.2, but you may need to manually modify SHADOW_ROOT in CmakeLists.txt as shown below.
ex) 23: set(SHADOW_ROOT "$ENV{HOME}/blockchain-sim/.shadow")

What the patch basically does is edit somelines in CMakeLists.txt file as follows.
240: ~~SET(PIE_FLAGS "-fPIE")~~

240: SET(PIE_FLAGS "-shared -fPIC")

261: ~~SET(BITCOIN_LDFLAGS "-pthread -Wl,-z,relro -Wl,-z,now -pie")~~

261: SET(BITCOIN_LDFLAGS "-pthread -Wl,-z,relro -Wl,-z,now")

ex) 23: set(SHADOW_ROOT "$ENV{HOME}/Install")
### 5. Set bitcoind with corresponding openssl library
```
cd ~/shadow-plugin-bitcoin/build
git clone https://github.com/bitcoin/bitcoin.git
cd bitcoin
git checkout v0.16.0
./autogen.sh
```
The following command depends on the location where you installed Shadow and openssl. If you install it in a different location than /home/${USER}/.shadow, make sure you make some changes accordingly.  
```
PKG_CONFIG_PATH=/home/${USER}/.shadow/lib/pkgconfig LDFLAGS=-L/home/${USER}/.shadow/lib CFLAGS=-I/home/${USER}/.shadow/include ./configure --prefix=/home/${USER}/.shadow --disable-wallet
```
For example, if shadow is installed in $HOME/Install direcotry instead of /home/${USER}/.shadow, you may use the following command:
```
PKG_CONFIG_PATH=/home/${USER}/Install/lib/pkgconfig LDFLAGS=-L/home/${USER}/Install/lib CFLAGS=-I/home/${USER}/Install/include ./configure --prefix=/home/${USER}/Install --disable-wallet
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
```
curl https://raw.githubusercontent.com/Emegua/shadow-plugin-bitcoin/master/updateExampleXml.patch | git apply
```
### 8. Run an experiment
```
mkdir run
cd run
mkdir -p data/bcdnode1
mkdir -p data/bcdnode2
shadow -d datadir ../resource/example.xml
```
### References

https://github.com/shadow/shadow-plugin-bitcoin

https://github.com/shadow/shadow-plugin-bitcoin/wiki#openssl

https://github.com/shadow/shadow-plugin-bitcoin/wiki#bitcoin
