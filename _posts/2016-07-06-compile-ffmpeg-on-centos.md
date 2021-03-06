---
layout: post
title: "Compile FFmpeg on Centos"
description: ""
category: 
tags: [centos,ffmpeg]
last_updated: Wed, 28 Aug 2016 16:36:08 +0700
---
{% include JB/setup %}

This guide is based on a minimal installation of the latest CentOS release,
and will provide a local, non-system installation of FFmpeg with support for
several extarnal encoding libraries. These instructions should also work for
recent Red Hat Enterprise Linux (RHEL) and Fedora. This is non-invasive guide
and undoing all steps is simple and is shown at the end of this page.


## Get the dependencies

> The ❯ sign indicates that the command should be executed as superuser or
> root, $ sign for normal user.
{: .note_quote }

These are required compiling, but you can remove them when you are done if you
prefer (except `make`, it should be installed by default and many things depend
on it).

{% highlight bash %}
❯ yum install autoconf automake cmake freetype-devel gcc gcc-c++ git libtool make mercurial nasm pkgconfig zlib-devel
{% endhighlight %}


In your home directory, make a new directory to put all of the source code
into: `$ mkdir ~/ffmpeg_sources`

## Compilation and Installation

- **Yasm**: Yasm is an assembler used by x264 and FFmpeg.

{% highlight bash %}
$ cd ~/ffmpeg_sources
$ git clone --depth 1 git://github.com/yasm/yasm.git
$ cd yasm
$ autoreconf -fiv
$ ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin"
$ make && make install
$ make distclean
{% endhighlight %}

- **libx264:** H.264 video encoder. See
[H.264 Encoding Guide](https://goo.gl/Y44My4) for more information and usage
samples.

Requires `ffmpeg` to be configured with `--enable-gpl` and `--enable-libx264`

{% highlight bash %}
$ cd ~/ffmpeg_sources
$ git clone --depth 1 git://git.videolan.org/x264
$ cd x264
$ PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --enable-static
$ make && make install
$ make distclean
{% endhighlight %}

- **libx265:** H.265/HEVC video encoder. See the
[H.265 Encoding Guide](https://goo.gl/gByzia) for more information and usage
samples.
 
Requires `ffmpeg` to be configured with `--enable-gpl` and `--enable-libx265`
  
{% highlight bash %}
$ cd ~/ffmpeg_sources
$ hg clone https://bitbucket.org/multicoreware/x265
$ cd ~/ffmpeg_sources/x265/build/linux
$ cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="$HOME/ffmpeg_build" -DENABLE_SHARED:bool=off ../../source
$ make && make install
{% endhighlight %}

- **libfdk_aac:** AAC audio encoder.

Requires `ffmpeg` to be configured with `--enable-libfdk-aac` (and
`--enable-nonfree` if you also included `--enable-gpl`)

{% highlight bash %}
$ cd ~/ffmpeg_sources
$ git clone --depth 1 git://git.code.sf.net/p/opencore-amr/fdk-aac
$ cd fdk-aac
$ autoreconf -fiv
$ ./configure --prefix="$HOME/ffmpeg_build" --disable-shared
$ make && make install
$ make distclean
{% endhighlight %}


- **libmp3lame:** Mp3 audio encoder. Requires `ffmpeg` to be configured with
`--enable-libmp3lame`.

{% highlight bash %}
$ cd ~/ffmpeg_sources
$ curl -L -O http://downloads.sourceforge.net/project/lame/lame/3.99/lame-3.99.5.tar.gz
$ tar xzvf lame-3.99.5.tar.gz
$ cd lame-3.99.5
$ ./configure --prefix="$HOME/ffmpeg_build" --bindir="$HOME/bin" --disable-shared --enable-nasm
$ make && make install
$ make distclean
{% endhighlight %}

- **libopus:** Opus audio decoder and encoder. Requires `ffmpeg` to be
configured with `--enable-libopus`.


{% highlight bash %}
$ cd ~/ffmpeg_sources
$ git clone http://git.opus-codec.org/opus.git
$ cd opus
$ autoreconf -fiv
$ ./configure --prefix="$HOME/ffmpeg_build" --disable-shared
$ make && make install
$ make distclean
{% endhighlight %}


- **libogg:** Ogg bitstream library. Required by `libtheora` and `libvorbis`.

{% highlight bash %}
$ cd ~/ffmpeg_sources
$ curl -O http://downloads.xiph.org/releases/ogg/libogg-1.3.2.tar.gz
$ tar xzvf libogg-1.3.2.tar.gz
$ cd libogg-1.3.2
$ ./configure --prefix="$HOME/ffmpeg_build" --disable-shared
$ make && make install
$ make distclean
{% endhighlight %}


- **libvorbis:** Vorbis audio encoder. Requires libogg. Requires `ffmpeg` to
be configured with `--enable-libvorbis`.

{% highlight bash %}
$ cd ~/ffmpeg_sources
$ curl -O http://downloads.xiph.org/releases/vorbis/libvorbis-1.3.4.tar.gz
$ tar xzvf libvorbis-1.3.4.tar.gz
$ cd libvorbis-1.3.4
$ LDFLAGS="-L$HOME/ffmeg_build/lib" CPPFLAGS="-I$HOME/ffmpeg_build/include" ./configure --prefix="$HOME/ffmpeg_build" --with-ogg="$HOME/ffmpeg_build" --disable-shared
$ make && make install
$ make distclean
{% endhighlight %}


- **libvpx:** VP8/VP9 video encoder. Requires `ffmpeg` to be configured with
`--enable-libvpx`.

{% highlight bash %}
$ cd ~/ffmpeg_sources
$ git clone --depth 1 https://chromium.googlesource.com/webm/libvpx.git
$ cd libvpx
$ ./configure --prefix="$HOME/ffmpeg_build" --disable-examples
$ make && make install
$ make clean
{% endhighlight %}


- **FFmpeg**

{% highlight bash %}
$ cd ~/ffmpeg_sources
$ git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
$ cd ffmpeg
$ PKG_CONFIG_PATH="$HOME/ffmpeg_build/lib/pkgconfig" ./configure \
--prefix="$HOME/ffmpeg_build" --extra-cflags="-I$HOME/ffmpeg_build/include" \
--extra-ldflags="-L$HOME/ffmpeg_build/lib" --bindir="$HOME/bin" \
--pkg-config-flags="--static" --enable-gpl --enable-nonfree \
--enable-libfdk-aac --enable-libfreetype --enable-libmp3lame --enable-libopus \
--enable-libvorbis --enable-libvpx --enable-libx264 --enable-libx265
$ make && make install
$ make distclean
$ hash -r
{% endhighlight %}


Compilation is now complete and ffmpeg (also ffprobe, ffserver, lame and x264)
should now be ready to use. The rest of this guide shows how to update or
remove FFmpeg.

**Tip**: Keep the ffmpeg_sources directory and all contents if you intend to
update as show below. Otherwise you can delete this directory.


## Updating
