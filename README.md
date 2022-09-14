# What is this?
This is a config files for mpv player. MPV player itself have no settings in its GUI, as it is a minimalist video player, that normally would be used via command line and traditionally have no GUI at all except OSD, all controlled via keyboard and mouse interaction. It does have however a pseudo GUI, which is jut and OSD and an OSC (on screen controls).

# How to use
Put the `portable_config` folder into the the same directory of your mpv.exe, this will overwrite the existing `portable_config`, if you do not use portable mode, config will be located at `%AppData%\mpv\mpv`. Refer to [Files on Windows](https://mpv.io/manual/stable/#files-on-windows) in the mpv manual.

# What to do afterwards?
Change the mpv.conf to your own preference, these are a few tweaks I've made to mine.

## Change your decoder
Mine is set to use NVDEC (NVidia Decoder) `hwdec=nvdec`, if you want to use hardware decoder but not sure which one, use `hwdec=auto-safe` (which I have put for this config). As far as I know the old adage is Software Decoder is better than hardware in terms of quality at cost of performance, though I think more modern hardware decoder such as nvdec are virtually as good as software decoder,  and it uses its own separate chip inside the gpu which results in negligible performance loss from both cpu and gpu. It used to that using hardware decoder might result in loss of quality due to poor color management, information loss, etc, but more modern method of hardware decoder have gone better. But if your CPU is capable, you can never be wrong when using software decoder. In the past hardware decoder is used only when your CPU isn't strong enough to do the decoding.
To use Software Decoder, set hwdec to no `hwdec=no` or just comment out or disable the line, as by default mpv will use software decoder, as it is mentioned in its manual:

>In theory, hardware decoding does not reduce video quality (at least for the codecs h264 and HEVC). However, due to restrictions in video output APIs, as well as bugs in the actual hardware decoders, there can be some loss, or even blatantly incorrect results.
>
>...
>
>In general, it's very strongly advised to avoid hardware decoding unless absolutely necessary, i.e. if your CPU is insufficient to decode the file in questions. If you run into any weird decoding issues, frame glitches or discoloration, and you have --hwdec turned on, the first thing you should try is disabling it.

P.S.: non-copy method of hardware decoding usually only works in openGL mode (`gpu-api=opengl`), Vulkan also work, however iirc Vulkan is still buggy with mpv.

> Note: Most non-copy methods only work with the OpenGL GPU backend. Currently, only the vaapi, nvdec and cuda methods work with Vulkan

## How to Check Hardware Decoder is Actually running
If you own an NVidia GPU and use their drivers you could use `nvidia-smi` to check the utilisation of your harware decoder inside the gpu.
Open terminal and type the following:

    nvidia-smi dmon

You can then look at the `dec` column, which will tell decoder utilisation

A more accurate method is by using mpv player log-file function. It will create a log file, akin to using verbose (-v) in command line. I've put the argument in the mpv.conf in General Section, remove the # in front of the argument to uncomment the argument. It will the create log file into logs folder inside your mpv.exe directory. Open this file and look for the sentence `using hardware decoding (nvdec)`^\*\*\*^ just few couple lines below the lines that tells you It have opened the video file (last line that mention the video file name).

\*\*\* or which ever hwdec methods you choose


## Scroll to change volume
By default mpv use scroll wheel to seek, however I'm used to using scroll wheel to change audio quickly rather than pressing 0 and 9 in my keyboard.
If you do not want scroll wheel to change audio volume, and instead use it back as a scroll to seek, delete the `input.conf` in `portable_config`.

## Subtitle set to always be the same size at fullscreen
I've made it so that subtitle—when the video is played at full screen—will render the subtitle relative to the display size instead of the video resolution height. However this come at a cost, subtitle will scale with window height, meaning in windowed mode, as the window of the player is now following the height of the video (instead of the height of the display when full screen), thus subtitle size become variable when using in windowed mode.
Most video players out there (MPC, VLC, etc) scale subtitle by video resolution height, which change subtitle size depending on video height. For example when playing a 16:9 video, the subtitle will be larger when full screen-ed compared to video that's using cinemascope ratio 2.40:1. This is caused by the way mpv renders subtitle (probably other video players too).
According to the manual, this is how the `sub-font-size` is implemented.
> The unit is the size in scaled pixels at a window height of 720. The actual pixel size is scaled with the window height: if the window height is larger or smaller than 720, the actual size of the text increases or decreases as well.
The "scaled pixels" in the first line actually refer to typeface unit "pixel" which is not a arbitrary unit but instead a fixed real physical lenght of 1/96 of an inch. If you set the font size to 36 on a 720p height video, then set the player to play at 100% size, the font height will be 36/96 of an inch.
In my 108 ppi display, I use font size 36 which after real physical measurement, is 2.54% of my screen height. Which a subtitle X-height target mentioned in this [article](https://www.md-subs.com/saa-subtitle-font-size) by Max Deryagin.

## Playback
I set it to loop the file forever, the loop playlist argument in redundant in this case, but i just left it on.

## Tone Mapping
I use mobius, as I prefer color accuracy over others. The summary for tone mapping algorithm:
- Hable for detail preservation
- Mobius for color accuracy
- Reinhard for overall brightness preservation
- BT.2390 for standard
- Linear
- Clip
Just try all of them, and check which one do you like. Read the [manual](https://mpv.io/manual/stable/#options-tone-mapping) for more information

## Subtitle Loading Path
I've set the `sub-auto=all` which allow mpv to automatically load all subtitle within the same directory of the video file. This will allow `sub-file-paths` to load subtitle regardless of the subtitle filename. This is useful ass many times subtitle are put within Subtitle/Subs/Sub subfolder, without the file name i.e. "English.srt", and it will not load, as with fuzzy mode, the subtitle still requires to have parts of the video filename.
This leads to a problem if you put videos and subtitle files into a big folder instead of putting them separately to their own subfolder for each movie title, it will load all the subtitle not belonging (as the method is set to all). So if you're this guy, change the `sub-auto=` to `sub-auto=fuzzy` or `sub-auto=exact` this will load subtitle which contain the video name.

## Typeface of the UI and Subtitle
I've set mine to Helvetica for subtitle, Inter for the UI (OSC and OSD), and Input+JetBrainsMono for the Stat screen (when pressing i). Find these 3 separate lines to change the font.

        sub-font="Helvetica"
        
        script-opts=stats-font="Inter",stats-font_mono="JetBrains Mono"
        
        osd-font='Inter'
        
 ## Subtitle in letterbox or just video
 `sub-use-margins=no` will disallow subtitle to be displayed within the letterbox, and only within the video frame. This is purely preference. Personally I don't like putting the subtitle so far below, while it retains the video from being blocked by text, it requires more eye travel from the video and text.
P.S. This only will work when the video does not have blackbar harcoded into the video. Some video contains black bar as part of the video itself and mpv can't detect that.

## Change screenshot Format
Screenshot is saved in png format, to change it to JPEG, find the SCREENSHOT section and change `screenshot-format=png` to `screenshot-format=jpeg`

I've set mpv to save screenshot into Screenshots folder inside your Pictures folder (`screenshot-directory="~\Pictures\Screenshots"), change the path inside the "" to change where mpv saves screenshots.
