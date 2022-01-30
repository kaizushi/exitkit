EXITKIT
=======
This is a guide you can read and some configuration files you can upload to have what I call a *private tor exit* for systems which are isolated behind a *tor transparent proxy/middlebox*. Such systems typically use Tor exits to access the *clearnet* (the regular Internet). One can run another transparent proxy locally that takes this traffic and sends it to a proxy server via a Hidden Service. This prevents bad exits from exploiting software updates and such. It also means that your requests are not from a known Tor exit which are often blocked by many service providers. This guide is based on what I have been doing in production. It was first written in January 2022.

Licence
-------
This guide and the configuration files within are under the MIT Licence.

See the file LICENCE for more details.

About Kaizushi
--------------
I am a young lady who has been since my late adolesence hosting Tor hidden services on the dark web. I call that operation **Kaizushi's Little Onion Server** AKA **KLOS**

You can contact me on my email address **kaizushi@infantile.us** if you have any questions.

The **KLOS** website is available in these places...
- https://kloshost.online/
- http://kaizushih5iec2mxohpvbt5uaapqdnbluaasa2cmsrrjtwrbx46cnaid.onion/
- http://bzznfzwjjeiwzrsy6xxlsahswldtq2jcfydq7qhopjctt327qlna.b32.i2p/

About Guide
-----------

This is a first draft and I have not actually tested everything here. I am pretty certain it is correct.

This is both a guide in a text file (README.txt) and some configuration files which can be modified and uploaded as per the instructions within.

To get started clone the Git and start reading README.txt!

	git clone https://github.com/kaizushi/exitkit

Donate
------

If this guide was useful to you donate some money my way...
- Bitcoin (BTC): 1FN1Tb6f8WE8uk6Wj5kBy6ryzohhrqfvui
- Monero (XMR): 83eVdewjYYzQMqcwFVtbKKNjJnowHVXRJA7aLXB6Tov3iAuRATHRWSNaxUWs9Ez2uN6at3vvdJ6JmF2kDNrCTFBdEGRG1Z9
