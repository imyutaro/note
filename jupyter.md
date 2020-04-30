# Jupyter関連

Jupyter Notebookで使えるKernel(言語)は以下のページに書いてある．

* [Jupyter kernels · jupyter/jupyter Wiki](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels)

## RをJupyter Notebook上で実行できるようにする

RKernelのRepositoryのinstallationをもとにインストールすればいい．CRANが必要．

`install.packages('IRkernel')`をrで実行すれば`CRAN`のインストールが始まる．

> CRAN is a network of ftp and web servers around the world that store identical, up-to-date, versions of code and documentation for R. Please use the CRAN mirror nearest to you to minimize network load.
ref : [The Comprehensive R Archive Network](https://cran.r-project.org/)

とのことなのでMirrorはJapanを選択．

* References
  * [IRkernel/IRkernel: R kernel for Jupyter](https://github.com/IRkernel/IRkernel)
