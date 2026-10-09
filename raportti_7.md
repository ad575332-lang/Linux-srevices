# Raportti — Linux Exercises, Module 7 (Shell scripting basics)

## 1. Bash ja `~/.bashrc`

`~/.bashrc` on skripti, jonka bash suorittaa aina, kun interaktiivinen pääte käynnistyy. Tiedosto on piilotettu (nimi alkaa pisteellä), joten se näkyy vain komennolla `ls -a`.

```bash
ls -la ~/.bashrc
# -rw-r--r-- 1 alina alina 3881 Feb 15 2026 /home/alina/.bashrc
cat ~/.bashrc
```

<img width="1060" height="860" alt="image" src="https://github.com/user-attachments/assets/82318b40-0e8d-44dd-b22a-76e3d1e02495" />
 — `ls -la ~/.bashrc` ja tiedoston alku (alkuperäiset asetukset)

### Varmuuskopio

```bash
cp ~/.bashrc ~/.bashrc.orig
ls -a
```

Listaukseen ilmestyi tiedosto `.bashrc.orig`.

**Havainto:** ensimmäinen yritys `cp .bashrc.orig testfolder` antoi virheen `No such file or directory`, koska tiedostoa `.bashrc.orig` ei vielä ollut. `cp`:n ensimmäinen argumentti on kopioitava tiedosto ja viimeinen kohde (kansio tai uusi nimi). Oikea komento: `cp ~/.bashrc ~/.bashrc.orig`.

Tarkistin myös, että `which` etsii vain `$PATH`:ssa olevia suoritettavia tiedostoja, joten tavallista tekstitiedostoa `.bashrc` se ei löydä.

<img width="1568" height="434" alt="image" src="https://github.com/user-attachments/assets/4b578c03-e5fc-4d9d-ae2d-38ef5b2455b0" />
 — varmuuskopion luonti ja `ls -a`, jossa näkyy `.bashrc.orig`

## 2. Tervetuloteksti

Lisäsin `~/.bashrc`-tiedoston loppuun rivin:

```bash
echo "Hello, Linuxuser"
```

```bash
nano ~/.bashrc
source ~/.bashrc
# Hello, Linuxuser
```

Tulos: `Hello, Linuxuser` tulostuu aina, kun pääte avataan (ja kun ajetaan `source ~/.bashrc`).

<img width="1512" height="807" alt="image" src="https://github.com/user-attachments/assets/8fb1d0f4-e2f2-4ee0-b278-5dc03e4f5356" />
<img width="1534" height="784" alt="image" src="https://github.com/user-attachments/assets/841a67b2-5311-4b5d-bc5b-5f9a51eb1d77" />
 — tervetuloteksti komennon `source ~/.bashrc` jälkeen

## 3. Kaksi aliasta

Alias on lyhyt nimi pidemmälle komennolle. `=`-merkin ympärillä ei saa olla välilyöntejä.

```bash
alias su-gi='sudo apt-get install'
alias ll='ls -la'
```

Ensimmäisen aliaksen ensimmäinen versio oli:

```bash
alias su-gi='sudo apt-get update'
```

Komento `su-gi python3-pip` antoi virheen:

```
E: The update command takes no arguments
```

**Syy:** alias-komennon perään kirjoitetut argumentit liitetään loppuun, jolloin tuli `sudo apt-get update python3-pip`, eikä `update` hyväksy argumentteja. **Korjaus:** alias päättyy sanaan `install`, jolloin `su-gi python3-pip` muuttuu muotoon `sudo apt-get install python3-pip`. Korjauksen jälkeen komento toimi: `python3-pip is already the newest version (24.0+dfsg-1ubuntu1.3)`.

Uudet aliakset eivät näkyneet ennen kuin ajoin `source ~/.bashrc` (tai avasin uuden terminaalin). Pelkkä `source` ilman tiedostoa antaa virheen `filename argument required`.

**Havainto `ll`-aliaksesta:** Ubuntun `.bashrc`:ssä `ll` on jo määritelty muodossa `ls -alF`. Oma aliakseni on tiedostossa sen alapuolella, joten se ohittaa oletuksen (viimeinen määrittely voittaa).

Tarkistus:

```bash
alias
```
<img width="1534" height="784" alt="image" src="https://github.com/user-attachments/assets/b8f37a7b-b9b8-4aef-a272-4d29ec785a92" />
 — `update`-virhe ja aliaslista ja toimiva `su-gi python3-pip`

`~/.bashrc`-tiedoston loppu:

```bash
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
export PATH="$HOME/.local/bin:$PATH"
echo "Hello, Linuxuser"
alias su-gi='sudo apt-get install'
alias ll='ls -la'
alias rm='rm -i'
```

<img width="1512" height="794" alt="image" src="https://github.com/user-attachments/assets/b6392f0a-14d1-4312-9500-e409d48b390b" />
 — `tail -20 ~/.bashrc`

## 4. HISTSIZE ja HISTFILESIZE

- `HISTSIZE` — kuinka monta komentoa historia säilyttää **muistissa** avoimessa terminaali-ikkunassa.
- `HISTFILESIZE` — kuinka monta riviä tiedostossa `~/.bash_history` säilytetään.
- Historia kirjoitetaan tiedostoon ikkunaa suljettaessa; pakotettuna komennolla `history -a`.

Ennen muutosta (oletusarvot `~/.bashrc`-tiedostossa): `HISTSIZE=1000`, `HISTFILESIZE=2000`. Vieressä on `HISTCONTROL=ignoreboth` — historiaan ei tallennu kaksoiskappaleita eikä välilyönnillä alkavia komentoja.

Muutin `~/.bashrc`-tiedostossa:

```bash
HISTSIZE=5
HISTFILESIZE=10
```

```bash
source ~/.bashrc
history
```

Tulos: ennen muutosta `history` näytti satoja komentoja (numerointi ulottui lukuun 479). Muutoksen jälkeen `history` näytti vain **5 viimeisintä komentoa** (numerot 475–479: `nano ~/.bashrc`, `source ~/.bashrc`, `pwd`, `ls`, `history`).

**Johtopäätös:** `HISTSIZE` rajoittaa, kuinka monta komentoa kuori muistaa avoimessa ikkunassa; vanhat komennot syrjäytyvät uusien tieltä. `HISTFILESIZE=10` rajaa samalla tavalla tiedoston `~/.bash_history` 10 riviin tallennuksen yhteydessä. Kokeen jälkeen arvot kannattaa palauttaa arvoihin 1000/2000 (tai suurempiin), muuten historia on lähes hyödytön.

<img width="1288" height="928" alt="image" src="https://github.com/user-attachments/assets/119cb3bc-c2d1-4264-945c-afc69f41de66" />
 — `HISTSIZE=5` ja `HISTFILESIZE=10` nanossa sekä `history`-tuloste

## 5. Challenge — oma `.bashrc`

| Asetus | Miksi | Miten testattu |
|---|---|---|
| `PS1='\t \u:\w\$ '` (`.bashrc`:n rivit 60 ja 62) | Kehotteessa näkyy kellonaika (`\t`), käyttäjänimi (`\u`) ja nykyinen kansio (`\w`). Kellonaika näkyy jokaisen komennon kohdalla, mikä helpottaa virheiden jäljitystä ja kuvakaappauksia. Koneen nimi ja väri poistettiin, jolloin kehote lyhenee | Ensin väliaikaisesti terminaalissa: `echo $PS1` näytti oletusarvon, sitten `PS1='\t \u:\w\$ '` antoi kehotteen `19:29:21 alina:~$`. Tämän jälkeen rivi kirjoitettiin `.bashrc`-tiedostoon |
| `alias rm='rm -i'` | `rm` kysyy vahvistuksen ennen tiedoston poistamista, mikä suojaa vahingossa tehdyltä poistolta | [TÄYTÄ: luo testitiedosto, poista se `source ~/.bashrc`:n jälkeen ja näytä vahvistuskysely] |
| `alias ll='ls -la'` | Lyhyt komento yksityiskohtaiselle tiedostolistalle, piilotetut tiedostot mukaan lukien | [TÄYTÄ: näytä `ll`-komennon tuloste `source ~/.bashrc`:n jälkeen] |

Selitykset:
- Oletus-`PS1` on `.bashrc`:ssä kahdessa `if [ "$color_prompt" = yes ]` -haarassa: värillinen pääte (rivi 60) ja mustavalkoinen (rivi 62). Muutin molemmat, jotta tulos ei riipu siitä, tunnistiko bash värituen.
- Rivi 69 (`PS1="\[\e]0;...$PS1"`) vain lisää terminaali-ikkunan otsikon, joten en muuttanut sitä.
- Ennen muokkausta tehtiin kopio `.bashrc.orig`, joten virhetilanteessa oletustilan saa takaisin: `cp ~/.bashrc.orig ~/.bashrc`.

<img width="1290" height="370" alt="image" src="https://github.com/user-attachments/assets/6fcf669a-6ca3-4ba2-b1a2-ed07e28d5009" />
 — `.bashrc`:n rivit 118–123 aliaksineen (`ll` ja `rm`)
<img width="1479" height="812" alt="image" src="https://github.com/user-attachments/assets/acc36587-2022-48c2-801f-57952f6bd406" />
 — uusi kehote kellonajalla ja `.bashrc`:n rivit 59–63 muutetulla `PS1`:llä

## 6. Shell-skripti (vaihtoehto a)

Tehtävä: luo tiedosto `info.txt`, kirjoita siihen käyttäjänimi ja päivämäärä sekä listaa hakemiston sisältö.

Tiedosto `myscript.sh`:

```bash
#!/bin/bash
touch projects/info.txt
echo $USER > projects/info.txt
date >> projects/info.txt
ls projects
```

Ajo:

```bash
chmod +x myscript.sh
./myscript.sh
```

- `#!/bin/bash` — kertoo, millä tulkilla skripti ajetaan.
- `chmod +x` — antaa suoritusoikeuden.
- `./` — ajaa tiedoston nykyisestä kansiosta (nykyinen kansio ei ole `$PATH`:issa).
- `touch` — luo tyhjän tiedoston, jos sitä ei ole.
- `$USER` — muuttuja, jossa on nykyisen käyttäjän nimi; `date` tulostaa päivämäärän ja ajan.
- `>` korvaa tiedoston sisällön, `>>` lisää loppuun.

Tulos:

```bash
cat projects/info.txt
```
```
alina
Fri Oct  9 18:13:40 EEST 2026
```

Skripti tulostaa `ls projects`:

```
demo_txt.py  file.txt  info.txt  main  main1.ipynb  pycharm1.py  requirements.txt
```

**Uudelleenajon tarkistus:** poistin tiedoston `projects/info.txt` komennolla `rm projects/info.txt`, minkä jälkeen `./myscript.sh` loi sen uudelleen ja `info.txt` näkyi taas listassa.

**Havainnot:**
- Komento `echo $USER > project/info.txt` antoi virheen `No such file or directory`, koska hakemistoa `project` ei ollut — `>` luo tiedoston mutta ei kansioita (siihen tarvitaan `mkdir` / `mkdir -p`).
- Skripti toimii jo olemassa olevassa kansiossa `~/projects`, joten listauksessa näkyy vanhoja tiedostoja.
- Skriptin toisen rivin `>` korvaa `info.txt`:n sisällön joka ajokerralla, ja kolmannen rivin `>>` lisää päivämäärän nimen perään. Alun `touch` ei ole välttämätön, koska `>` luo tiedoston itsekin, mutta se tekee skriptin tarkoituksen selväksi.

<img width="465" height="372" alt="image" src="https://github.com/user-attachments/assets/2ca238b3-35ca-476b-bd92-c0508ac09c1f" />
 — skriptin koodi nanossa
<img width="1512" height="794" alt="image" src="https://github.com/user-attachments/assets/c75c7757-59d9-42fb-8227-9168474c9f3e" />
 — `chmod +x`, `./myscript.sh` ja `cat projects/info.txt`

## Yhteenveto

- `~/.bashrc` on paikka henkilökohtaisille pääteasetuksille: aliakset, tervetuloteksti, muuttujat ja `PS1`; ennen muokkausta kannattaa tehdä varmuuskopio.
- Muutokset tulevat voimaan komennolla `source ~/.bashrc` tai uudessa ikkunassa.
- Alias vain korvaa tekstin, joten argumentit liitetään komennon loppuun.
- `HISTSIZE` ja `HISTFILESIZE` ohjaavat historian pituutta muistissa ja tiedostossa.
- Skripti on tiedosto, jossa on komentoja, shebang ja suoritusoikeus; `>` ja `>>` käyttäytyvät eri tavalla toistuvissa ajoissa.
