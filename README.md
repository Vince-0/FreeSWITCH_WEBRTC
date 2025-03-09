# VoIP WebRTC Server

Author [https://github.com/Vince-0](https://github.com/Vince-0/Projects)

This guide documents how to install and configure [FreeSWITCH](https://signalwire.com/freeswitch) on [Debian](https://www.debian.org/) 12 to create a WebRTC server as a proof of concept.


## What?

<p align="center">
<img src="https://github.com/Vince-0/FreeSWITCH_WEBRTC/blob/85b3d117a4d24f82a80262456e2a9f3229a3a54b/FreeSWITCH_WebRTC_bg.png" />
</p>


Connect voice, video, chat and canvas web applications, phone calls to servers and peer-to-peer.

This can be done using the [WebRTC](https://webrtc.org/) (real time communication) [HTML5 specification](https://www.w3.org/TR/webrtc/) and a FreeSWITCH server.

[FreeSWITCH](https://developer.signalwire.com/freeswitch/FreeSWITCH-Explained/) is an open source carrier-grade telephony platform implemented as a back-to-back user agent.

FreeSWITCH module [Verto](https://developer.signalwire.com/freeswitch/FreeSWITCH-Explained/Modules/mod_verto_3964934/) implements a "subset of a JSON-RPC connection designed for use over secure websockets".

FreeSWITCH connects [endpoints](https://developer.signalwire.com/freeswitch/FreeSWITCH-Explained/Introduction/Endpoints/) including

- WebRTC (Verto)
- VOIP applications (SIP, H323, IAX2, SCCP, GSM)
- Telephony [Zaptel](https://developer.signalwire.com/freeswitch/FreeSWITCH-Explained/Interoperability/OpenZap/Zapata-zaptel/Zapata-zaptel-interface_13173132/)
- Audio/video devices (ALSA, PortAudio)

FreeSWITCH has [modules](https://developer.signalwire.com/freeswitch/FreeSWITCH-Explained/Modules/) implementing

- Java
- Lua
- Perl
- Python
- Ruby
- Curl
- v8 (Javascript)
- Verious media codecs
- MongoDB
- Memcache
- ODBC
- XML
- Event Socket Layer
- [REST](https://developer.signalwire.com/freeswitch/FreeSWITCH-Explained/Modules/mod-verto/RESTful-Verto_8454242/)


## Why?

WebRTC enabled applications with use cases for

- Auction and bidding platforms

- Gaming and E-sports

- Language processing

- Internet-of-Things data transfer

- Tele-health and remote monitoring

- Tele-operations

- Video and voice streaming

- Virtual classroom


## How

### 1. Create a Signal Wire Personal Access Token

https://developer.signalwire.com/freeswitch/FreeSWITCH-Explained/Installation/how-to-create-a-personal-access-token/how-to-create-a-personal-access-token/

### 2. Install FreeSWITCH from Signal Wire repository

```bash
# Set your Signal Wire token
TOKEN=YOURSIGNALWIRETOKEN

# Install prerequisites

apt-get update && apt-get install -y gnupg2 wget lsb-release

# Add repository key

wget --http-user=signalwire --http-password=$TOKEN -O /usr/share/keyrings/signalwire-freeswitch-repo.gpg https://freeswitch.signalwire.com/repo/deb/debian-release/signalwire-freeswitch-repo.gpg

# Configure authentication and repository

echo "machine freeswitch.signalwire.com login signalwire password $TOKEN" > /etc/apt/auth.conf
chmod 600 /etc/apt/auth.conf

echo "deb [signed-by=/usr/share/keyrings/signalwire-freeswitch-repo.gpg] https://freeswitch.signalwire.com/repo/deb/debian-release/ `lsb_release -sc` main" > /etc/apt/sources.list.d/freeswitch.list

echo "deb-src [signed-by=/usr/share/keyrings/signalwire-freeswitch-repo.gpg] https://freeswitch.signalwire.com/repo/deb/debian-release/ `lsb_release -sc` main" >> /etc/apt/sources.list.d/freeswitch.list


# Install FreeSWITCH

apt-get update && apt-get install -y freeswitch-meta-all

```

### 3. Configure FreeSWITCH Global Variables

Edit the FreeSWITCH configuration file to set:

```xml
<!-- Domain Name -->

<X-PRE-PROCESS cmd="set" data="domain_name=$${domain}"/>

<!-- Default Password -->

<X-PRE-PROCESS cmd="set" data="default_password=1234"/>
```

### 4. Start and Verify FreeSWITCH

```bash

# Start the service

systemctl start freeswitch

# Check service status

systemctl status freeswitch

# Monitor logs

tail -f -n 1000 /var/log/freeswitch/freeswitch.log

# Verify console access

fs_cli

```

### 5. Configure TLS Certificates with Certbot

```bash

# Install certbot

apt -y install certbot

# Clear firewall rules temporarily to allow verification

iptables --flush

# Get certificate (replace <HOSTNAME> with your actual hostname)

certbot certonly --standalone -d <HOSTNAME>

# Certificate will be saved at: /etc/letsencrypt/live/<HOSTNAME>/fullchain.pem

# Key will be saved at: /etc/letsencrypt/live/<HOSTNAME>/privkey.pem

```

### 6. Prepare Certificate for FreeSWITCH

```bash

# Concatenate certificate and private key

cat fullchain.pem > wss.pem

cat privkey.pem >> wss.pem

# Backup existing certificate

mv /etc/freeswitch/wss.pem /etc/freeswitch/wss.pem.BAK

# Move new certificate to FreeSWITCH location

mv /etc/letsencrypt/live/<HOSTNAME>/wss.pem /etc/freeswitch/tls/

# Set proper permissions

chown freeswitch:freeswitch /etc/freeswitch/tls/wss.pem

# Restart FreeSWITCH to apply changes

systemctl restart freeswitch

```

## References

- [FreeSWITCH Configuration Guide](https://developer.signalwire.com/freeswitch/FreeSWITCH-Explained/Configuration/Configuring-FreeSWITCH/)

- [FreeSWITCH mod_verto Module](https://developer.signalwire.com/freeswitch/FreeSWITCH-Explained/Modules/mod_verto_3964934/)

- [FreeSWITCH WebRTC with SIPML5](https://nickvsnetworking.com/freeswitch-webrtc-with-sipml5/)

- [SIPML5 Documentation](https://www.doubango.org/sipml5/index.html)
