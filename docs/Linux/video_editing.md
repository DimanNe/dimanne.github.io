title: Video editing

# **Video editing**


## **DaVinci**

* `sudo apt install pulseaudio pulseaudio-utils pulseaudio-module-x11 libasound2-plugins`
* `LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libglib-2.0.so.0 /opt/resolve/bin/resolve`
* [Tutorial / Intro](https://www.youtube.com/watch?v=63Ln33O4p4c&ab_channel=JustinBrown-PrimalVideo).
* [Changing speed](https://www.youtube.com/watch?v=MgSIDRvgwIg&t=250s&ab_channel=John%E2%80%99sFilms).

Below are stages of workflow in Davinci.

**Re-encode**

See intermediate codecs below.

**New Project**

* Create new project.
* Choose resolution (UHD) & fps (60fps) of timeline (this is the most important part)
* Choose same or lower resolution in Video Monitoring.

**Import media, organise, mark**

* On the "Media tab" import media.
* **Merge multiple physical videos into a single logical one**: put them on timeline (in the Edit tab) -> select ->
  right click -> New Compound Clip
* Organise & Cut
    * **Use bins**, for example
        * a bin for clips on timeline vs not-yet-on-timelne
        * bin with timelapses
        * etc..
    * **Use markers**: select clip and press `M`. Press it a second time to adjust colour/text.
    * **Use the blade tool**
* **Use multiple timelines**: At the upper-left corner of timeline, click "Timeline view options" button, press
  "Stacked timelines", at the upper-right corner, a new button with plus sign appears, press it.

**Cutting**

* **Temporarily speed up video**: j (speeds up backward), k, l (speeds up forward).
* **Move cursor to the edges** of clips and adjust their length/start/end. You can always expand later (nothing is removed).
* **Use both viewports** to adjust start/end with high precision.
* **Use Blade tool**: `B`. Note, blade tool does not remove anything, you can you expand again the clip later.
* **Cut from curr pos to the right**: `Ctrl+Shift+]`
* **Cut from curr pos to the left**: `Ctrl+Shift+[`


## **Other Tools**

### ffmpeg
List available encoders: `ffmpeg -encoders`


### Mencoder

Reduce resolution of video:

```bash
mencoder -aid 1 -slang eng  -ovc lavc -vf scale -zoom -xy 640 -oac mp3lame -lameopts cbr:br=160 Click.2006.720p.BluRay.ac3.x264-HqF.mkv -o click.avi
```



## **Recording**

* OBS Studio
* SimpleScreenRecorder: a very user-friendly software which has a wizard-like setup process.
* Kazam
* VokoScreen
* Peek
* ~~recordmydesktop~~

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
# cd to a dir, where you have ./source dir and ./intermediate dir
for s in (ls -1 ./source)
  set s ./source/$s
  set d ./intermediate/(basename $s).mov
  echo "$s => $d"
  ffmpeg -i $s -c:v dnxhd -profile:v dnxhr_hqx -pix_fmt yuv422p10le $d;
end
```


### libxvid
ffmpeg -ss 00:04:00 -i out.mp4 -c:v libxvid -q:v 2  out_libxvid.mov


### ProRes

* PROXY (bad): `#!bash -c:v prores_ks -profile:v 0 -qscale:v 5 -vendor ap10 -pix_fmt yuv422p10le -acodec pcm_s16le`
* LT: `#!bash -c:v prores_ks -profile:v 1 -qscale:v 5 -vendor ap10 -pix_fmt yuv422p10le -acodec pcm_s16le`
* SQ: `#!bash -c:v prores_ks -profile:v 2 -qscale:v 5 -vendor ap10 -pix_fmt yuv422p10le -acodec pcm_s16le`
* 422 HQ: `#!bash -c:v prores_ks -profile:v 3 -qscale:v 5 -vendor ap10 -pix_fmt yuv422p10le -acodec pcm_s16le`


## **Output / export codecs**

### h.265 (new/better codec).
See the [guide](https://trac.ffmpeg.org/wiki/Encode/H.265).

??? info "You can see available options with: `ffmpeg -h encoder=libx265`"

    ```bash
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


* Use `-ss 00:00:20 -t 00:40:00` before `-i` to specify start & duration of the clip to encode (for faster tests)

* Add `-loglevel debug` for more info.

* **Choose crf**: No difference between `-crf` 10, 14 18, 19, 20 (21 & 22 barely differs, but you cannot tell which
  one is worse).
    * **24 is still perfectly fine**, but you can tell the difference.
    * 25 --- this is where compression *starts* to get noticeable.

* **Example of command**:

    ```fish title="do not forget to remove -ss and -t"
    set basename "Swiss-Part-1";       \
    set codec libx265;                 \
    set crf 24;                        \
    set preset medium;                 \
    set pix_fmt yuv420p;               \
    ffmpeg -ss 00:00:40 -t 00:00:10 -i $basename.mov -c:v $codec -crf $crf -preset $preset -pix_fmt $pix_fmt -c:a aac -b:a 192k \
    $basename-$codec-crf=$crf-preset=$preset-pix_fmt=$pix_fmt.mkv
    ```

* **Reduce priority**: `sudo renice 19 -p (pgrep -x ffmpeg)`.



### VP9 (produces much larger output)


Single-pass:

```fish
set basename "Swiss-Part-3.2";
set codec libvpx-vp9;
set crf 20;         # Lower is better quality. 0–63
set deadline best;  # realtime, good, best
set passes single
set pix_fmt yuv420p;
ffmpeg -ss 00:00:20 -t 00:00:40 -i $basename.mov -c:v $codec -crf $crf  -b:v 0 -deadline $deadline -cpu-used 0 -pix_fmt $pix_fmt -c:a libopus -b:a 192k -row-mt 1 $basename-$codec-crf=$crf-$passes-pass-deadline=$deadline-pix_fmt=$pix_fmt.webm
```

Two-pass:

```fish
set basename "Swiss-Part-3.2";
set codec libvpx-vp9;
set crf 20;         # Lower is better quality. 0–63
set deadline best;  # realtime, good, best
set passes 2
# set pix_fmt yuv420p;
ffmpeg -ss 00:00:20 -t 00:00:40 -i $basename.mov -c:v $codec -b:v 0 -crf $crf -pass 1 -deadline $deadline -cpu-used 0 -an -f null /dev/null
ffmpeg -ss 00:00:20 -t 00:00:40 -i $basename.mov -c:v $codec -b:v 0 -crf $crf -pass 2 -deadline $deadline -cpu-used 0 -c:a libopus -b:a 192k -row-mt 1 $basename-$codec-crf=$crf-$passes-pass-deadline=$deadline-pix_fmt=$pix_fmt.webm

```


### h.264 (old codec).
See the [guide](https://trac.ffmpeg.org/wiki/Encode/H.264#crf).

`#!bash ffmpeg -i LakeDistrict2.mov -c:v libx264 -crf 16 -preset veryslow -c:a copy LakeDistrict2_crf_16_veryslow.mkv`

* Pixel format is not supported on phones and mplayer (`-pix_fmt yuv420p` is needed?)
* You might want to to consider adding `-x264-params opencl=true`, but see below
    * But you you probably do NOT want to add `-hwaccel auto`. Hardware encoders typically generate output of
      significantly lower quality than good software encoders like x264, but are generally faster and do not
      use much CPU resource. (That is, they require a higher bitrate to make output with the same perceptual
      quality, or they make output with a lower perceptual quality at the same bitrate.)
