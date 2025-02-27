# Install the AIY Projects software

This page describes how to install all software for an AIY Vision Bonnet or Voice Bonnet.

If you're updating an existing AIY kit or starting from scratch, we recommend you [install our
pre-built image](#install-our-pre-build-aiy-projects-image). But if you have your own Raspbian
system that you'd like to use with an AIY kit, then you can
[install our software on your existing Raspbian system](#install-aiy-software-on-an-existing-raspbian-system).

## Install our pre-build AIY Projects image

To flash our latest pre-built system image onto an SD card, follow these steps:

1. Download the latest `.img.xz` file from our [releases page on GitHub][github-releases].
   (For release details, see the [Change log][changelog].)
1. Plug-in your MicroSD card to your computer with an adapter.
1. Use a program such as [balenaEtcher](https://www.balena.io/etcher/) to flash the `.img.xy` file
   onto your MicroSD card. (balenaEtcher is free and works on Windows, Mac, and Linux.)

When flashing is done, put the MicroSD card back in your kit and you're good to go!


## Install AIY software on an existing Raspbian system

Follow these steps to install the AIY drivers and software onto an existing Raspbian system.

**Note:** This process is compatible with Raspbian Buster 64 bits from 2021-05-07 available there : https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2021-05-28/


### 1. Configure the Raspberry Pi

Enable SSH :

Put a file named "ssh" (without extension) on the boot partition

Now, start the Raspberry Pi

Connect to the Raspberry Pi SSH

Setup locales and WiFi country :

```bash
sudo raspi-config
```
Reboot :

```bash
sudo reboot
```

### 2. Add the AIY Debian packages repo

Add AIY package repo:

```bash
echo "deb https://packages.cloud.google.com/apt aiyprojects-stable main" | sudo tee /etc/apt/sources.list.d/aiyprojects.list
```

Add Google package keys:

```bash
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Update sources :

```bash
sudo apt-get update
```

(Raspbian 64 bits) Repair VLC packages :
```bash
sudo apt install libvlc-bin vlc-plugin-qt
```

Install the latest system updates (including kernel) :

```bash
sudo apt-get upgrade
```

Install Raspberry kernel headers :

```bash
sudo apt-get install -y raspberrypi-kernel-headers
```

Then reboot:

```bash
sudo reboot
```

### 3. Install optional packages

#### RGB Button Driver

This package is needed only if you're using the light-up RGB button that's included with
the Vision/Voice Bonnet:

```bash
sudo apt-get install -y leds-ktd202x-dkms
```

Run `sudo modprobe leds_ktd202x` to load the driver and `sudo modprobe -r leds_ktd202x` to
unload it. Vision/Voice Bonnet does this automatically via built-in device tree overlay
saved in the board's EEPROM.

#### Piezo Buzzer Driver

This package is needed only if you're using the piezo buzzer included with the Vision Bonnet:

```bash
sudo apt-get install -y pwm-soft-dkms
```

#### Pi Zero Ethernet-over-USB

This package is needed only if you're using Ethernet-over-USB on Pi Zero:

```bash
sudo apt-get install -y aiy-usb-gadget
```
Default Pi IP address is `192.168.11.2`, host IP address will be assigned automatically.

#### Support for AIY Projects app

In order to make the Pi work with the [AIY Projects][aiy-app] app:

```bash
sudo apt-get install -y aiy-bt-prov-server
```

### 4. Install required packages

Use the following commands to install packages for either the
[Vision Bonnet](#install-vision-bonnet-packages) or the
[Voice Bonnet/HAT](#install-voice-bonnethat-packages).

#### Install Vision Bonnet packages

Install the bonnet drivers:

```bash
sudo apt-get install -y aiy-vision-dkms
```

Install the [example vision models][aiy-models]:

```bash
sudo apt-get install -y aiy-models
```

Install the optimized `protobuf` library for better performance:

```bash
sudo apt-get install -y aiy-python-wheels
```

Enable camera module:

```bash
echo "start_x=1" | sudo tee -a /boot/config.txt
```

Set GPU memory to 128MB:

```bash
echo "gpu_mem=128" | sudo tee -a /boot/config.txt
```

Make sure to *not* use GPIO6 for SPI0 (required since 5.4 kernel):

```bash
echo "dtoverlay=spi0-1cs,cs0_pin=7" | sudo tee -a /boot/config.txt
```

Reboot :

```bash
sudo reboot
```

Then verify that `dmesg` output contains `Myriad ready` message :

```bash
dmesg | grep -i "Myriad ready"
```

You can also verify that camera is working fine by watching video on the
connected monitor:

```bash
raspivid -t 0
```

Or use `ffplay` to get video output on the host machine:

```bash
ssh pi@raspberrypi.local "raspivid --nopreview --timeout 0 -o -" | ffplay -loglevel panic -
```

#### Install Voice Bonnet/HAT packages

Voice HAT does not require any driver installation. You only need to load
device tree overlay on boot:

```bash
echo "dtoverlay=googlevoicehat-soundcard" | sudo tee -a /boot/config.txt
```

Voice Bonnet requires driver installation:

```bash
sudo apt-get install -y dkms aiy-voicebonnet-soundcard-dkms
```

Disable built-in audio:

```bash
sudo sed -i -e "s/^dtparam=audio=on/#\0/" /boot/config.txt
```

Install PulseAudio:

```bash
sudo apt-get install -y pulseaudio
sudo mkdir -p /etc/pulse/daemon.conf.d/
echo "default-sample-rate = 48000" | sudo tee /etc/pulse/daemon.conf.d/aiy.conf
```

You may also need to disable `module-suspend-on-idle` PulseAudio module for the
Voice HAT:

```bash
sudo sed -i -e "s/^load-module module-suspend-on-idle/#load-module module-suspend-on-idle/" /etc/pulse/default.pa
```

Disable the "To install the screen reader press control alt spce" audio message :

```bash
sudo rm /etc/xdg/autostart/piwiz.desktop
```

If you want to use Google Assistant, install the Raspberry-Pi-compatible
`google-assistant-library` python library from `aiy-python-wheels` package:

```bash
sudo apt-get install -y aiy-python-wheels
```

**(64 bits)** Then install protobuf:

```bash
pip3 download protobuf
pip3 install ./protobuf-*.whl
sudo nano /var/lib/dpkg/info/aiy-python-wheels.postinst
```

**(64 bits)** Comment out the lines containing

```
# pip3 install --no-deps --no-cache-dir --disable-pip-version-check \
       #     /opt/aiy/python-wheels/protobuf-3.6.1-cp35-cp35m-linux_armv6l.whl
```

**(64 bits)** Finish installation :   

```bash
sudo dpkg --configure --force-overwrite --force-overwrite-dir -a
```

Reboot:

```bash
sudo reboot
```

Check Voice bonnet is working :

```bash
aplay -l
```
Note : if Voice Bonnet is not detected after a reboot, try again after a complete shutdown. In my case, the Voice Bonnet works only after a complete shutdown, and is not detected after a reboot.

Then verify that you can record audio:

```bash
arecord -f cd test.wav
```

...and play a sound:

```bash
aplay test.wav
```

Additionally, the Voice Bonnet/HAT requires access to Google Cloud APIs.
To complete this setup, follow the [Voice Kit setup instructions][aiy-voice-setup].


### 5. Install the AIY Projects Python library

Finally, you need to install the [AIY Projects Python library](
https://aiyprojects.readthedocs.io/en/latest/index.html).

First make sure you have `git` installed:

```bash
sudo apt-get install -y git
```

Then clone this `aiyprojects-raspbian` repo from GitHub:

```bash
git clone https://github.com/google/aiyprojects-raspbian.git AIY-projects-python
```

And now install the Python library in editable mode:

```bash
pip3 install --upgrade google-assistant-grpc
```

Copy aiy library to examples folders :

```bash
cp -R ~/AIY-projects-python/src/aiy/ ~/AIY-projects-python/src/examples/voice/
cp -R ~/AIY-projects-python/src/aiy/ ~/AIY-projects-python/src/examples/vision/
```

**(32 bits)** Install google-assistant-library :

```bash
pip3 install google-assistant-library
```

**(64 bits)** Install google-auth :

```bash
pip3 install google-auth
```

**(64 bits)** Install google-auth-oauthlib :

```bash
pip3 install google-auth-oauthlib
```

Test audio :

```bash
python3 checkpoints/check_audio.py
```

## Appendix: List of all AIY Debian packages

The following is just a reference of all packages that are installed when you
follow the above steps.

### Vision and Voice Bonnets

* `aiy-dkms` contains MCU drivers:

  * `aiy-io-i2c` &mdash; firmware update support
  * `pwm-aiy-io` &mdash; [PWM][kernel-pwm] sysfs interface
  * `gpio-aiy-io` &mdash; [GPIO][kernel-gpio] sysfs interface
  * `aiy-adc`  &mdash; [Industrial I/O][kernel-iio] ADC interface

* `aiy-io-mcu-firmware` contains MCU firmware update service
* `leds-ktd202x-dkms` contains `leds-ktd202x` LED driver
* `pwm-soft-dkms` contains `pwm-soft` software PWM driver

* `aiy-python-wheels` contains optimized `protobuf` python
wheel (until [this issue][protobuf-issue] is fixed) along with [Google Assistant Library][assistant-library] for different Raspberry Pi boards.

### Vision Bonnet

* `aiy-vision-dkms` contains `aiy-vision` Myriad driver
* `aiy-vision-firmware` contains Myriad firmware
* `aiy-models` contains [models][aiy-models] for on-device inference:

  * Face Detection
  * Object Detection
  * Image Classification
  * Dish Detection
  * Dish Classification
  * iNaturalist Classification (plants, insects, birds)

### Voice Bonnet

* `aiy-voicebonnet-soundcard-dkms` contains sound drivers:

  * `rl6231`
  * `rt5645`
  * `snd_aiy_voicebonnet`


[changelog]: CHANGES.md
[raspbian]: https://www.raspberrypi.org/downloads/raspbian/
[image-flash]: https://www.raspberrypi.org/documentation/installation/installing-images/
[aiy-models]: https://aiyprojects.withgoogle.com/models/
[github-releases]: https://github.com/google/aiyprojects-raspbian/releases
[aiy-voice-setup]: https://aiyprojects.withgoogle.com/voice#google-assistant--get-credentials
[assistant-library]: https://pypi.org/project/google-assistant-library/
[protobuf-issue]: https://github.com/bennuttall/piwheels/issues/97
[kernel-pwm]: https://www.kernel.org/doc/Documentation/pwm.txt
[kernel-gpio]: https://www.kernel.org/doc/Documentation/gpio/sysfs.txt
[kernel-iio]: https://www.kernel.org/doc/Documentation/driver-api/iio/core.rst
[aiy-app]: https://play.google.com/store/apps/details?id=com.google.android.apps.aiy
