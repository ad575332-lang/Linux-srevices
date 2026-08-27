# Raportti 2 — Linux Komentorivi
 
## 1. Basic Commands
 
Kansioiden luominen kotidirektorissa:
```
~/practice/
├── docs/
├── images/
├── backups/
└── archive/
```
 
Komennot:
```bash
mkdir practice
cd practice
mkdir docs images backups archive
cd docs
touch notes1.txt notes2.txt notes3.txt notes4.txt
```
 
10 eläimien lisääminen `notes1.txt`:ssa `nano` kautta:
```
koira, kissa, puppu, tiikeri, karhu, hevonen, lehmä, appina, leijona, kani
```
 
10 heldelmien/vihanksien lisääminen `notes2.txt`:ssa `nano` kautta:
```
omena, banaani, appelsiini, sitruuna, vesimeloni, porkkana, tomaatti, kurkku, peruna, salaatti
```
 
Tiedostojen nimien vaihtaaminen:
```bash
mv notes1.txt animals.txt
mv notes2.txt vegetables.txt
```
 
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/d5a91ffd-4188-460f-8cae-223543451f28" />

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/8ff5786a-c6b0-4215-bd79-0240e0a9743e" />
 
## 2. Grep и Pipe
 
### fruits.txt
```bash
echo -e "apple\nbanana\norange\nApple pie" > fruits.txt
```
 
Komentojen tulokset:
| Komento | Tulos |
|---|---|
| `grep apple fruits.txt` | `apple` |
| `grep -i apple fruits.txt` | `apple`, `Apple pie` |
| `grep -n apple fruits.txt` | `1:apple` |
| `grep -ni apple fruits.txt` | `1:apple`, `4:Apple pie` |
| `grep -v apple fruits.txt` | `banana`, `orange`, `Apple pie` |
 
**Yhteenveto:** `-i` tekee haun kirjainkoko epäherkkäksi, `-n` näyttää rivin numeron, `-v` näyttää rivit ilman sovitusta.
 
### wc fruits.txt:lla
```bash
wc -l fruits.txt   # 4  (rivien määrä)
wc -w fruits.txt   # 5  (sanojen määrä)
wc -c fruits.txt   # 30 (merkkien määrä)
```
 
### animals.txt ja pipe
```bash
echo -e "dog\ncat\ncatfish\ncow" > animals.txt
 
cat animals.txt | grep -ni cat
# 2:cat
# 3:catfish
 
cat animals.txt | wc -l
# 4
```
 
**Yhteenveto:** Pipe (`|`) antaa yhden komennon tulos kuin toisen komennon inputi, ja ei tarvita enää kirjoittaa paljon koodia.
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/867d712c-6194-40d9-9135-e01e9719757f" />
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/4a0f28d1-d8ab-4107-9185-5b921eb08884" />

 
## 3. GPL-2 License
 
```bash
wc -l /usr/share/common-licenses/GPL-2
# 338 rivit
```
 
```bash
grep GNU /usr/share/common-licenses/GPL-2
```
Se näyttää kaikkii rivit jotka sisältävät "GNU" sanan - maininnat lisenssin nimestä, Free Software Foundation-organisaatiosta ja siihen liittyvistä ehdoista.
```bash
grep -c GNU /usr/share/common-licenses/GPL-2
# 8
```
`-c` rivien määrä.
 
```bash
grep license /usr/share/common-licenses/GPL-2
grep -i license /usr/share/common-licenses/GPL-2
```
Toinen vaihtoehto `-i` kautta etsiä enemmän sopivat sanat koska harkitsevät "License" isoimalla kirjaimella.
 
**Oma GPL_2 komento:**
```bash
grep -ni GNU /usr/share/common-licenses/GPL-2 | wc -w
# 92
```
Kysely etsii merkkijonoja sanalla " GNU " (asia-epäherkkä, rivinumeroilla) ja käyttää putkea laskeakseen löydettyjen merkkijonojen sanojen määrän. 
** Lyhyt yhteenveto GPL-2: sta:**
- Lisenssi antaa oikeuden vapaasti kopioida, muokata ja levittää ohjelmistoa.
- Johdannaisteokset on jaettava samoin ehdoin (copyleft-periaate).
- Lähdekoodin on oltava saatavilla ohjelman mukana.
- Lisenssi ei anna mitään takuita — ohjelma toimitetaan "sellaisenaan" ilman tekijöiden vastuuta mahdollisista virheistä.
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/24bccfc3-20aa-4f01-9efc-d024818e6768" />
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/e723dfeb-4afa-4450-85dc-27944a50ab41" />
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/2c9113f7-15e9-4bf1-a42e-fe0798632a11" />

## 4. btop
 
Asennus:
```bash
sudo apt-get install btop
```
 
Tarkastaminen FHS-tiedostojen sijainti:
```bash
dpkg -L btop
```
Näyttää, että paketti asentaa binäärien lisäksi myös teemat ('/usr/share/btop/themes/..."), dokumentaatio ("/usr/share/doc/btop/..."), kuvakkeet ("/usr/share/icons/..." ) ja man-The page ("/usr/share/man/man1/btop.1.gz`). 
Konfiguroinnin käsittely:
```bash
ls ~/.config/btop/
cp ~/.config/btop/btop.conf ~/.config/btop/btop.conf.orig
nano ~/.config/btop/btop.conf
```
'Update_ms `(päivitysnopeus) ja` proc_tree' (prosessipuunäkymä) parametreja on muutettu. Varmuuskopio alkuperäisestä config tehtiin ennen muutoksia. 
**Kuorman tuottaminen:**
```bash
ping -i 0.1 8.8.8.8
```
Btop: ssa pingin ollessa käynnissä verkkoliikenteen kasvu näkyi Verkko — lataus-ja Latausosioissa, tiedonsiirto (KB/s) näkyi, kun taas levossa liikenne oli lähellä nollaa. 
Prosessorin kuormitustesti:
```bash
yes > /dev/null
```
Btop:ssa tulos on heti näkyvissä: rivi "kyllä" näkyy prosessiluettelossa, jonka suorittimen kuormitus on " 80,5%", core **C1 nousee 100%**: iin (toinen ydin C0 pysyy tavanomaisella 30%: n tasollaan), suorittimen kokonaiskuorma kasvoi 46%: iin. Samaan aikaan muistin käyttö (Mem) ei ole juurikaan muuttunut (1,42 GiB), mikä vahvistaa, että "kyllä" lataa prosessorin, ei muistia.
 
Järjestelmän tilan kuvaus btop-tietojen mukaan:
btop-näytöt: suorittimen käyttö kokonaisuutena ja kunkin ytimen osalta erikseen prosessorin taajuus ja kuormituskeskiarvo; RAM-muistin käyttö (yhteensä/käytetty/saatavilla/välimuistissa/Vapaa); levyn ja I/O-tilan; verkkoliikenne (Lataa/Lataa); luettelo käynnissä olevista prosesseista PID, käyttäjä, muistin käyttö ja CPU%. Verkkokuorman ('ping`) aikana verkko — osiossa näkyy aktiivisuus alhaisella suorittimen kuormituksella; prosessorin kuormituksen (`Kyllä > /dev/null') aikana yhden ytimen kuormituksen odotetaan kasvavan 100%: iin lähes ilman muistin muutoksia-tämä osoittaa eron verkko-ja laskentakuorman välillä.
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/6093d35d-871f-49cb-bca8-7f4a6329f651" />
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/e0a068ba-2cf5-427a-a168-3ed8f375a105" />
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/ce5d27d9-096e-45e8-b9bd-7c14aaec0bfe" />
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/98b39e3e-c0e7-41ad-90fd-deee657f24bd" />

## 5. Oman CLI-ohjelman asennus:
 
Asennettu ohjelma `cowsay`:
```bash
sudo apt install cowsay
cowsay "hello"
```
 <img width="895" height="566" alt="image" src="https://github.com/user-attachments/assets/d8cf79cc-372b-4cd0-8e1b-baf333b07231" />

Ohjelma tuottaa tekstiä käsin piirretyn lehmän ASCII-"kopion" muodossa — sitä käytetään kevytmielisenä esimerkkinä konsoliohjelmasta asennuksen harjoitteluun apt: n kautta. Asetuksia ei ole muutettu — ohjelma toimii heti asennuksen jälkeen ilman asetustiedostoa. 
