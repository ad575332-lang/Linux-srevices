# Raportti — Linux Exercises, Module 5 (TLS Certificates)

## 1. DNS

dig-työkalun asennus:
```bash
sudo apt-get install dnsutils
```

DNS-tarkistus omalle domainille:
```bash
dig tls-test006.linuxkurssi.xyz
```
```
;; ANSWER SECTION:
tls-test006.linuxkurssi.xyz. 1799 IN A 20.238.120.81
```
DNS-tietue osoittaa oikein palvelimen julkiseen IP-osoitteeseen.

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/9b5c680c-8aa8-46ee-8e9e-2517b7fdb933" />
 — dnsutils-asennus, ensimmäinen dig-kysely

### DNS-liikenteen tarkkailu tcpdumpilla

Ensimmäinen yritys epäonnistui, koska komento suoritettiin vahingossa paikallisella VirtualBox-koneella (verkkoliitäntä siellä on `enp0s3`, ei `eth0`). Oikean palvelimen (`tls-test006`) yhdistämisen ja tcpdumpin asennuksen jälkeen:

```bash
sudo tcpdump -i eth0 port 53 -n -v
```

**Ensimmäinen yritys** — 0 pakettia siepattu:
```
0 packets captured
0 packets received by filter
0 packets dropped by kernel
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/cb92ea15-e025-4d94-b273-a3261116f7f0" />
 — dig-kysely ja tyhjä tcpdump-tulos (0 pakettia)

**Miksi liikennettä ei näkynyt:** vastaus oli jo tallennettu järjestelmän paikalliseen DNS-välimuistiin (`127.0.0.53`, systemd-resolved), joten varsinaista verkkopyyntöä porttiin 53 ei tapahtunut.

**Uusi kysely** — nyt tcpdump näytti todellista liikennettä:
```
20:07:04 IP 10.0.0.16.45358 > 168.63.129.16.53: [1au] SRV? _http._tcp...
20:07:43 IP 10.0.0.16.35568 > 168.63.129.16.53: [1au] A? tls-test006.linuxkurssi.xyz.
168.63.129.16.53 > 10.0.0.16.35568: A 20.238.120.81
```

Samanaikainen dig-kysely näytti käytetyn resolverin:
```
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/fbe08130-9760-4c55-80d8-65e460c6eb58" />
 — tcpdump todellisella DNS-liikenteellä (kyselyt Azuren resolveriin 168.63.129.16) ja dig, joka näyttää järjestelmän resolverin 127.0.0.53

**Havainnot:**
- Kun sama domain kysytään uudelleen lyhyen ajan sisällä, vastaus tulee yleensä paikallisesta välimuistista (systemd-resolved), eikä varsinaista verkkopakettia lähetetä porttiin 53.
- Jos DNS-paketteja ei näy siepatussa liikenteessä, se tarkoittaa, että vastaus tuli välimuistista eikä verkon kautta.
- Pakotettu kysely ulkoiselle DNS-palvelimelle (`dig ... @8.8.8.8`) ohittaa paikallisen välimuistin ja järjestelmän resolverin, joten se tuottaa aina näkyvää verkkoliikennettä — tämä osoittaa, että käyttöjärjestelmä käyttää oletuksena omaa välimuistoivaa resolveriaan (`127.0.0.53`) sen sijaan, että se ottaisi suoraan yhteyttä ulkoisiin DNS-palvelimiin joka kyselyllä.

## 2. Name-Based VirtualHost

Sivuston sisällön luonti:
```bash
mkdir public-sites
cd public-sites
echo "this is my public web server" > index.html
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/37c2e38a-8305-4df6-bc28-9fb5869a30ba" />
 — index.html-tiedoston luonti ja sisällön kirjoitus

Virtual host -määrityksen luonti:
```bash
sudo nano /etc/apache2/sites-available/tls-test006.linuxkurssi.xyz.conf
sudo a2ensite tls-test006.linuxkurssi.xyz.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

### 403 Forbidden -virheen korjaus

Ensimmäisellä testauskerralla sivusto palautti 403 Forbidden -virheen — Apachella (`www-data`) ei ollut riittäviä oikeuksia väliin jääviin hakemistoihin:
```bash
sudo chmod o+x index.html
sudo chmod o+x /home/linuxuser
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/0030105a-c83f-4e49-b83e-2901c06d9c67" />
 — 403 Forbidden -virhe ja oikeuksien korjaus chmodilla

Korjauksen jälkeen sivusto toimi:
```bash
curl http://tls-test006.linuxkurssi.xyz
# this is my public web server
```

### Lisätiedostojen luonti kahdelta käyttäjältä

linuxuser-käyttäjänä:
```bash
echo "file created by linuxuser" > ~/public-sites/linuxuserfile.txt
```

edituser-käyttäjänä (ensimmäinen yritys epäonnistui — Permission denied):
```bash
su - edituser
echo "file created by edituser" > /home/linuxuser/public-sites/editorfile.txt
# Permission denied
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/ef216510-c972-4d21-b83f-22611987a618" />
 — onnistunut curl-testi, käyttäjien välillä vaihtaminen, edituser-käyttäjän ensimmäinen epäonnistunut yritys

Korjaus — webteam-ryhmä oli jo olemassa edellisestä moduulista, mutta hakemiston omistajuus oli nollautunut:
```bash
sudo groupadd webteam
# group 'webteam' already exists
sudo chown linuxuser:webteam ~/public-sites
```

Tämän jälkeen edituser onnistui luomaan tiedoston:
```bash
echo "file created by edituser" > /home/linuxuser/public-sites/editorfile.txt
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/b76d9d3b-c5ad-4eac-9cf7-9ef76d5f095a" />
 — webteam-ryhmän omistajuuden palautus, edituser-käyttäjän onnistunut tiedoston luonti

### Muiden kuin index.html-tiedostojen testaus

```bash
curl http://tls-test006.linuxkurssi.xyz/editorfile.txt
# file created by edituser
curl http://tls-test006.linuxkurssi.xyz/linuxuserfile.txt
# file created by linuxuser
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/3da04fb2-446c-4fa9-99a9-ea7ac7f8cec1" />
 — onnistunut pääsy molempiin tiedostoihin curlilla

**Havainto:** Apache näyttää index.html-tiedoston automaattisesti vain, kun pyyntö kohdistuu sivuston juureen (`/`). Kun URL-osoitteessa mainitaan tiedoston nimi suoraan (`/editorfile.txt`), Apache etsii kyseisen nimisen tiedoston `DocumentRoot`-hakemistosta ja tarjoilee sen sisällön suoraan — näin mikä tahansa tiedosto `public-sites/`-hakemistossa on saatavilla suoralla linkillä, ei vain index.html.

## 3. TLS Certificate

certbotin asennus:
```bash
sudo apt-get install certbot python3-certbot-apache
```

Sertifikaatin hankinta:
```bash
sudo certbot --apache -d tls-test006.linuxkurssi.xyz,www.tls-test006.linuxkurssi.xyz
```
```
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/tls-test006.linuxkurssi.xyz/fullchain.pem
Key is saved at: /etc/letsencrypt/live/tls-test006.linuxkurssi.xyz/privkey.pem
This certificate expires on 2026-12-20.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Congratulations! You have successfully enabled HTTPS on https://tls-test006.linuxkurssi.xyz
```

Tarkistus certbot certificates -komennolla:
```bash
sudo certbot certificates
```
```
Certificate Name: tls-test006.linuxkurssi.xyz
Domains: tls-test006.linuxkurssi.xyz www.tls-test006.linuxkurssi.xyz
Expiry Date: 2026-12-20 17:11:50+00:00 (VALID: 89 days)
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/1de1b69c-286a-46fd-bdcc-5c92cfb9e121" />
— sertifikaatin onnistunut hankinta ja certbot certificates -tuloste

Tarkistus selaimella: `https://tls-test006.linuxkurssi.xyz` avautuu lukko-kuvakkeella osoiterivillä, yhteys on suojattu.

crt.sh-tarkistus (`https://crt.sh/?q=tls-test006.linuxkurssi.xyz`) ei tarkistushetkellä näyttänyt tuloksia — todennäköisesti Certificate Transparency -lokien indeksoinnin viiveen tai itse crt.sh-palvelun väliaikaisen epävakauden vuoksi (osa yrityksistä palautti 502 Bad Gateway -virheen). Suora tarkistus `certbot certificates`-komennolla vahvistaa, että sertifikaatti on todellisuudessa myönnetty ja toimii.

Uusimisen automaation tarkistus:
```bash
sudo certbot renew --dry-run
```
Sertifikaatin automaattinen uusiminen on jo asetettu certbotin toimesta systemd-ajastimen kautta sertifikaatin alkuperäisen hankinnan yhteydessä; `--dry-run` vain varmistaa, että uusimisprosessi toimii oikein tulevaisuudessa, ilman todellista sertifikaatin uudelleenmyöntämistä.

## 4. Monitoring — curl verbose-tilassa

### HTTPS-liikenne

```bash
curl -v https://tls-test006.linuxkurssi.xyz
```
Vastauksen sisältö näkyy täysin luettavana:
```
< Content-Type: text/html
this is my public web server
```

**Selitys:** curl on itse TLS-yhteyden laillinen osapuoli ja purkaa salauksen automaattisesti vastaanotettaessa — siksi sisältö näkyy tulosteessa. Tämä ei ole ristiriidassa salauksen kanssa: ulkopuolinen verkon tarkkailija (esim. tcpdump/ngrep portissa 443, ilman osallistumista TLS-handshakeen) näkisi vain salattua, lukukelvotonta dataa, kuten osoitettiin SSH-liikenteellä Module 4:ssä.

### HTTP-liikenne (uudelleenohjaus päällä)

```bash
curl -v http://tls-test006.linuxkurssi.xyz
```
```
< HTTP/1.1 301 Moved Permanently
< Location: https://tls-test006.linuxkurssi.xyz/
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/de5e30af-174f-4b66-b4ac-30d655470d53" />
 — curl -v http näyttää 301-uudelleenohjauksen HTTPS:ään

**Selitys:** certbotin asentama sertifikaatti lisäsi automaattisesti uudelleenohjaussäännön (RewriteRule) portin 80 konfiguraatioon — jokainen HTTP-pyyntö ohjataan heti HTTPS-versioon sivustosta, joten sisältöä ei tarjoilla suoraan HTTP:n kautta.

### Uudelleenohjauksen väliaikainen poistaminen käytöstä

Konfiguraatiotiedosto `/etc/apache2/sites-available/tls-test006.linuxkurssi.xyz.conf` — uudelleenohjausrivit kommentoitu pois:
```apache
#RewriteEngine on
#RewriteCond %{SERVER_NAME} =tls-test006.linuxkurssi.xyz [OR]
#RewriteCond %{SERVER_NAME} =www.tls-test006.linuxkurssi.xyz
#RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/105b55c4-9782-4e54-9f18-4640bab076b8" />
 — uudelleenohjaus kommentoitu nanossa

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
curl -v http://tls-test006.linuxkurssi.xyz
```
```
< HTTP/1.1 200 OK
this is my public web server
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/9ab453e9-7eca-4cdd-b144-cd22dd7adee1" />
 — uudelleenohjauksen poistamisen jälkeen: suora 200 OK -vastaus HTTP:n kautta, sisältö näkyy selväkielisenä

**Selitys:** ilman uudelleenohjausta pyyntö menee suoraan tavallisen HTTP:n kautta, ilman ohjausta HTTPS:ään — sisältö välittyy yhtä lailla luettavana, mutta nyt kokonaan ilman salausta: kuka tahansa verkon ulkopuolinen tarkkailija pystyisi lukemaan sen suoraan verkkoliikenteestä, mikä havainnollistaa, miksi pysyvä HTTPS-uudelleenohjaus on tärkeä oikeassa tuotantokäytössä.

Testin jälkeen uudelleenohjaus otettiin uudelleen käyttöön (rivit poistettu kommenteista), konfiguraatio tarkistettiin ja Apache ladattiin uudelleen.

## 5. TLS — Yhteenveto

TLS luo suojatun yhteyden selaimen ja palvelimen välille handshake-prosessin kautta: palvelin esittää selaimelle sertifikaatin, jonka on allekirjoittanut luotettu varmentaja (esimerkiksi Let's Encrypt), ja todistaa yksityisen avaimensa avulla, joka vastaa sertifikaatin julkista avainta, että se todella omistaa kyseisen sertifikaatin. Onnistuneen tunnistautumisen jälkeen osapuolet sopivat kertakäyttöisestä salausavaimesta kyseiselle istunnolle, jolla kaikki siirrettävä data salataan.

Tietojen suojaaminen internetin yli siirrettäessä on tärkeää, koska ilman salausta (kuten tavallisessa HTTP:ssä) kuka tahansa, jolla on pääsy asiakkaan ja palvelimen väliseen verkkoon, voi lukea tai muokata siirrettävää tietoa — mukaan lukien salasanat ja henkilökohtaiset tiedot. TLS tarjoaa kolme ominaisuutta samanaikaisesti: luottamuksellisuuden (salaus piilottaa sisällön ulkopuolisilta), todentamisen (varmistaa, että yhteys on muodostettu juuri oikean palvelimen kanssa) ja eheyden (varmistaa, ettei dataa ole muutettu matkan varrella).
