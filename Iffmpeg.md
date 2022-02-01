ffmpeg
======
## install
+ sample
```sh
sudo apt-get install yasm libx264-dev
	./configure --enable-shared --enable-libx264 --enable-gpl --prefix=/TOOLS/software/ffmpeg
```
+ complix
```sh
sudo apt install libgsm1-dev  libmp3lame-dev  libx265-dev libwebp-dev libwavpack-dev libvpx-dev libvorbis-dev libzvbi-dev  libspeex-dev libmysofa-dev libshine-dev libopus-dev libtheora-dev libfontconfig1-dev libopencv-dev  libopencv-core-dev libopenjp2-7-dev  libopenmpt-dev libpulse-dev librsvg2-dev librubberband-dev  libsnappy-dev libsoxr-dev libssh-dev libtwolame-dev libxvidcore-dev libzmq3-dev libopenal-dev libcdio-dev libsdl2-dev
```


```sh
./configure --prefix=/home/solar/ffmpeg-4.1 --extra-version=0ubuntu0.18.04.1 --toolchain=hardened --libdir=/usr/lib/x86_64-linux-gnu --incdir=/usr/include/x86_64-linux-gnu --enable-gpl --disable-stripping --enable-avresample --enable-avisynth  --enable-ladspa --enable-libass --enable-libbluray --enable-libbs2b --enable-libcaca  --enable-libfontconfig --enable-libfreetype --enable-libfribidi --enable-libgme --enable-libgsm --enable-libmp3lame --enable-libmysofa --enable-libopenjpeg --enable-libopenmpt --enable-libopus --enable-libpulse --enable-librubberband --enable-librsvg --enable-libshine --enable-libsnappy --enable-libsoxr --enable-libspeex --enable-libssh --enable-libtheora --enable-libtwolame --enable-libvorbis --enable-libvpx --enable-libwavpack --enable-libwebp --enable-libx265 --enable-libxml2 --enable-libxvid --enable-libzmq --enable-libzvbi  --enable-openal  --enable-sdl2 --enable-libdc1394 --enable-libdrm  --enable-chromaprint --enable-frei0r  --enable-libx264 --enable-shared --enable-pthreads
```



## 参数

```
	 -encoders
	 -s 720*576
	 -target dvd
	 	-target参数匹配行业标准，参数值可以是vcd、svcd、dvd、dv、dv50等，可能还需要加上电视制式作为前缀（pal-、ntsc-或film-)
	 -vf "rotate=90*PI/180"
	 	我发现用手机拍的视频中，有些是颠倒的，我想让它顺时针旋转90度。这时候，可以使用-vf参数加入一个过滤器
	 -ss 2 -t 10 -i
	 	从第2秒的地方开始，往后截取10秒钟
	 -vcodec copy -an
	 	-vcodeccopy的意思是对源视频不解码，直接拷贝到目标文件；-an的意思是将源文件里的音频丢弃
	 -crf
	 	Constant Rate Factor
	 	For x264 your valid range is 0-51
		The range of the quantizer scale is 0-51: where 0 is lossless, 23 is default, and 51 is worst possible. A lower value is a higher quality and a subjectively sane range is 18-28. Consider 18 to be visually lossless or nearly so: it should look the same or nearly the same as the input but it isn't technically lossless
		
	 -preset
	 	是一個選項集合，這設定一編碼速度來決定壓縮比。速度越慢則會得到更好的壓縮編碼效率 (畫質-位元率比 或 畫質-檔案大小比)。也就是說，若你設定一個目標位元率或是檔案大小，則越慢的 Preset 將會得到更好的輸出品質。而對於設定一個恆定品質 (CRF) 或是恆定量化值 (QP)，你可以透過選擇更慢的 Preset 來簡單的節省位元率 (也就是得到更小的檔案大小)。
	一般而言是使用你所能忍受的最慢 Preset。目前 Presets 依速度遞減排序是: ultrafast, superfast, veryfast, faster, fast, medium, slow, slower, veryslow, placebo。該預設 Preset 是 medium。忽略 placebo 因為它會比 veryslow 浪費更多時間而且效果差異太小。
```
## 视频信息

```
ffprobe -i xx.mp4 
```

## 锯齿
### -deinterlace

## 样例
```sh
ffmpeg -y -i g7-2009-kbs.rm -c:v libx264 -strict -2 -deinterlace  -vb 3300000 usb-target/g7-2009-kbs.deinterlace.mp4
```
## webm to mp4 
```sh
ffmpeg -fflags +genpts -i 吸毒打击.webm   吸毒打击.mp4
```

## 屏幕直播

```
sudo apt-get install libx11-dev libXtst-dev yasm libx264-dev
./configure --prefix=/TOOLS/software/ffmpeg --enable-x11grab --enable-gpl --enable-libx264
./ffmpeg  -f x11grab -s 1366*768 -r 15  -i :0.0 -vcodec libx264 -preset ultrafast -pix_fmt yuv420p -s 720*576 -f flv rtmp://localhost/myrtmp/mystream
```



## 压缩

自动进行目录的压缩

```sh
#!/bin/bash
set -x
_local=$0
root_dir=$1
out_dir=$2
function main(){
	IFS=$(echo -en "\n\b")
	echo -en $IFS
	files=`find $root_dir|grep .mp4$`
	for i in $files
	do
		file_check $i
		echo $out_dir
		out_file=`echo $i|sed "s|$root_dir|$out_dir|g"`
		mkdir -p `dirname $out_file`
		out_base=$out_file.base.ts
		run_ffmpeg $i $out_base 720*576 25 2300000 96000 4:3
		echo $out_base >>$out_dir/file_ok.txt
		out_fhd=$out_file.fhd.ts 
		run_ffmpeg $i $out_fhd 1920*1080 25 7200000 96000 16:9
		echo $out_fhd >>$out_dir/file_ok.txt
		out_hd=$out_file.hd.ts 
		run_ffmpeg $i $out_hd 1280*720 25 3300000 96000 16:9
		echo $out_hd >>$out_dir/file_ok.txt
	done
}
function file_check(){
file=$1
if [ -f "$file" ]
then
	echo "$file found."
else
	echo "$file not found."
	echo $file >> $out_dir/file_error.txt
fi
}
function run_ffmpeg(){
	in=$1
	out=$2
	resolution=$3
	fps=$4
	vb=$5
	ab=$6
	aspect=$7
#	ffmpeg  -i $in -s $resolution  -deinterlace -strict -2 -threads 0 -r $fps -vb $vb $out
	ffmpeg	-i $in -c:v libx264 -s $resolution -aspect $aspect -r $fps -b:v $vb -b:a $ab -c:a mp2 $out
}
main
```

