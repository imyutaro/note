## ffmpegの使い方

## Convert gif to mp4

以下のコマンドでできる。スケールのところを変えれば動画サイズも変えることができる。

```console
ffmpeg -r 30 -i input.gif -movflags faststart -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" output.mp4
```

* Ref
  * [GIF to MP4 by ffmpeg](https://www.clas.kitasato-u.ac.jp/~fujiwara/Mathematica/GIFtoMP4.html)
  * [How to do I convert an animated gif to an mp4 or mv4 on the command line?](https://unix.stackexchange.com/questions/40638/how-to-do-i-convert-an-animated-gif-to-an-mp4-or-mv4-on-the-command-line)

