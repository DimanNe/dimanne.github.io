title: Video editing

# **Video editing**

## **Tools**

### ffmpeg
List available encoders: `ffmpeg -encoders`

### DaVinci
* [Tutorial / Intro](https://www.youtube.com/watch?v=63Ln33O4p4c&ab_channel=JustinBrown-PrimalVideo).
* [Changing speed](https://www.youtube.com/watch?v=MgSIDRvgwIg&t=250s&ab_channel=John%E2%80%99sFilms).


### Mencoder

Reduce resolution of video:

```bash
mencoder -aid 1 -slang eng  -ovc lavc -vf scale -zoom -xy 640 -oac mp3lame -lameopts cbr:br=160 Click.2006.720p.BluRay.ac3.x264-HqF.mkv -o click.avi
```

### recordmydesktop

Record video from desktop:
```bash
recordmydesktop --no-sound --v_bitrate=2000000 --width=1500 --height=1380 --fps 29 \
                --full-shots --no-wm-check --workdir /mnt/freedata/home/Void -o myfile
```


## **Codecs**

[Main doc](https://blog.frame.io/2017/02/15/choose-the-right-codec/). 
[Comparison table](https://blog.frame.io/2017/02/13/compare-50-intermediate-codecs/).

There are several groups of codecs for different purposes:

##### Capture codecs
Codecs that are used by device that records a video:

Most consumer-grade devices use H.264 codec because it is highly compressed. But it is not the best codec
for editing because it is very computation-heavy.

##### Edit codecs
Codecs that used for editing, intermediate codecs: `DNxHD`, `DNxHR`, `ProRes` and `Cineform`. If your camera
is not using them directly, you will need to convert source video to one of them.

More info:
* This means that they're designed to be used to trans-code footage from other sources into a form that's easy
for video editing programs to work with while maintaining quality. So, unlike most h.264 implementations, they
focus on low CPU usage, retaining as much detail as possible, and an ability to be re-compressed several times
without significant loss in quality. They have larger bitrates than consumer video codecs, but they still
represent a significant space savings over fully uncompressed video.

##### Export codec
A highly compressed codec, such as H.264, should be fine.




## **Intermediate codecs**

### DNxHR
DNxHR is for resolutions bigger than 1080p (2K, 4K, and 8K) (as opposed to DNxHD).

```bash
ffmpeg -ss 00:04:00 -i out.mp4 -c:v dnxhd -profile:v dnxhr_hq  -pix_fmt yuv422p     out_dnxhd_dnxhr.mov
ffmpeg -ss 00:04:00 -i out.mp4 -c:v dnxhd -profile:v dnxhr_hqx -pix_fmt yuv422p10le out_dnxhd_dnxhr_444.mov
ffmpeg -ss 00:04:00 -i out.mp4 -c:v dnxhd -profile:v dnxhr_444 -pix_fmt yuv444p10le out_dnxhr_444.mov
```

Instead of `_hq` in the name of profile, you can specify:

* `LB` --- Low Bandwidth. 8-bit 4:2:2 (yuv422p).
* `SQ` --- Standard Quality. 8-bit 4:2:2 (yuv422p). Suitable for delivery format.
* `HQ` --- High Quality. 8-bit 4:2:2 (yuv422p).
* `HQX` --- High Quality. 10-bit 4:2:2 (yuv422p10le). UHD/4K Broadcast-quality delivery.
* `444` --- Finishing Quality. 10-bit 4:4:4 (yuv444p10le). Cinema-quality delivery.

Output format container for DNxHD is typically `MXF` or `MOV`.

Script:

```fish
for f in (ls -1 | grep -v "for-edit-")
   echo $f
   ffmpeg -i $f -c:v dnxhd -profile:v dnxhr_hqx -pix_fmt yuv422p10le for-edit-$f.mov
end
```


### libxvid
ffmpeg -ss 00:04:00 -i out.mp4 -c:v libxvid -q:v 2  out_libxvid.mov


### ProRes

* PROXY (bad): `-c:v prores_ks -profile:v 0 -qscale:v 5 -vendor ap10 -pix_fmt yuv422p10le -acodec pcm_s16le`
* LT: `-c:v prores_ks -profile:v 1 -qscale:v 5 -vendor ap10 -pix_fmt yuv422p10le -acodec pcm_s16le`
* SQ: `-c:v prores_ks -profile:v 2 -qscale:v 5 -vendor ap10 -pix_fmt yuv422p10le -acodec pcm_s16le`
* 422 HQ: `-c:v prores_ks -profile:v 3 -qscale:v 5 -vendor ap10 -pix_fmt yuv422p10le -acodec pcm_s16le`


## **Output / export codecs**

### h.265 (new/better codec).
See the [guide](https://trac.ffmpeg.org/wiki/Encode/H.265).

You can see available options with:

```bash
$ ffmpeg -h encoder=libx265

    Supported pixel formats: yuv420p yuvj420p yuv422p yuvj422p yuv444p yuvj444p gbrp yuv420p10le yuv422p10le yuv444p10le gbrp10le yuv420p12le yuv422p12le yuv444p12le gbrp12le gray gray10le gray12le
libx265 AVOptions:
  -crf               <float>      E..V...... set the x265 crf (from -1 to FLT_MAX) (default -1)
  -qp                <int>        E..V...... set the x265 qp (from -1 to INT_MAX) (default -1)
  -forced-idr        <boolean>    E..V...... if forcing keyframes, force them as IDR frames (default false)
  -preset            <string>     E..V...... set the x265 preset
  -tune              <string>     E..V...... set the x265 tune parameter
  -profile           <string>     E..V...... set the x265 profile
  -x265-params       <dictionary> E..V...... set the x265 configuration using a :-separated list of key=value parameters
```


Add `-ss 00:01:40` before `-i` for faster tests, add `-loglevel debug` for more info.

`ffmpeg -i LakeDistrict2.mov -c:v libx265 -crf 22 -preset veryslow -pix_fmt yuv420p -c:a aac -b:a 128k LakeDistrict2_libx265_crf_22_veryslow_yuv420p.mkv`

Works: `ffmpeg -ss 00:01:40 -i LakeDistrict2.mov -c:v libx265 -crf 21 -preset veryfast -pix_fmt yuvj420p -c:a aac -b:a 128k LakeDistrict2_h.265_crf_28_yuvj420p_veryfast.mkv`


### h.264 (old codec).
See the [guide](https://trac.ffmpeg.org/wiki/Encode/H.264#crf).

`ffmpeg -i LakeDistrict2.mov -c:v libx264 -crf 16 -preset veryslow -c:a copy LakeDistrict2_crf_16_veryslow.mkv`

* Pixel format is not supported on phones and mplayer (`-pix_fmt yuv420p` is needed?)
* You might want to to consider adding `-x264-params opencl=true`, but see below
    * But you you probably do NOT want to add `-hwaccel auto`. Hardware encoders typically generate output of
      significantly lower quality than good software encoders like x264, but are generally faster and do not
      use much CPU resource. (That is, they require a higher bitrate to make output with the same perceptual
      quality, or they make output with a lower perceptual quality at the same bitrate.)
