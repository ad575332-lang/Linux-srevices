# Raportti — Linux Exercises, Module 6 (Linux as a Development Workstation)

## 1. GitHub — sähköpostiasetukset

Tarkistin GitHub-tilin sähköpostin yksityisyysasetukset (Settings → Emails):
- Asetus "Keep my email address private" on päällä
- Löysin henkilökohtaisen noreply-osoitteen:
```
238963506+ad575332-lang@users.noreply.github.com
```

## 2. Gitin asennus ja määrittäminen

```bash
sudo apt-get install git
git -v
# git version 2.47.3
```

Globaalin identiteetin määrittäminen:
```bash
git config --global user.email "238963506+ad575332-lang@users.noreply.github.com"
git config --global user.name "Alina"
cat ~/.gitconfig
```
```
[user]
	email = 238963506+ad575332-lang@users.noreply.github.com
	name = Alina
```
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/2ad083e9-a88c-406c-b359-d0d0aed7891c" />
 — gitin asennus, user.email/user.name-asetukset, ~/.gitconfig-tiedoston sisältö

## 3. SSH-avaimen luonti GitHubia varten

```bash
ssh-keygen -t ed25519 -C "238963506+ad575332-lang@users.noreply.github.com" -f ~/.ssh/github_key
```
Avain tallennettu selkeällä nimellä (ei oletusnimellä), passphrase jätettiin tyhjäksi harjoitusympäristöä varten.

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/70909713-fa2b-44f7-b550-179425d58d74" />
 (jatkoa) — github_key-avaimen generointi

### ~/.ssh/config-tiedoston määrittäminen

```bash
nano ~/.ssh/config
```
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_key
```
```bash
chmod 600 ~/.ssh/config
```

**Prosessin aikana ilmennyt ongelma:** ensimmäisellä yrityksellä tiedosto luotiin vahingossa väärällä nimellä (`conf` eikä `config`), minkä vuoksi SSH ei tunnistanut sitä ja palautti edelleen virheen `Permission denied (publickey)`. Lisäksi tiedostoon päätyi vahingossa ylimääräinen rivi (`chmod 600 ...`), joka oli kirjoitettu tiedoston sisään terminaalin sijaan. Väärän tiedoston poistamisen (`rm ~/.ssh/conf`) ja uuden, oikein nimetyn tiedoston (`config`) luomisen jälkeen, jossa oli vain neljä tarvittavaa riviä, ongelma korjaantui.

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/5e05b20b-ed69-4e0d-9e22-7aa8466d158f" />

 — oikean ~/.ssh/config-tiedoston sisältö nanossa
 
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/d3c33482-d868-4485-b346-7dc9264c56a3" />

 — vianetsintäprosessi: epäonnistuneet ssh -T -yritykset, tiedostonimen korjaus, lopullinen onnistunut tulos

### Julkisen avaimen lisääminen GitHubiin

```bash
cat ~/.ssh/github_key.pub
```
Julkinen avain lisätty polun GitHub → Settings → SSH and GPG keys → New SSH key kautta.

### Autentikoinnin tarkistus

```bash
ssh -T git@github.com
```
```
Hi ad575332-lang! You've successfully authenticated, but GitHub does not provide shell access.
```
Autentikointi onnistui.

## 4. Testirepositorio git-testing

Kloonaus SSH:n kautta:
```bash
git clone git@github.com:linuxkurssi/git-testing.git
cd git-testing
ls -la
```
```
aura_tip.txt  ismail_tip.txt  justtesting.md  minux_of_the_caribbean.txt
minux_tech_tip.txt  README.md  tompanvinkki.txt  treasure_map.txt
useful_command.txt  vinkkivitonen.txt
```

Toisen opiskelijan vinkin tarkastelu esimerkkinä:
```bash
cat minux_tech_tip.txt
```
```
Ahoy, me hearties!
If recent commands ye wish to see,
The proper spell would be:
history | tail -20
Run it once and ye shall know,
Where all yer latest commands did flow.
```
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/3be58069-76f5-4977-967b-4dfe8139af94" />

 — repositorion kloonaus, tiedostolistaus, toisen vinkin tarkastelu

### Oman tiedoston lisääminen

```bash
nano alina_tips.txt
git status
git add alina_tips.txt
git commit -m "Add my linux tip"
git push
```
```
[main 12f473b] Add my linux tip
 1 file changed, 5 insertions(+)
 create mode 100644 alina_tips.txt
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/17a8e144-83df-4b43-be1e-611ba1a13fc7" />

 — tiedoston luonti, git status/add/commit, onnistunut push

Push onnistui, tiedosto ilmestyi yhteiseen repositorioon muiden opiskelijoiden vinkkien joukkoon.

## 5. Docker

### Asennus (virallisen apt-repositorion kautta)

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/c96807ce-16f6-4960-bf71-d6853290896a" />
 — Dockerin asennusprosessi virallisen repositorion kautta

### Asennuksen testaus

```bash
sudo docker run hello-world
```
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### Toisen imagen ajaminen — interaktiivinen Ubuntu-kontti

```bash
sudo docker run -it ubuntu bash
```
```
root@3363242c9837:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  ...
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/cd1c14e3-b648-4072-849a-bc9fc1d04fed" />
 — onnistunut hello-world ja interaktiivinen ubuntu-kontti

**Havainto:** Docker latasi valmiin `ubuntu:latest`-imagen Docker Hubista ja käynnisti sen interaktiivisessa tilassa (`-it`), jolloin pystyin työskentelemään eristetyssä kontissa kuin erillisessä järjestelmässä — kontti kuitenkin käyttää isäntäkäyttöjärjestelmän (paikallisen VirtualBox-koneen Debianin) ydintä, eikä luo täysin erillistä virtuaalikonetta.

## 6. Oma kehitystyöasema (Development Workstation)

Rakentaisin itselleni ihanteellisen kehittäjän työaseman keskittyen pilvi- ja DevOps-kehitykseen sopivaan työkalusarjaan:

- **Käyttöjärjestelmä:** Debian/Ubuntu — sama jakelu, jonka kanssa olen työskennellyt koko kurssin ajan, yhtenäisyyden vuoksi paikallisen koneen ja pilvipalvelimien välillä
- **Editori:** VS Code — Python-, Terraform- ja Remote-SSH-laajennuksilla (jotta koodia voi muokata suoraan etäpalvelimilla ilman tiedostojen kopiointia edestakaisin)
- **Versionhallintatyökalut:** Git + erikseen määritetyt SSH-avaimet jokaiselle palvelulle (GitHub, työpalvelimet) — kuten tässä moduulissa jo tehtiin
- **Konttiteknologia:** Docker — sovellusten testaamiseen paikallisesti eristetyssä ympäristössä ennen niiden käyttöönottoa oikealla palvelimella
- **Verkon diagnostiikkatyökalut:** jo tutut curl, dig, tcpdump, ngrep — verkko- ja DNS-ongelmien selvittämiseen
- **Infrastructure as Code -työkalut:** Terraform — pilvi-infrastruktuurin kuvaamiseen ja hallintaan koodina

**Mukautukset:** määrittäisin `~/.ssh/config`-tiedoston erillisillä avaimilla eri palvelimille/palveluille (aloitin tämän jo tässä moduulissa), ottaisin käyttöön bashin komentojen automaattisen täydennyksen ja määrittäisin aliakset usein käytetyille pitkille komennoille (esim. `alias dcu="docker compose up"`).

**Asennettu uusi työkalu:** asensin tämän tehtävän yhteydessä Dockerin — työkalun, jonka kanssa en ollut aiemmin tällä kurssilla työskennellyt.
