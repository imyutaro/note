# Python

## Install python from source in Ubuntu

### Install build tools and libraries

```bash
sudo apt update
sudo apt install build-essential libbz2-dev libdb-dev \
  libreadline-dev libffi-dev libgdbm-dev liblzma-dev \
  libncursesw5-dev libsqlite3-dev libssl-dev \
  zlib1g-dev uuid-dev tk-dev
```

### Download source

Download source from [python.org](https://www.python.org/downloads)

```bash
wget https://www.python.org/ftp/python/3.7.5/Python-3.7.5.tgz
```

tar

```bash
tar xzf Python-3.7.5.tgz
```

### Build

```bash
cd Python-3.7.5
./configure --enable-shared
make
sudo make install
sudo sh -c "echo '/usr/local/lib' > /etc/ld.so.conf.d/custom_python3.conf"
sudo ldconfig
```

`python3` command is installed to `/usr/local/bin`. Now you can use `python3` command.

### Install pip

```bash
# download install script
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
# run script
python3 get-pip.py
```

* References
  * [Ubuntu環境のPython - python.jp](https://www.python.jp/install/ubuntu/index.html)

## `if __name__=="__main__":` について

Pythonの`if __name__=="__main__":`に書いたものはそのファイル自体を実行したときに実行されるが，globalなものとして実行されてしまう(if文の中が実行されているのでglobalにｋ実行されている)．
つまり，`if __name__=="__main__":`ないである変数xを宣言した場合，そのファイル内の他の関数の中で値を渡していないにもかかわらず使用することができる．
なので`main()`関数を別で定義して，`if __name__=="__main__":`ないで`main()`を実行したほうが良い．

## datetime

今日の日付を取得．

```python
import datetime

today = datetime.date.today()
print(type(today)) # => <class 'datetime.date'>
print(type(f"today")) # => <class 'str'>
print(f"{today}")

today = str(today)
print(type(today)) # => <class 'str'>
```

現在時刻を取得．

```python
import datetime

c_time = datetime.datetime.now().strftime("%H-%M-%S")
print(type(c_time)) # => <class 'str'>
print(f"{c_time}")
```

* References
  * [datetime — Basic date and time types — Python 3.8.0 documentation](https://docs.python.org/3/library/datetime.html)
