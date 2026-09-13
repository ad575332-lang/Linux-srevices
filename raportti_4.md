# Raportti — Linux Exercises, Module 4 (Basic Configuration, Azure VM)

## 1. Kirjautuminen ja järjestelmän päivitys

Kirjauduin etäpalvelimelle annetuilla tunnuksilla:
```bash
ssh linuxuser@<ip-osoite>
```

Vaihdoin salasanan ensimmäisen kirjautumisen jälkeen:
```bash
passwd
```
```
Changing password for linuxuser.
Current password:
New password:
Retype new password:
passwd: password updated successfully
```
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/d94ec541-af85-4d93-9f67-fdc6fb6659a9" />
 — salasanan vaihto onnistuneesti

Päivitin järjestelmän:
```bash
sudo apt-get update
sudo apt-get upgrade
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/5c289b5f-b47a-4cf9-87d5-e418bb608515" />
 — pakettilistan päivitys ja asennettujen pakettien päivitys

## 2. SSH-avainautentikointi

Tarkistin SSH-palvelimen tilan etäpalvelimella:
```bash
sudo systemctl status ssh
```
```
● ssh.service - OpenBSD Secure Shell server
     Active: active (running) since Tue 2026-09-08 10:36:44 UTC; 4 days ago
```
SSH-palvelin oli jo valmiiksi asennettuna ja käynnissä — pilvipalveluntarjoaja (Azure) asentaa sen automaattisesti, koska se on ainoa tapa saada etäyhteys palvelimeen.

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/aeed0892-f027-48fd-b0aa-1f6eb3a394f5" />
 — SSH-palvelimen tila

Loin avainparin **paikallisella koneella**:
```bash
ssh-keygen
```
```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/alina/.ssh/id_ed25519):
Enter passphrase for "/home/alina/.ssh/id_ed25519" (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/alina/.ssh/id_ed25519
Your public key has been saved in /home/alina/.ssh/id_ed25519.pub
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/5234efc0-7911-431f-965d-a7fa346dc536" />
 — avainparin generointi ja `~/.ssh/`-hakemiston sisältö

Kopioin julkisen avaimen etäpalvelimelle:
```bash
ssh-copy-id linuxuser@<ip-osoite>
```

Testasin salasanattoman kirjautumisen:
```bash
ssh linuxuser@<ip-osoite>
```
Kirjautuminen onnistui ilman salasanakysymystä — julkinen avain oli lisätty onnistuneesti tiedostoon `~/.ssh/authorized_keys` etäpalvelimella.

## 3. Apache-webpalvelimen asennus

```bash
sudo apt-get install apache2
```

Muutin oletussivun sisällön:
```bash
echo "this is my test page" | sudo tee /var/www/html/index.html
```

Testasin selaimella ja curlilla:
```bash
curl http://<ip-osoite>
```
```
this is my test page
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/bb41d282-c9b5-46ce-9146-87342c0b94bc" />
 — Apachen asennus, oletussivun testaus curlilla, `systemctl status apache2`

Loin hakemiston tulevaa sivustoa varten:
```bash
mkdir -p /home/linuxuser/public-sites
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/d7c635e8-d343-4fa9-b660-f0eadd15d0b3" />
 — hakemiston luonti ja tarkistus (`ls -ld`)

## 4. ufw-palomuurin asennus ja määrittäminen

```bash
sudo apt-get install ufw
sudo ufw allow ssh
sudo ufw enable
sudo ufw allow http
sudo ufw allow https
sudo ufw status verbose
```
```
22/tcp    ALLOW IN    Anywhere
80/tcp    ALLOW IN    Anywhere
443       ALLOW IN    Anywhere
```
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/c993c480-e3d2-4a18-92d8-ee36103e46bb" />
 — ufw:n asennus, sääntöjen lisäys ja tila

## 5. Verkko (Networking)

```bash
ip addr
```
Näytti vain **yksityisen** IP-osoitteen (`10.0.0.16`) — julkista IP-osoitetta ei näy käyttöjärjestelmän sisältä, koska Azure soveltaa sitä NAT:in kautta verkkokortin/isännän tasolla.

```bash
ip route
```
Näytti oletusreitin (`default via 10.0.0.1`) ja paikallisen aliverkon (`10.0.0.0/20`) sekä Azuren sisäiset palvelu-osoitteet (`168.63.129.16`, `169.254.169.254` — instanssin metadata).

Julkisen IP-osoitteen selvittäminen palvelimen sisältä:
```bash
curl -L ifconfig.me
```

## 6. Pakettien tarkastelu (Packet inspection)

Asensin ngrepin:
```bash
sudo apt-get install ngrep
```

### HTTP-liikenteen tarkastelu

Terminaali 1:
```bash
sudo ngrep -d eth0 -W byline "" host <ip-osoite> and port 80
```
Terminaali 2:
```bash
curl http://<ip-osoite>
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/d00cae3f-5c0a-4c3b-9c4c-e2aec1c76759" />
 — ngrep-komento ja sen tulos HTTP-liikenteestä

**Havainnot:**
- Näkyvissä: itse HTTP-pyyntö (`GET / HTTP/1.1`, `Host:`, `User-Agent:`) ja vastaus (`HTTP/1.1 200 OK`)
- Sivun sisältö **näkyy selväkielisenä** ("this is my test page") — HTTP-protokolla ei salaa liikennettä, joten kuka tahansa verkkoa kuunteleva näkee koko sisällön.
- Parametrit: `-d eth0` valitsee verkkoliitännän, `-W byline` tulostaa rivi kerrallaan, `host`/`and`/`port` suodattavat näytettävät paketit.

### SSH-liikenteen tarkastelu

```bash
sudo ngrep -d eth0 -W byline "" port 22
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/67944ade-e552-4efb-986d-f28e4b99ed4f" />
 — ngrep-tulos SSH-liikenteestä

**Havainnot:** paketit näkyvät (lähettäjä, vastaanottaja, portti), mutta sisältö on täysin **lukukelvotonta** satunnaista tavudataa. Tämä osoittaa, että SSH-protokolla **salaa koko liikenteen** — toisin kuin HTTP, jonka sisältö oli luettavissa suoraan.

### ICMP (ping) -liikenteen tarkastelu

```bash
sudo ngrep -d eth0 -W byline "" icmp
```
```bash
ping 8.8.8.8
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/bb8d126e-d001-456c-8fbe-167dde8358d4" />
 — ngrep-tulos ICMP-liikenteestä

**Havainnot:** näkyvissä ICMP Echo Request (`8:0`) / Echo Reply (`0:0`) -parit jokaista pingiä kohden. ICMP on verkon diagnostiikkaan tarkoitettu protokolla — sillä ei siirretä varsinaista sovellusdataa, vaan sitä käytetään yhteyden toimivuuden tarkistamiseen ja virheilmoituksiin. Tämä tekee siitä keskeisen työkalun verkko-ongelmien vianetsinnässä.

### Vertailu tcpdumpiin

```bash
sudo tcpdump -i eth0 icmp
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/0870889a-5cfb-4e27-91f8-90d16093933a" />
 — tcpdump-tulos ja vertailu ngrep-tulokseen

**Vertailu:** `tcpdump` esittää tiedon tiiviimmin ja luettavammin ("ICMP echo request"/"echo reply" sanoina), kun taas `ngrep` näyttää paketin raa'an sisällön tavuina — hyödyllistä nimenomaan silloin, kun halutaan nähdä paketin todellinen tekstisisältö (kuten HTTP-testissä).

## 7. Challenge — edituser ja jaettu ryhmä

Loin uuden käyttäjän ilman sudo-oikeuksia:
```bash
sudo adduser edituser
sudo groupadd webteam
sudo usermod -aG webteam linuxuser
sudo usermod -aG webteam edituser
```

Määritin oikeudet public-sites-hakemistolle:
```bash
sudo chown linuxuser:webteam /home/linuxuser/public-sites
sudo chmod g+s /home/linuxuser/public-sites
sudo chmod -R u=rwx,g=rwx,o=rx /home/linuxuser/public-sites
```

**Ongelma ja korjaus:** hakemistorakenne muodostui vahingossa syvemmäksi kuin oletettiin (`/home/linuxuser/home/linuxuser/public-sites`). Lisäksi väliin jäävät hakemistot eivät aluksi antaneet `edituser`-käyttäjälle kulkuoikeutta (`x`), mikä esti pääsyn `public-sites`-hakemistoon, vaikka itse hakemiston oikeudet olivat kunnossa. Korjasin tämän:
```bash
sudo chmod g+rx /home/linuxuser /home/linuxuser/home /home/linuxuser/home/linuxuser
sudo chown :webteam /home/linuxuser
sudo chown :webteam /home/linuxuser/home
sudo chown :webteam /home/linuxuser/home/linuxuser
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/7cbdb7b3-6267-43be-9d20-7dc468999a5d" />
 — oikeuksien korjaus välissä oleviin hakemistoihin

### Testaus

```bash
echo "file from linuxuser" > .../public-sites/file1.txt
echo "second file after setgid" > .../public-sites/file2.txt
```
`file1.txt` (luotu **ennen** setgid-bitin asettamista) sai ryhmäksi `linuxuser`, kun taas `file2.txt` (luotu **setgid-bitin jälkeen**) sai automaattisesti ryhmäksi `webteam` — tämä osoittaa, että setgid vaikuttaa vain sen käyttöönoton jälkeen luotuihin tiedostoihin.

Vaihdoin käyttäjäksi edituser:
```bash
su - edituser
echo "test after chown fix" > .../public-sites/file_edit.txt
```
Onnistui — `edituser` pystyi luomaan tiedoston yhteiseen hakemistoon oikeuksien korjauksen jälkeen.

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/66baa916-872b-4ae9-bed9-b6ed1bc87a42" />
 — lopullinen tiedostolistaus, jossa näkyy kaikkien testien tulokset

### SFTP-testi

Paikallisella koneella:
```bash
echo "test file from local machine" > test_local.txt
sftp linuxuser@<ip-osoite>
```
```
sftp> put test_local.txt /home/linuxuser/home/linuxuser/public-sites/test_local.txt
Uploading test_local.txt to .../test_local.txt
test_local.txt   100%   29   0.2KB/s   00:00
sftp> exit
```

Tarkistus etäpalvelimella:
```bash
ls -la /home/linuxuser/home/linuxuser/public-sites/
```
```
-rwxrwxr-x 1 linuxuser linuxuser   19 file1.txt
-rwxrwxr-x 1 linuxuser webteam    25 file2.txt
-rw-rw-r-- 1 edituser  webteam    21 file_edit.txt
-rw-r--r-- 1 linuxuser webteam    29 test_local.txt
```

**Havainnot:** SFTP toimii saman SSH-yhteyden ja samojen käyttöoikeuksien päällä kuin tavallinen terminaalityöskentely — myös SFTP:llä ladattu tiedosto sai oikean ryhmän (`webteam`) setgid-bitin ansiosta. Sekä `linuxuser` että `edituser` pystyivät lisäämään ja muokkaamaan tiedostoja samassa hakemistossa ilman oikeuksiin liittyviä ristiriitoja.
