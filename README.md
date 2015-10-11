# Asterisk patch for AMR and AMR-WB

To add a codec for SIP/SDP (m=, rtmap, and ftmp), you create a format module in Asterisk: `codecs/codec_amr.patch` (for m= and rtmap) and `res/res_format_attr_amr.c` (for fmtp). However, this requires both call legs to support AMR (pass-through only). If one leg does not support AMR, the call has no audio. Or, if you use the pre-recorded voice and music files of Asterisk, these files cannot be heard, because they are not in AMR but in slin. Therefore, this repository adds not just a format module for the audio-codecs AMR and AMR-WB but a transcoding module as well.

This is an implementation of IETF [RFC 4867](http://tools.ietf.org/html/rfc4867). Sometimes, AMR is called AMR Narrowband (AMR-NB). AMR Wideband (ITU-T Recommendation G.722.2) is sometimes abbreviated W-AMR ([GSA](http://www.gsacom.com/hdvoice/)). GSMA Mobile [HD Voice](https://www.youtube.com/playlist?&list=PLj1MyDu3jckpSciPQ1Max0W6HDSaY8-n4) is AMR-WB. Research papers comparing AMR and AMR-WB with other audio codecs: [InterSpeech 2010](http://research.nokia.com/files/public/%5B12%5D_Interspeech%202010_Voice%20Quality%20Evaluation%20of%20Recent%20Open%20Source%20Codecs.pdf), [ICASSP 2010](http://research.nokia.com/files/public/%5B11%5D_ICASSP2010_Voice%20Quality%20Evaluation%20of%20Various%20Codecs.pdf), [InterSpeech 2011](http://research.nokia.com/files/public/%5B16%5D_InterSpeech2011_Voice_Quality_Characterization_of_IETF_Opus_Codec.pdf). Further [examples…](http://www.voiceage.com/Audio-Samples-Listening-Room.html)

## Installing the patch

The patch was built on top of Asterisk 13.6.0. If you use a newer version and the patch fails, please, [report](http://help.github.com/articles/creating-an-issue/)!

    cd /usr/src/
    wget downloads.asterisk.org/pub/telephony/asterisk/asterisk-13-current.tar.gz
    tar zxf ./asterisk*
    cd ./asterisk*
    apt-get --assume-yes install build-essential libssl-dev libncurses-dev libnewt-dev libxml2-dev libsqlite3-dev uuid-dev libjansson-dev libblocksruntime-dev

Install libraries:

If you do not want transcoding but pass-through only (because of license issues) please, skip this step. To support transcoding, you’ll need to install OpenCORE AMR, for example in Debian/Ubuntu:

    apt-get --assume-yes install libopencore-amrnb-dev libopencore-amrwb-dev libvo-amrwbenc-dev

Apply all patches:

    wget github.com/traud/asterisk-amr/archive/master.zip
    unzip -qq master.zip
    cp --verbose ./asterisk-amr*/* ./
    patch -p0 <./codec_amr.patch
    patch -p0 <./build_tools.patch
    patch -p0 <./capability_cached_format.patch
    patch -p0 <./register_interface_with_cached.patch
    patch -p0 <./rtp_fmtp_RFC_default.patch
    patch -p0 <./translate_joint_format.patch

Run the bootstrap script to re-generate configure:

    ./bootstrap.sh

Configure your patched Asterisk:

    ./configure

Enable slin16 in menuselect for transcoding, for example via:

    make menuselect.makeopts
    ./menuselect/menuselect --enable-category MENUSELECT_CORE_SOUNDS

Compile and install:

    sudo make install

## Testing
You can test AMR-WB out of the box using

A.  (Google Android) [CSipSimple](http://play.google.com/store/apps/details?id=com.csipsimple)

B.  (Google Android) [CounterPath Bria](http://play.google.com/store/apps/details?id=com.bria.voip)

C.  (Apple iOS) [CounterPath Bria](http://itunes.apple.com/app/bria-iphone-edition-voip-softphone/id373968636)

D.  (Windows Phone 8) [Linphone](http://www.windowsphone.com/s?appId=99661466-8c5c-489b-a567-569c1f480d29)

On ingress, this module supports the octet-aligned mode and the bandwidth-efficient mode. Currently on egress, only the bandwidth-efficient mode is advertised when transcoding. However, if the originating party supports AMR, its mode is passed transparently. Because Linphone supports only the octet-aligned mode, but does not honor the line fmtp in SDP, the mode is not negotiated correctly and ingress calls create distorted audio in Linphone. This bug is reported to the Linphone team. However, these VoIP/SIP clients offer G.722 and Opus, which should be used for wide-band audio instead. G.722 transcoding is build into Asterisk already. Opus can be [added as transcoding module …](http://github.com/seanbright/asterisk-opus/)

Actually, this repository was created for Nokia Mobile Phones which support AMR (since the year 2006) and AMR-WB (since the year 2009) in VoIP/SIP, like:

* [Nokia Asha 503](http://www.gsmarena.com/nokia_asha_503-5794.php) (Asha Software Platform),
* [Nokia 303](http://www.gsmarena.com/nokia_asha_303-4278.php) (Nokia Series 40),
* [Nokia E75](http://www.gsmarena.com/nokia_e75-2688.php) (Symbian/S60), and [Nokia Belle](http://www.gsmarena.com/results.php3?sOSes=5&sOSversions=5400).

## What is missing
This transcoding module is shoddy work. Therefore, please, double-check the source of `codec_amr.c` for any feature you need. For example IETF [RFC 4867](http://tools.ietf.org/html/rfc4867) offers CRCs, robust sorting and interleaving. If you need this, please, [add](http://help.github.com/articles/using-pull-requests/) that code or [report](http://help.github.com/articles/creating-an-issue/) the device/app which requires this. Several frames per payload are another issue: I simply do not have a testing device for this. The transcoding module works for me and contains everything I need. If you cannot code yourself, however, a feature is missing for you, please, [report](http://help.github.com/articles/creating-an-issue/) and send me at least a testing device.

## Thanks goes to
* the teams of the Android Open Source Project (AOSP), OpenCORE AMR, Debian Multimedia, and Ubuntu for providing the libraries.
* the Asterisk team: Thanks to their efforts and architecture this AMR module was written in one working day.
* [Sean Bright](http://github.com/seanbright/asterisk-opus/) provided the starting point with his Opus patch.