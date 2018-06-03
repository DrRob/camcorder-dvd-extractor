# camcorder-dvd-extractor
Extracts and transcodes clips from Canon DC50 camcorder DVDs.

I've never had the time or inclination to edit my camcorder clips into movies to watch.
Instead I prefer to mix the clips in with photos taken at the same occasion and view them
like I would photos.

I already put the timestamps of photos into their filenames to make it a bit easier to 
organise them, and wanted to do the same with my camcorder clips, and also to convert them 
from MPEG-2 to h264 to save disk space.

My camcorder, a Canon DC50, writes to mini DVDs in one of two formats - DVD-Video or DVD-VR.
The latter is only available on rewritable disks, which is a shame because it's much easier to 
extract timestamped clips from that format.

For the DVD-VR format, I found this tool: https://github.com/pixelb/dvd-vr that extracts a 
"vob" file (MPEG-2) for each clip found on the disk, naming the file with the date and time, 
which is perfect.

For the DVD-Video format, it's nowhere near as straightforward.  The only place I could find
the timestamps was in the DVD menus that the camcorder generates, in bitmap text form.  Long
story short, I've worked out a process to extract the menu bitmaps (after patching a tool, because
it missed the first page of menus), use OCR to convert the date and time to text, then use
another tool to enumerate the clips (chapters), and finally extract, transcode and rename the
files.

## Usage

It can probably read the files direct from the DVD, but I always make a copy to the hard
drive first.

You will need to finalise DVD-Video disks first.

I find that the Camcorder itself can read the disks with fewer errors that the DVD drive in my
computer.

The container requires three volumes to be mounted:

* `/dvd` - where the `VIDEO_TS.IFO` or `VR_MOVIE.VRO`, etc. files are located
* `/mp4` - where the resulting MP4 files will be written to
* `/log` - where log files will be written

    docker run --rm \
      -v /mnt/sdc1/miniDVD/A:/dvd \
      -v /mnt/sdb1/miniDVD/A/mp4:/mp4 \
      -v /mnt/sdb1/miniDVD/A/log:/log \
      camcorder-dvd-extractor

You should see something like this:

    docker@serf:~$ docker run --rm -v /mnt/sdc1/miniDVD.bak/A:/dvd -v /mnt/sdb1/miniDVD/A/mp4:/mp4 -v /mnt/sdb1/miniDVD/A/log:/log camcorder-dvd-extractor
    Found a DVD-VR format disk:
    Transcoding 20121225_034142
    Transcoding 20121225_035148

or this:

    docker@serf:~$ docker run --rm -v /mnt/sdc1/miniDVD.bak/1:/dvd -v /mnt/sdb1/miniDVD/1/mp4:/mp4 -v /mnt/sdb1/miniDVD/1/log:/log camcorder-dvd-extractor
    Found a DVD-Video format disk:
    Transcoding 20110412_1012
    Transcoding 20110412_1104
    Transcoding 20110412_1107
    Transcoding 20110412_1108
    Transcoding 20110412_1110
    Transcoding 20110412_1111
    Transcoding 20110412_1113
    Transcoding 20110412_1113-2
    Transcoding 20110412_1115
    Transcoding 20110412_1205
    Transcoding 20110412_1321
    Transcoding 20110412_1327
    Transcoding 20110412_1329
    Transcoding 20110412_1330
    Transcoding 20110412_1334
    Transcoding 20110412_1400
    Transcoding 20110412_1442
    Transcoding 20110412_1500
    Transcoding 20110412_1502
    Transcoding 20110412_1503
    Transcoding 20110412_1503-2
    Transcoding 20110412_1603
    Transcoding 20110412_1606
    Transcoding 20110412_1606-2
    Transcoding 20110412_1607
    Transcoding 20110412_1608
    Transcoding 20110412_1611
    Transcoding 20110412_1612
    Transcoding 20110412_1613
    Transcoding 20110412_1614
    Transcoding 20110412_1615
    Transcoding 20110412_1617

The filenames follow the pattern `YYYYMMDD_HHMM` (DVD-Video) or `YYYYMMDD_HHMMSS` (DVD-VR) 
so that they appear in chronological order if sorted alphanumerically.
If two clips have the same timestamp (unlikely in the DVD-VR case because it includes seconds)
then an index is appended (see three examples above).

## Third-party components

You can see all this in the Dockerfile, but to summarise:

* It's based on the phusion minimal Ubuntu base image: https://github.com/phusion/baseimage-docker
* `bash`
* `lsdvd`
* `tcextract`
* `subtitleripper`, with a patch applied otherwise it misses the first page of menu items.  I tried contacting the author about this but got no reply
* `pamcut` and `pnminvert`
* `gocr`
* `awk`
* `dvd-vr` from https://github.com/pixelb/dvd-vr
* `handbrake`
