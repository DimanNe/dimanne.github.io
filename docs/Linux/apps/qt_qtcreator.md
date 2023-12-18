title: Qt & QtCreator

# **Qt & QtCreator**

## **Install all qt dev libraries**

`sudo apt install (apt-cache search --names-only qt | cut -d ' ' -f 1 | rg  "libqt5" | rg -v -- '-gles')`

## **Build Qt from source**

```bash linenums="1"
apt install libclang-12-dev libclang1-12
git clone --recursive https://code.qt.io/qt-creator/qt-creator.git
cd qt-creator/
git checkout 4.15
cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_PREFIX_PATH=$HOME/devel/qt5-bin ../qt-creator/ # -DCMAKE_BUILD_TYPE=RelWithDebInfo
export LD_LIBRARY_PATH=$HOME/devel/qt5-bin/lib/:$LD_LIBRARY_PATH
make -j
```

## **Build QtCreator from source**

```bash linenums="1"
apt install build-essential perl python git '^libxcb.*-dev' libx11-xcb-dev libglu1-mesa-dev \
            libxrender-dev libxi-dev libxkbcommon-dev libxkbcommon-x11-dev flex bison gperf \
            libicu-dev libxslt-dev ruby libxcursor-dev libxcomposite-dev libxdamage-dev \
            libxrandr-dev libxtst-dev libxss-dev libdbus-1-dev libevent-dev libfontconfig1-dev \
            libcap-dev libpulse-dev libudev-dev libpci-dev libnss3-dev libasound2-dev \
            libegl1-mesa-dev gperf bison nodejs libasound2-dev libgstreamer1.0-dev \
            libgstreamer-plugins-base1.0-dev libgstreamer-plugins-good1.0-dev \
            libgstreamer-plugins-bad1.0-dev
git clone git://code.qt.io/qt/qt5.git
cd qt5
git checkout 5.15.2
perl init-repository
cd ../ && mkdir build-qt5 && cd build-qt5/ # find ../build-qt5/ -mindepth 1 -delete
../qt5/configure -opensource -release -confirm-license -prefix $HOME/devel/qt5-bin
nice -n 19 make -j25
sudo make install
```
