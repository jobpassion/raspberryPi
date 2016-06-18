### RaspberryPi 3B Bluetooth Speaker

#### Required
  * RaspberryPi Model 3B
  * headset(Speaker)

#### Install Dependency Packages
  
  Prepare your RaspberryPi, with RASPBIAN JESSIE LITE(or RASPBIAN JESSIE). 
  
  Assume you are login as root.
  
  Install the dependency packages:
  	
    apt-get update
    apt-get install bluez pulseaudio-module-bluetooth python-gobject python-gobject-2
#### Check And Pair Bluetooth
  check cmds:
  
    bluetoothctl:
      scan on
      trust xx:xx:xx:xx:xx:xx(your phone bluetooth mac)
      connect xx:xx:xx:xx:xx:xx(use pi connect phone first, next time you can connect phone with pi on your phone)
  if connected, completed checking of bluetooth
  
  If you're having problems connecting bluetooth try restarting pulseaudio:
  
  	pulseaudio -k
  	pulseaudio --start
  If that doesn't work you may need to load this module:
  
  	sudo pactl load-module module-bluetooth-discover
  Sometimes it works if you use the command without "sudo"
  
  If you have more problems this [stack exchange page](http://unix.stackexchange.com/questions/258074/error-when-trying-to-connect-to-bluetooth-speaker-org-bluez-error-failed) or this [arch linux page](https://wiki.archlinux.org/index.php/Bluetooth_headset) may be of help.
  
#### Config Bluetooth
  we'll config Bluetooth, to enable Bluetooth Audio
  cmds:
  
    vim /etc/bluetooth/main.conf
    
  Add the following line in the [General] section:
  
      Enable=Source,Sink,Media
  Set Bluetooth Class, Modify the following line to:
      
      Class = 0x00041C
      
  41C means your bluetooth contains audio functions
#### Config Raspberry Pi Audio
  

	  vim /etc/pulse/daemon.conf

  
  Add the following line after the commented example ";resample-method = speex-float-3":
  
  	resample-method = trivial
  
  start pulseaudio service with:
    
    pulseaudio -D
  lately, you can add this command to the rc.local script to enable Audio on start.
  load modules with:
    
    pactl load-module module-bluetooth-discover
  plugin your headset on raspberyPi
  test it with:
    
    apt-get install omxplayer
    omxplayer example.mp3
  now you should have heard the sound from the headset
  ctrl+C, when tests end
#### Connect Bluetooth with Audio
  now restart your bluetooth service:
    service bluetooth restart
    
    bluetoothctl:
      power on
      connect xx:xx:xx:xx:xx:xx
  now, you should have see your bluetooth sources with:
    
    pactl list sources short
  some texts like:
  
  	0    alsa_output.platform-bcm2835_AUD0.0.analog-stereo.monitor    module-alsa-card.c    s16le 2ch 44100Hz    SUSPENDED
  
  	1    bluez_source.XX_XX_XX_XX_XX_XX    module-bluetooth-device.c    s16le 2ch 44100Hz    SUSPENDED
  then show the sinks:
    
    pactl list sinks short
  	
  	0    alsa_output.platform-bcm2835_AUD0.0.analog-stereo    module-alsa-card.c    s16le 2ch 44100Hz    SUSPENDED
  remember the alsa_output.xxxxxxx, for later uses
  Connect the source to the sink:
    
    pactl load-module module-loopback source=bluez_source.XX_XX_XX_XX_XX_XX sink=alsa_output.xxxxxxx
  config raspberryPi use headset output Interface:
    
    raspi-config:
      Advanced Options
      	Audio
      	  Force 3.5mm ('headphone') jack
      	  
  now apply the volume settings:
  
    amixer -D pulse sset Master 0%
    pacmd set-sink-volume 0 65537
    
  play some music from phone, raspberryPi should have redirected the sound to your headset.Just like you have plugin headset directly in the phone


#### End
  The key section is the "Config Raspberry Pi Audio" and "Connect Bluetooth with Audio"
  When first test, you could start pulseaudio without -D, so you can see any error logs directly
