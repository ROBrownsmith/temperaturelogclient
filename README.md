# temperaturelogclient
Setting up pi zero w as temperaturelogclient 

*Purchase

pre-soldered gpio header version
https://thepihut.com/products/raspberry-pi-zero-w?ref=isp_rel_prd&isp_ref_pos=1&variant=547421782033

Blank micro sd card
https://thepihut.com/collections/raspberry-pi-sd-cards-and-adapters/products/blank-micro-sd-cards

case
https://www.modmypi.com/raspberry-pi/cases-183/raspberry-pi-zero--zero-w-cases-1123/plastic-cases-1145/unipicase-pi-zero-case-tall-justboom-dac-zero-faceplate

Power supply 
https://thepihut.com/collections/raspberry-pi-power-supplies/products/official-raspberry-pi-universal-power-supply

Pluggable sensor kit
https://www.amazon.co.uk/SODIAL-Temperature-Waterproof-Stainless-Terminal-black/dp/B07DXSTFZW/ref=sr_1_42?ie=UTF8&qid=1536842694&sr=8-42&keywords=ds18b20

*Assemble sensor and attach to GPIO pins

See JPEG in repository

*Install raspbian to sd card (or buy one pre-loaded)

In windows:

Use Etcher program to make raspbian image on sd card

Download raspbian stretch lite .zip file from raspberry pi.org

*Configure wireless & ssh 

In Linux, usng a card reader, mount the card and using a terminal

    sudo nano /etc/wpa_supplicant/wpa_supplicant.conf

Change the SSID and password values to match your network.

    network={

        ssid="SSID"
        psk="password"
        key_mgmt=WPA-PSK

    }

Create a file on the /boot partition of sd card named “ssh”.
    
    touch /boot/ssh

*Connect to pi 

Put sd card in pi zero and power up

Log in to router to find IP address of pi

From Linux terminal

    ssh pi@192.168.. # after @ symbol put whatever IP the router told you
    
Default password is raspberry

You are now connected to the Pi

*change pi root password 

    passwd 

*Change Linux user to templogclient and give sudo privelage 

    adduser --gecos "" templogclient 

    usermod -aG sudo templogclient 

    su templogclient 

*Copy public ssh certificate for no-password login 

On Local machine in a different terminal window - check for keys

    ls ~/.ssh

If keys present will list a file called id_rsa.pub
If not, run

    ssh-keygen
    
Either way, then    

    cat ~/.ssh/id_rsa.pub

Copy output and paste to remote terminal (the Pi terminal). Replace <paste key> with your paste.

    echo <paste key> >> ~/.ssh/authorized_keys

    chmod 600 ~/.ssh/authorized_keys

*Enable GPIO pins (still in Pi's terminal)

    sudo nano /boot/config.txt
    
Change to 

    Gpio overlay =1

*Install dependencies  

    sudo apt-get update

    sudo apt-get install -y supervisor git ufw 

*Set up firewall 

    sudo ufw allow ssh 

    sudo ufw allow http

    sudo ufw allow 443/tcp

    sudo ufw —force enable

    sudo ufw status

*Install client script 

    Git clone https://github.com/ROBrownsmith/temperaturelogclient

    cd temperaturelogclient

    Python3 -m venv venv 

    Source venv/bin/activate

    Pip install -r requirements.txt

*Edit python script so JSON is sent to your correct URL (look on line 69.)

    sudo nano *.py

*Set up supervisor 

    sudo cp temperaturelogclient.conf /etc/supervisor/conf.d/temperaturelogclient.conf

or

    cd /etc/supervisor/conf.d/

    sudo nano temperaturelogclient.conf
(adjust your home directory if not using templogclient as user)

    [program:temperaturelogclient]
    command=/home/templogclient/temperaturelogclient/venv/bin/python home/templogclient/temperaturelogclient/temperaturelogclient.py

    startsecs=60
    user=templogclient
    autostart=true
    autorestart=true
    stopasgroup=true
    killasgroup=true
    stdout_logfile=/home/templogclient/temperaturelogclient/temperaturelogclient_output.txt
    redirect_stderr=true

*restart Supervisor service

    sudo supervisorctl reload
