# gitea-setup
Step by step instructions for installing standalone gitea server (with explanations)

## Opste informacije

Gitea je self-hosted Git like VCS servis. Gitea (Git with a cup of tea) ima podrsku za :
-   code reviews
-   PR 
-   registar paketa
-   CI/CD.

Slican je GItub-u, Bitbucket-u, GitLab-u.

Implementirana je u Golang programskom jeziku te je paltform agnosticna zaL
-   Linux
-   macOS
-   Windows

[Online demo verzija](https://demo.gitea.com/)

## Glavne mogucnosti

-   Code hosting - Podrska za kreiranje i rad sa repozitorijumima, pretraga istorije,
reviews i merging sabmisija, upravljanje kolaboratirima, grananje itd. Podrska osnovnih git funkcionalnosti u vidu
tagova, cherry-picks, hooks itd.

-   Brz i lagan - Konzimacija resursa je manja u odnosu na neke druge platforme.

-   Jednostavan deploy i odrzavanje - Vrlo jednostavan deploy na raznim serverima, bez 
kompleksne konfiguracije niti dependencija. Vrlo je pogodan za manje timove. 

-   Security - Akcenat je na bezbednosti, i nudi mogucnosti za permisije , access control 
i druge vidove zastite koda i podataka.

-   Code review - Podrska za Pull Request workflow i AGit workfflow

-   Project management - kreiranje issue-a

-   Artifact repository - Podrska za preko 20 razlicitig alata za upravljanje paketima medju kojima
su i npm i Maven itd.

## Requirements

-   RAspberry Pi 3 
-   2CPU cores i 1GB ram dovoljan za manje timove/projekte
-   Pokretanje gitea procesa treba da bude sa nekim ne root korisnikom (GItea radi sa ~/.ssh/authorized_keys fajl. Pokretanje
procesa sa nekim regularnim korisnikom moze da spreci njegov login.
-   Git verzija >= 2.0.0


## Instalacija


### Baza podataka

Za Gitea servis neophodna je baza podataka. Gitea ima podrsku za:
-   PostgreSQL  (>=12)i
-   MySQL       (>=8.0)
-   SQLite      (ugradjen)

Preporuka je da se koristi PostgreSQL u produkcionom okruzenju.
Setup je sledeci:

1. u okviru postgresql.conf fajla izmeniti password_encryption

```conf
    password_encryption = scram-sha-256
```

2. Na database serveru ulogovati se na db kao super user

```bash
    su -c "psql" - postgres
```

3. Kreirati database korisnika (Bolje koristiti bezbedniju sifru od ovog primera)

```sql
    CREATE ROLE gitea WITH LOGIN PASSWORD 'gitea';
```

4, Kreirati bazu sa UTF-8 charset u vlasnistvu korisnika gore. 

```sql
CREATE DATABASE giteadb WITH OWNER gitea TEMPLATE template0 ENCODING UTF8 LC_COLLATE 'en_US.UTF-8' LC_CTYPE 'en_US.UTF-8';
```

5. Dozvoliti korisniku da pristupi bazi kreiranoj iznad tako sto azuriramo pg_hba.conf

```conf
local    giteadb    gitea    scram-sha-256
```

6. Restart postgreSQL procesa

7. Testirati konekciju ka bazi podataka

```bash
    psql -U gitea -d giteadb
```

### Preuzimanje i verifikacija verzije git-a

1. Preuzeti binary koristeci wget komandu (promeniti verziju po potrebi)

```bash
    wget -O gitea https://dl.gitea.com/gitea/1.23.8/gitea-1.23.8-linux-amd64
    chmod +x gitea
```

2. GPG verifikacija. Preuzeti signature fajl (zavrsava se sa .asc) i verifikovati
postojeci binary na osnovu njega.

```bash
    gpg --keyserver keys.openpgp.org --recv 7C9E68152594688862D62AF62D9AE806EC1592E2
    gpg --verify gitea-1.23.8-linux-amd64.asc gitea-1.23.8-linux-amd64
```

3. Verzija gita. Uveriti se da je instalirana verzija gita >= 2.0.0

```bash
    git --version
```

### Priprema git okruzenja

1. Dodati korisnika koji ce pokretati gitea

```bash
# On Ubuntu/Debian:
adduser \
   --system \
   --shell /bin/bash \
   --gecos 'Git Version Control' \
   --group \
   --disabled-password \
   --home /home/git \
   git
```

2. Organizacija direktorijuma na git serveru

```bash
mkdir -p /var/lib/gitea/{custom,data,log}
chown -R git:git /var/lib/gitea/
chmod -R 750 /var/lib/gitea/
mkdir /etc/gitea
chown root:git /etc/gitea
chmod 770 /etc/gitea
```

3. Konfiguracija radnog direkortijuma za Gitea-u

```bash
    export GITEA_WORK_DIR=/var/lib/gitea
    cp gitea /usr/local/bin/gitea
```

4. Pokretanje Gitea servisa (kao Linux servis)

```bash
sudo systemctl enable gitea
sudo systemctl start gitea
```

5. Restart
```bash
systemctl restart gitea
```

## Administracija

