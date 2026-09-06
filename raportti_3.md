# Raportti — Linux Exercises, Module 3 (Apache2, Name-based Virtual Host)

## 1. Apache2:n asennus

```bash
sudo apt-get install apache2
```

Tilan tarkistus:
```bash
sudo systemctl status apache2
```
Tulos: palvelu on aktiivinen ja käynnissä (`active (running)`), käynnistynyt järjestelmän käynnistyksen yhteydessä, ja se on synnyttänyt 3 työprosessia (`apache2 -k start`). Automaattinen käynnistys on päällä (`enabled`).

Automaattisen käynnistyksen tarkistus:
```bash
sudo systemctl is-enabled apache2
# enabled
```
Apache oli jo valmiiksi asetettu käynnistymään automaattisesti järjestelmän käynnistyessä — lisätoimenpiteitä ei tarvittu.
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/0cb0c234-a859-49cf-a403-24184d97e9d2" />
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/78098ea4-451f-4035-bb15-8e28293156de" />

## 2. Oletussivun tarkistus

**Selaimella:** `http://localhost` — avautui Apache2:n oletussivu (Apache2 Debian Default Page, "It works!").

**curl:lla:**
```bash
curl localhost
```
Palautti saman oletussivun HTML-koodin tekstinä, ilman selainta.
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/730553f3-54b8-4673-973f-615f0d363c2a" />

## 3. Oletussivun sisällön muuttaminen

```bash
echo 'this is the default page of my new web server' | sudo tee /var/www/html/index.html
```

Tarkistus:
```bash
curl localhost
# this is the default page of my new web server
```

### Komennon selitys vaihe vaiheelta

1. `echo 'teksti'` — tulostaa annetun merkkijonon vakiotulosteeseen (stdout).
2. `|` (putki, pipe) — ohjaa tämän tulosteen ei näytölle, vaan seuraavan komennon syötteeksi.
3. `sudo tee /var/www/html/index.html` — `tee` vastaanottaa tekstin putken kautta ja tekee kaksi asiaa samanaikaisesti: (a) kirjoittaa sen annettuun tiedostoon ja (b) näyttää saman tekstin myös näytöllä. `sudo` antaa `tee`-komennolle root-oikeudet, joita tarvitaan kirjoittamiseen järjestelmän kansioon `/var/www/`.

### Muita tapoja saavuttaa sama tulos
- `sudo nano /var/www/html/index.html` — avataan tiedosto editorissa root-oikeuksin ja kirjoitetaan teksti käsin
- `sudoedit /var/www/html/index.html` — turvallinen tapa muokata järjestelmätiedostoja väliaikaisen kopion kautta
- `sudo cp myfile.html /var/www/html/index.html` — kopioidaan valmiiksi tehty tiedosto olemassa olevan tilalle

### Miksi `sudo echo 'text' > /var/www/html/index.html` ei toimi

Uudelleenohjausoperaattorin `>` suorittaa itse **shell (bash)**, ei komento (`echo`), johon sudo kohdistuu. Kun kirjoitetaan `sudo echo ...`, `sudo` nostaa oikeudet vain `echo`-prosessille, mutta itse kirjoitusoperaation tiedostoon (`>`) suorittaa edelleen tavallinen shell-prosessi ilman root-oikeuksia — sillä ei ole pääsyä kansioon `/var/www/`, mistä seuraa "Permission denied" -virhe. `tee` puolestaan on erillinen ohjelma, joka itse avaa ja kirjoittaa tiedostoon; `sudo tee` antaa root-oikeudet nimenomaan tälle kirjoitusprosessille, joten komento toimii.
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/bfbde63b-edac-44a9-ab20-84907a44af4a" />

## 4. /etc/hosts-tiedoston määrittäminen

```bash
sudo nano /etc/hosts
```
Lisättiin rivit:
```
127.0.0.1    site1.com
127.0.0.1    www.site1.com
127.0.0.1    site2.com
127.0.0.1    www.site2.com
```

Tarkistus:
```bash
ping -c 4 site1.com
```
Tulos: `PING site1.com (127.0.0.1) 56(84) bytes of data` — 4 pakettia lähetetty, 4 vastaanotettu, 0 % hävikkiä, vastaus tulee osoitteesta `127.0.0.1` (localhost) — vahvistaa, että `/etc/hosts` toimii oikein ja tietokone ratkaisee `site1.com`-nimen itseensä viittaavaksi osoitteeksi.
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/7dbda59e-c21b-410b-8fec-5e281b477ebf" />

## 5. ufw:n asennus ja määrittäminen

```bash
sudo apt-get install ufw
sudo ufw allow ssh
sudo ufw enable
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Sääntöjen tarkistus:
```bash
sudo ufw status verbose
```
```
Status: active
22/tcp    ALLOW IN    Anywhere
80/tcp    ALLOW IN    Anywhere
443/tcp   ALLOW IN    Anywhere
(+ vastaavat säännöt IPv6:lle)
```

### Testi: ufw ja localhost

```bash
sudo ufw deny 80/tcp
sudo ufw status verbose
```
Portin 80 sääntö muuttui muotoon `DENY IN Anywhere`.

```bash
curl localhost
curl site1.com
```
**Tulos:** vaikka portti 80 oli nimenomaisesti estetty ufw:n säännöissä, molemmat pyynnöt — `curl localhost` ja `curl site1.com` — **toimivat onnistuneesti** ja palauttivat odotetun sisällön.

**Johtopäätös:** ufw ei suodata liikennettä, joka kulkee loopback-rajapinnan (127.0.0.1) kautta. ufw:n `deny`/`allow`-säännöt kohdistuvat ulkoiseen verkkoliikenteeseen, ei tietokoneen omiin, itseensä kohdistuviin yhteyksiin. Tämä tarkoittaa, että porttien estäminen palomuurissa ei suojaa paikallisilta (localhost) yhteyksiltä — jos halutaan täysin estää pääsy vain paikallisesti käytettävään palveluun, tarvitaan muita mekanismeja (esimerkiksi palvelu voidaan asettaa kuuntelemaan vain tiettyä verkkorajapintaa).

Testin jälkeen sääntö palautettiin:
```bash
sudo ufw allow 80/tcp
```
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/9fd44bed-9a1b-486c-b04a-ff1a6cfb1270" />
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/4ddc6ff3-9854-43b2-a02e-ce9c1e86bccc" />

## 6. Name-based Virtual Hostin luominen (site1.com)

Sivuston kansion ja tiedoston luonti kotihakemistoon (ilman sudoa):
```bash
mkdir -p ~/public-sites
echo "<h1> my first virtual host</h1>" > ~/public-sites/index.html
```

Konfiguraatiotiedoston luonti:
```bash
sudo nano /etc/apache2/sites-available/site1.com.conf
```
```apache
<VirtualHost *:80>
    ServerName site1.com
    ServerAlias www.site1.com
    DocumentRoot /home/alina/public-sites/

    <Directory /home/alina/public-sites/>
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error-site1.log
    CustomLog ${APACHE_LOG_DIR}/access-site1.log combined
</VirtualHost>
```

### Käyttöoikeuksien tarkistus ja korjaus

```bash
ls -la ~/public-sites/
```
```
drwxrwxr-x  2 alina alina 4096  (kansio public-sites)
-rw-rw-r--  1 alina alina   32  index.html
```
Tiedoston ja kansion `public-sites` oikeudet olivat jo oletuksena kunnossa (lukuoikeus "muille" oli olemassa).

```bash
ls -la /home/
```
```
drwx------ 17 alina alina  alina   ← ei x-oikeutta muille
```
Kotihakemistolla `/home/alina` ei ollut Apachelle (käyttäjä `www-data`) edes oikeutta "astua sisään" kansioon. Korjattiin:
```bash
chmod o+x /home/alina
```
Tämän jälkeen oikeudet muuttuivat muotoon `drwx-----x` — muille annettiin vain oikeus kulkea kansion läpi, ei oikeutta selata muuta kotihakemiston sisältöä.

### Sivuston käyttöönotto

```bash
sudo a2ensite site1.com.conf
```
```
Enabling site site1.com.
To activate the new configuration, you need to run: systemctl reload apache2
```

Symbolisen linkin tarkistus:
```bash
ls -la /etc/apache2/sites-enabled/
```
```
site1.com.conf -> ../sites-available/site1.com.conf
```

Konfiguraation syntaksin tarkistus:
```bash
sudo apache2ctl configtest
# Syntax OK
```
(mukana ei-kriittinen varoitus AH00558 määrittelemättömästä globaalista ServerNamesta)

Uudelleenlataus ja testaus:
```bash
sudo systemctl reload apache2
curl site1.com
# <h1> my first virtual host</h1>
```

Virtual host luotiin onnistuneesti ja toimii — kun sivustoa pyydetään nimellä `site1.com`, Apache tarjoilee sisällön tiedostosta `~/public-sites/index.html` oletussivun sijaan.
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/8323363c-b7eb-4e1b-bacf-01c3b2aa16a4" />
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/294975eb-5ef8-4ee4-99a2-a742df3c9a0e" />
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/963f093f-7e3a-4bdf-b238-29d735874126" />

## 7. Lokien tarkistus

### Systemd journal

```bash
sudo journalctl -u apache2 -n 100 | grep -i error
```
Tulos: tyhjä tuloste — palvelutason virheitä ei havaittu, Apache käynnistyy ja lataa asetukset uudelleen normaalisti.

Tavanomaiset merkinnät journalissa käynnistyksen/uudelleenlatauksen yhteydessä:
```
systemd[1]: Starting apache2.service - The Apache HTTP Server...
apachectl[...]: AH00558: apache2: Could not reliably determine the server's fully qualified domain name...
systemd[1]: Started apache2.service - The Apache HTTP Server.
```

### site1.com:in access-loki

```bash
sudo tail -f /var/log/apache2/access-site1.log
```
```
127.0.0.1 - - [06/Sep/2026:18:52:05 +0300] "GET / HTTP/1.1" 200 259 "-" "curl/8.14.1"
127.0.0.1 - - [06/Sep/2026:20:15:53 +0300] "GET / HTTP/1.1" 200 259 "-" "curl/8.14.1"
127.0.0.1 - - [06/Sep/2026:20:46:50 +0300] "GET / HTTP/1.1" 200 259 "-" "curl/8.14.1"
```
Jokainen rivi on erillinen `curl site1.com`-pyyntö sivustolle, vastauskoodi `200` vahvistaa, että pyynnöt käsiteltiin onnistuneesti.

### Tahallinen konfiguraatiovirhe

Tiedostoon `site1.com.conf` lisättiin tahallinen kirjoitusvirhe `DocumentRoot`-polkuun:
```apache
DocumentRoot /home/alina/public-sates/
```
(oikea muoto olisi `public-sites`)

```bash
sudo systemctl reload apache2
sudo journalctl -u apache2 -n 20
```
Journaliin ilmestyi merkintä:
```
apachectl[5881]: AH00112: Warning: DocumentRoot [/home/alina/public-sates/] does not exist
```
Apache kirjasi varoituksen olemattomasta hakemistosta, mutta palvelu latautui silti uudelleen onnistuneesti (`Reloaded apache2.service`) — virhe ei kaatanut koko Apachea, vaan vaikutti ainoastaan tähän tiettyyn virtual hostiin.

Sivuston tarkistus virhetilanteessa:
```bash
curl site1.com
```
```
403 Forbidden
You don't have permission to access this resource.
Apache/2.4.68 (Debian) Server at site1.com Port 80
```
Koska DocumentRootiin määritettyä kansiota ei ollut olemassa, Apache ei löytänyt tarjoiltavia tiedostoja ja palautti kävijälle **403 Forbidden** -virheen.

Kirjoitusvirheen korjauksen (`public-sites`) ja uudelleenlatauksen jälkeen sivusto toimi jälleen odotetusti:
```bash
curl site1.com
# <h1> my first virtual host</h1>
```
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/8c2d8d1b-483e-4610-bf08-08ee9ad14c98" />
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/4caba7e5-6639-4dfe-8048-b2c2f08e0267" />
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/94d54ee8-a18b-4e72-962d-8f1d601d533e" />
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/5d3a06aa-45d4-478c-be84-9d2882dd590f" />

## 8. Challenge — toinen virtual host (site2.com)

```bash
mkdir -p ~/public-sites-2
echo "<h1>second virtual host</h1>" > ~/public-sites-2/index.html
```

```bash
sudo nano /etc/apache2/sites-available/site2.com.conf
```
```apache
<VirtualHost *:80>
    ServerName site2.com
    ServerAlias www.site2.com
    DocumentRoot /home/alina/public-sites-2/

    <Directory /home/alina/public-sites-2/>
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error-site2.log
    CustomLog ${APACHE_LOG_DIR}/access-site2.log combined
</VirtualHost>
```

```bash
sudo a2ensite site2.com.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

Molempien sivustojen lopullinen tarkistus:
```bash
curl site1.com
# <h1> my first virtual host</h1>

curl site2.com
# <h1>second virtual host</h1>
```

Molemmat sivustot toimivat samanaikaisesti samassa IP-osoitteessa (127.0.0.1), mutta näyttävät eri sisällön riippuen pyynnössä käytetystä verkkotunnuksesta — tämä vahvistaa, että name-based virtual hosting on onnistuneesti määritetty.

