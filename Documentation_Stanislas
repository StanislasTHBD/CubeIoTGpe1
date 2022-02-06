# Raspberry
# Installer os « Raspberry Pi OS Lite » sur carte SD 
    • ( Lien : https://downloads.raspberrypi.org/raspios_lite_armhf/images/raspios_lite_armhf-2022-01-28/2022-01-28-raspios-bullseye-armhf-lite.zip ) 
Configuration du Raspberry Pi 
    • Ajouter un fichier wpa_supplicant.conf =>
-----------------------------------------------------------------------------------------
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=FR
network={
	ssid="reseau_nom"
	psk="reseau_mdp"
	scan_ssid=1
	key_mgmt=WPA-PSK
}
-----------------------------------------------------------------------------------------
    • Ajouter un fichier ssh (sans extension) vide

Branchement & connexion avec le Raspberry Pi
    • Brancher le raspberry et le pinger avec la commande pour trouver l’adresse IP : ping raspberrypi.local
    • Connexion ssh => pi@adresse_ip

	ex :	Taper la commande :ssh pi@192.168.43.60
		Mdp : raspberry

    • Préparation raspberry :
    • sudo apt-get update && apt-get upgrade
    • sudo apt-get install php php-mysql apache2 mariadb-server
    • mysql_secure_installation
    • sudo apt-get install phpmyadmin
      

    • Page de connexion à phpmyadmin sur :192.168.43.60/phpmyadmin
user : root
mdp : root 
ESP8266
    • Pour effacer le flash du ESP8266 :
python3 -m esptool erase_flash
    • Pour mettre le flash sur l’ ESP8266 :
python3 -m esptool write_flash –flash_size=1MB 0 ./Téléchargements/esp8266-lm-20220117-v1.18.bin

Schéma du câblage
