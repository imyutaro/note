# GPU
CUDAなどのinstallなどのGPUの設定のMemo
2018/02/18での手順．変わっている場合があるのでその都度確認が必要．

### 目的
- **GPUに適したnvidia driverのinstall**
- **CUDA9.0をinstall**
- **CUDAのversionに対応したgcc, cuDNN, Tensorflow-gpuのinstall**

## 環境の確認
```
$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04.3 LTS"
$ uname -r
4.4.0-112-generic
```

## nvidiaのdriverのinstall
搭載しているGPUに適したdriverのinstall
[](GPUのdriverには依存関係がないので気にせずGPUに対応するdriverをinstallする)

0. グラフィックカードを刺してGPUをUbuntu16.04 LTSが認識することを確認
    ```
    lspci | grep -i nvidia
    ```
    出力結果
    ```
    04:00.0 VGA compatible controller: NVIDIA Corporation Device 1b81 (rev a1)
    ```

1. すでにdriverがinstallされていないかcheck
    ```
    dpkg -l | grep nvidia
    ```

2. installされていた場合driverを削除する
    - 普通にinstallした場合(repositoryからinstallした場合)
    ```
    # remove previous graphic driver
    sudo apt-get --purge remove "nvidia-*"
    ```
    - .runでインストールした場合
    (この方法でのuninstallをしたことがないのでこれでできるのかはわからない)
    ```
    /usr/bin/nvidia-uninstall
    ```
    ref : [Ubuntu 14.04にCuda 6.5をインストール](https://qiita.com/tshimba/items/69c17a4b42345d7bf895)

3. リポジトリの登録
    ```
    $ sudo add-apt-repository ppa:graphics-drivers/ppa
    $ sudo apt-get update
    ```
    ref : [UbuntuでNVIDIAのドライバをインストールする](https://qiita.com/hnw/items/fd84602b110a395091f4)

5. installできるdriver一覧の表示
    ```
    apt search "^nvidia-[0-9]{3}$"
    ```
    ref : [UbuntuでNVIDIAのドライバをインストールする](https://qiita.com/hnw/items/fd84602b110a395091f4)

5. 搭載しているGPUに適したdriverのinstall
    ```
    $ sudo apt-get install nvidia-390
    ```

6. 再起動
    ```
    $ sudo reboot
    ```

7. installできているかの確認
    ```
    $ nvidia-smi
    ```
    出力結果
    ```
    Sun Feb 18 13:13:44 2018
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 390.30                 Driver Version: 390.30                    |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  GeForce GTX 1070    Off  | 00000000:04:00.0 Off |                  N/A |
    |  0%   33C    P0    30W / 151W |      0MiB /  8118MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+

    +-----------------------------------------------------------------------------+
    | Processes:                                                       GPU Memory |
    |  GPU       PID   Type   Process name                             Usage      |
    |=============================================================================|
    |  No running processes found                                                 |
    +-----------------------------------------------------------------------------+
    ```

## CUDAのinstall (CUDA Toolkitのinstall)
0. tensorflowのサイトからsupportしているCUDAのversionをinstallする
    以下のtensorflow公式のTested source configurationsの項目を見てtensorflowがsupportしているCUDA Toolkitをinstallする
    [installing TensorFlow from Sources](https://www.tensorflow.org/install/install_sources#common_installation_problems)

1. すでにCUDAがinstallされていないかcheck
    ```
    dpkg -l | grep cuda
    ```

2. installされていた場合driverを削除する
    - 普通にinstallした場合(repositoryからinstallした場合)
    ```
    # remove previous cuda
    sudo apt-get --purge remove "cuda-*"
    ```
    - .runでインストールした場合
    (この方法でのuninstallをしたことがないのでこれでできるのかはわからない)
    ```
    /usr/local/cuda-*.*/bin/uninstall
    ```
    ref : [Ubuntu 14.04にCuda 6.5をインストール](https://qiita.com/tshimba/items/69c17a4b42345d7bf895)

3. debファイルのdownload
    [CUDA Toolkit 9.0 Downloads](https://developer.nvidia.com/cuda-90-download-archive)
    上記のnvidiaの公式サイトで以下の順で操作しdebファイルのdownload URLを取得する
    ```
    Linux > x86_64 > Ubuntu > 16.04 > deb (network)
    ```
    得られたURLからdebファイルをdownloadする
    ```
    sudo wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_9.0.176-1_amd64.deb
    ```

4. リポジトリの登録
    ```
    $ sudo dpkg -i cuda-repo-ubuntu1604_9.0.176-1_amd64.deb
    $ sudo apt-get update
    ```
5. CUDAのinstall
    ```
    $ sudo apt-get install cuda-9.0
    ```
    **'cuda-version'とversionの指定をしないと最新のversionがinstallされてしまうので注意**
    ref : [ImportError: libcublas.so.9.0: cannot open shared object file: No such file or directory (when trying to run object detection)](https://github.com/tensorflow/tensorflow/issues/16750)

6. pathの設定
    bashの場合.bashrcにzshの場合.zshrcに以下を書く
    ```
    # cuda path
    export CUDA_ROOT="/usr/local/cuda-9.0"
    export LIBRARY_PATH=$CUDA_ROOT/lib:$CUDA_ROOT/lib64:$LIBRARY_PATH
    export LD_LIBRARY_PATH=$CUDA_ROOT/lib64/
    # to enable nvcc command
    export PATH=$CUDA_ROOT/bin:$PATH
    ```
    ref : [GPUを使えるようにする for tensorflow - Qiita](https://qiita.com/kazetof/items/941c3463ac452b496d59)
7. nvcc commandでversionの確認
    ```
    $ nvcc --version
    ```
    出力結果
    ```
    nvcc: NVIDIA (R) Cuda compiler driver
    Copyright (c) 2005-2017 NVIDIA Corporation
    Built on Fri_Sep__1_21:08:03_CDT_2017
    Cuda compilation tools, release 9.0, V9.0.176
    ```

## cuDNNのinstall
cuDNNのインストールには，あらかじめNVIDIAのサイトでデベロッパー登録が必要になる．登録を済ませるとcuDNNをダウンロードできるようになる．
<https://developer.nvidia.com/rdp/cudnn-download>
1. packageのdownload
    以下の順で選択してpackageをdownload
    ```
    Download cuDNN v7.0.5 (Dec 5, 2017), for CUDA 9.0 > cuDNN v7.0.5 Library for Linux
    ```
2. CUDAのinstallされたdirectoryに解凍
    ```
    sudo tar xvf cudnn-9.0-linux-x64-v7.tar -C /usr/local
    cuda/include/cudnn.h
    cuda/NVIDIA_SLA_cuDNN_Support.txt
    cuda/lib64/libcudnn.so
    cuda/lib64/libcudnn.so.7
    cuda/lib64/libcudnn.so.7.0.5
    cuda/lib64/libcudnn_static.a
    ```

## gccのinstall
CUDA9.0の対応しているgccは4.8なので4.8をinstallする
```
$ sudo apt-get install gcc-4.8
```
versionの確認
```
$ gcc -v
```
出力結果
```
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/4.8/lto-wrapper
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 4.8.5-4ubuntu2' --with-bugurl=file:///usr/share/doc/gcc-4.8/README.Bugs --enable-languages=c,c++,java,go,d,fortran,objc,obj-c++ --prefix=/usr --program-suffix=-4.8 --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --with-gxx-include-dir=/usr/include/c++/4.8 --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --enable-gnu-unique-object --disable-libmudflap --enable-plugin --with-system-zlib --disable-browser-plugin --enable-java-awt=gtk --enable-gtk-cairo --with-java-home=/usr/lib/jvm/java-1.5.0-gcj-4.8-amd64/jre --enable-java-home --with-jvm-root-dir=/usr/lib/jvm/java-1.5.0-gcj-4.8-amd64 --with-jvm-jar-dir=/usr/lib/jvm-exports/java-1.5.0-gcj-4.8-amd64 --with-arch-directory=amd64 --with-ecj-jar=/usr/share/java/eclipse-ecj.jar --enable-objc-gc --enable-multiarch --disable-werror --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --with-tune=generic --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 4.8.5 (Ubuntu 4.8.5-4ubuntu2)
```

ref : [Ubuntu 14.04にCuda 6.5をインストール](https://qiita.com/tshimba/items/69c17a4b42345d7bf895)

## libcupti-devのinstall
いるかわからないけど．．．The CUDA Profiling Tools Interface もインストールしておく．
```
$ sudo apt-get -y install libcupti-dev
$ sudo reboot
```
ref : [Python: Keras/TensorFlow の学習を GPU で高速化する (Ubuntu 16.04 LTS)](http://blog.amedama.jp/entry/2017/03/13/123742)

## Bazelのinstall
これもいるかわからないけど．．．tensorflow公式のTested source configurationsの項目に書いてあるから一応．．．

```
$ sudo apt-get install openjdk-8-jdk
$ echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
$ curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
$ sudo apt-get update && sudo apt-get install bazel
$ sudo apt-get upgrade bazel
```
versionの確認
```
bazel version
```
出力結果
(verisonがtensorflowのサイトに記載されているversionと違うけどtensorflow-gpuを実行できたから関係ないかも？)
```
Extracting Bazel installation...
Build label: 0.10.1
Build target: bazel-out/k8-fastbuild/bin/src/main/java/com/google/devtools/build/lib/bazel/BazelServer_deploy.jar
Build time: Tue Mar 22 17:29:11 +50095 (1518685349351)
Build timestamp: 1518685349351
Build timestamp as int: 1518685349351
```

ref : [Ubuntu 16.04 + CUDA 9.1 + cuDNN 7.0.5 + Tensorflow 1.4.0](http://tyokota.hatenablog.com/entry/2017/12/20/170451)

## tensorflow-gpuのinstall
pipでtensorflowのgpu版 tenosrflow-gpuをinstallしないとプログラムを実行してもGPUを使ってくれない．
普通にpipでinstall
```
$ pip install tensorflow-gpu
```

----

#### kerasのCNN-MNISTで確認
sample programをdownload
```
$ curl -O https://raw.githubusercontent.com/fchollet/keras/master/examples/mnist_cnn.py
```
programに一行追加
```
$ echo 'K.clear_session()' >> mnist_cnn.py
```
実行
```
$ time python mnist_cnn.py
```
実行結果
```
/home/lab/.pyenv/versions/ueno/lib/python3.6/site-packages/h5py/__init__.py:36: FutureWarning: Conversion of the second argument of issubdtype from `float` to `np.floating` is deprecated. In future, it will be treated as `np.float64 == np.dtype(float).type`.
  from ._conv import register_converters as _register_converters
Using TensorFlow backend.
x_train shape: (60000, 28, 28, 1)
60000 train samples
10000 test samples
Train on 60000 samples, validate on 10000 samples
Epoch 1/12
2018-02-18 18:14:22.900606: I tensorflow/core/platform/cpu_feature_guard.cc:137] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA
2018-02-18 18:14:23.525207: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1105] Found device 0 with properties:
name: GeForce GTX 1070 major: 6 minor: 1 memoryClockRate(GHz): 1.7085
pciBusID: 0000:04:00.0
totalMemory: 7.93GiB freeMemory: 7.83GiB
2018-02-18 18:14:23.525239: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1195] Creating TensorFlow device (/device:GPU:0) -> (device: 0, name: GeForce GTX 1070, pci bus id: 0000:04:00.0, compute capability: 6.1)
60000/60000 [==============================] - 7s 109us/step - loss: 0.2730 - acc: 0.9171 - val_loss: 0.0595 - val_acc: 0.9813
Epoch 2/12
60000/60000 [==============================] - 5s 82us/step - loss: 0.0921 - acc: 0.9738 - val_loss: 0.0420 - val_acc: 0.9863
Epoch 3/12
60000/60000 [==============================] - 5s 82us/step - loss: 0.0683 - acc: 0.9798 - val_loss: 0.0348 - val_acc: 0.9889
Epoch 4/12
60000/60000 [==============================] - 5s 82us/step - loss: 0.0545 - acc: 0.9838 - val_loss: 0.0321 - val_acc: 0.9885
Epoch 5/12
60000/60000 [==============================] - 5s 82us/step - loss: 0.0466 - acc: 0.9852 - val_loss: 0.0334 - val_acc: 0.9889
Epoch 6/12
60000/60000 [==============================] - 5s 82us/step - loss: 0.0427 - acc: 0.9875 - val_loss: 0.0290 - val_acc: 0.9899
Epoch 7/12
60000/60000 [==============================] - 5s 82us/step - loss: 0.0384 - acc: 0.9882 - val_loss: 0.0313 - val_acc: 0.9899
Epoch 8/12
60000/60000 [==============================] - 5s 82us/step - loss: 0.0347 - acc: 0.9892 - val_loss: 0.0307 - val_acc: 0.9902
Epoch 9/12
60000/60000 [==============================] - 5s 82us/step - loss: 0.0326 - acc: 0.9902 - val_loss: 0.0270 - val_acc: 0.9912
Epoch 10/12
60000/60000 [==============================] - 5s 82us/step - loss: 0.0301 - acc: 0.9909 - val_loss: 0.0287 - val_acc: 0.9912
Epoch 11/12
60000/60000 [==============================] - 5s 82us/step - loss: 0.0283 - acc: 0.9913 - val_loss: 0.0307 - val_acc: 0.9909
Epoch 12/12
60000/60000 [==============================] - 5s 82us/step - loss: 0.0251 - acc: 0.9921 - val_loss: 0.0313 - val_acc: 0.9918
Test loss: 0.031283066169649466
Test accuracy: 0.9918
python mnist_cnn.py  54.97s user 10.13s system 103% cpu 1:02.78 total
```
ref : [Python: Keras/TensorFlow の学習を GPU で高速化する (Ubuntu 16.04 LTS)](http://blog.amedama.jp/entry/2017/03/13/123742)

----

## nvidia-smiができない問題(2018/04/08確認)
[Ubuntuでnvidiaドライバーが動作しない](https://qiita.com/bohemian916/items/7637b9b0b3494f447c03)
上記のサイトで示されているnvidia help内の
https://devtalk.nvidia.com/default/topic/1000340/cuda-setup-and-installation/-quot-nvidia-smi-has-failed-because-it-couldn-t-communicate-with-the-nvidia-driver-quot-ubuntu-16-04/4
のnvidiaが回答している通りのやり方でも解決しない．
cudaもすべて再install必要があるかも．．．

この記事も参考になるかもしれない．．．
[CUDA9.1の環境でTensorFlowをインストールする](http://sanshonoki.hatenablog.com/entry/2018/01/29/235021)

とりあえずは放置する(めんどくさいから)

### 2018年5月2日追記
- [何かをapt-get upgrade したら nvidia-smi でエラーを出すようになった](https://qiita.com/ikeyasu/items/dd32c4836a58cf697d78)のlyakaapさんのコメントを参照
- [Ubuntuでnvidiaドライバーが動作しない](https://qiita.com/bohemian916/items/7637b9b0b3494f447c03)
このサイトを参考に
ubuntuのカーネルを変えたら治った．

/etc/default/grubの
```
GRUB_DEFAULT=0
```
の部分を
```
GRUB_DEFAULT="Advanced options for Ubuntu>Ubuntu, with Linux 4.4.0-112-generic"
```
に変えて起動時のカーネルを変える．

### 2018年5月4日追記

2018年5月2日のやり方を試したがそもそも.runでdriverのinstallができないようなのでdriverのinstall自体を失敗しているせいで```nvidia-smi```ができないっぽい．ubuntuのkernelは無事変更することはできている．kernelの問題は解決しているはず．

----

reference 一覧
- [Ubuntu 14.04にCuda 6.5をインストール](https://qiita.com/tshimba/items/69c17a4b42345d7bf895)
- [ImportError: libcublas.so.9.0: cannot open shared object file: No such file or directory (when trying to run object detection)](https://github.com/tensorflow/tensorflow/issues/16750)
- [Python: Keras/TensorFlow の学習を GPU で高速化する (Ubuntu 16.04 LTS)](http://blog.amedama.jp/entry/2017/03/13/123742)
- [Ubuntu 16.04 + CUDA 9.1 + cuDNN 7.0.5 + Tensorflow 1.4.0](http://tyokota.hatenablog.com/entry/2017/12/20/170451)
- [TensorFlowのGPU環境セットアップの個人的決定版 (ubuntu 16.04)](https://qiita.com/kazetof/items/c2f8b2751d3ed9600a57)

[](https://qiita.com/kazetof/items/941c3463ac452b496d59#4cudnnのインストール)
