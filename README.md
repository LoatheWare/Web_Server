# Web Server i konfigurisanje Web servera na Windows Server 2019 u VirtualBox okruženju
## Šta je ovo?
Ovo je maturski (završni) ispitni zadatak za smer **Administrator računarskih mreža**, vezan za instalaciju i konfigurisanje **veb servera** u domenskoj mreži. Zadatak obuhvata: instalaciju Windows Server-a i učlanjenje u domen, instalaciju **LAMP/WAMP** komponenti (Apache, MySQL/MariaDB, PHP), pokretanje **CMS** sistema (WordPress/Joomla), kreiranje virtuelnog hosta, dijagnostiku mrežnih problema na klijentskim računarima, i pravljenje rezervnih kopija veb aplikacije i baze podataka. Zadak Vam je dat, kao i uputstvo za njegovo rešavanje. Ukoliko Vam nešto nije jasno tokom rešavanja ovih zadataka obratite mi se direktnom porukom na aplikaciji LinkedIn, www.linkedin.com/in/nikola-karanović-397185390

## Pitanja i odgovori — Web server, baze podataka i CMS

### 1. Šta je veb server i kako funkcioniše?
**Veb server** je softver (npr. **Apache**, Nginx, IIS) koji "sluša" na određenom portu (po default-u **port 80** za HTTP, **443** za HTTPS) i odgovara na zahteve (requests) klijenata — najčešće veb pregledača. Princip rada je **request-response**: klijent šalje HTTP zahtev za određeni resurs (npr. `GET /index.php`), server pronalazi/generiše traženi sadržaj i vraća ga klijentu kao HTTP odgovor (HTML, slike, JSON, itd.).

---

### 2. Šta je LAMP/WAMP stek i koje komponente ga čine?
**LAMP** (na Linux-u) ili **WAMP** (na Windows-u) je skraćenica za skup tehnologija koje zajedno omogućavaju pokretanje dinamičkih veb sajtova:
- **L/W** — Linux ili Windows (operativni sistem)
- **A** — Apache (veb server)
- **M** — MySQL ili MariaDB (server baze podataka)
- **P** — PHP (programski jezik za server-side logiku)

Ove komponente rade zajedno: Apache prima zahtev, prosleđuje PHP interpreteru ako je traženi fajl `.php`, PHP kod po potrebi komunicira sa MySQL/MariaDB bazom radi čitanja/pisanja podataka, a rezultat se vraća kao HTML klijentu.

---

### 3. Šta je virtuelni host (Virtual Host) i čemu služi?
**Virtual Host** je Apache koncept koji omogućava da **jedan fizički server** (jedna IP adresa, jedan Apache servis) hostuje **više različitih sajtova**, svaki sa svojim domenom i svojim direktorijumom na disku. Apache prepoznaje koji sajt treba da prikaže na osnovu **HTTP Host header-a** koji pregledač šalje (npr. zahtev za `www.skola.local` prikazuje sadržaj iz direktorijuma dodeljenog tom hostu, a ne neki drugi sajt na istom serveru).

Primer minimalne virtual host konfiguracije (Apache, Windows — `httpd-vhosts.conf` ili `httpd.conf`):

```apache
<VirtualHost *:80>
    ServerName www.skola.local
    DocumentRoot "${INSTALL_DIR}/www/sajt1"
    <Directory "${INSTALL_DIR}/www/sajt1">
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

---

### 4. Zašto je potrebno podesiti DNS zapis za `www.skola.local` da bi sajt bio vidljiv u lokalnoj mreži?
Veb pregledač prvo mora da **prevede ime hosta u IP adresu** (DNS rezolucija) pre nego što uopšte može da pošalje HTTP zahtev. Pošto `www.skola.local` nije javno ime na internetu, već **privatno ime** koje postoji samo unutar lokalne mreže, potrebno je na **DNS serveru domena** (na domenskom kontroleru, `server1`) kreirati odgovarajući **A zapis** (ili CNAME) koji `www.skola.local` povezuje sa IP adresom veb servera (`server2`, 192.168.1.10). Bez ovog zapisa, klijenti bi dobijali grešku da ime ne može da se reši (resolve), iako bi direktan pristup preko IP adrese radio normalno.

---

### 5. Šta je CMS (Content Management System) i koja je razlika između WordPress-a i Joomla-e?
**CMS** je veb aplikacija koja korisnicima omogućava da **kreiraju, uređuju i upravljaju sadržajem** sajta bez potrebe za direktnim pisanjem koda — kroz administratorski panel. I **WordPress** i **Joomla** su CMS sistemi pisani u **PHP-u** koji koriste **MySQL/MariaDB** bazu za čuvanje sadržaja (postovi, stranice, korisnici, podešavanja). Razlike su uglavnom u:
- **Nameni** — WordPress je prvobitno nastao kao blog platforma (jednostavniji, popularniji za manje sajtove), dok je Joomla od početka građena kao fleksibilniji CMS pogodan za složenije, poslovne sajtove.
- **Strukturi administracije** — Joomla ima koncept **Components/Modules/Plugins**, dok WordPress koristi **Plugins/Themes** kao osnovne building blokove.
- **Krivoj učenja** — WordPress se generalno smatra jednostavnijim za početnike.

Oba sistema, prilikom instalacije, zahtevaju unapred kreiranu **praznu MySQL bazu** i **korisnika sa pravima pristupa** toj bazi.

---

### 6. Koja prava treba dodeliti MySQL korisniku da bi mogao da instalira WordPress/Joomla?
Prilikom instalacionog procesa, CMS (WordPress/Joomla) automatski **kreira tabele** u bazi i **upisuje podatke** u njih, pa korisnik mora imati minimalno: **SELECT, INSERT, UPDATE, DELETE, CREATE, ALTER, DROP, INDEX** privilegije nad konkretnom bazom. U praksi, najjednostavnije i najčešće rešenje (i ono koje zadatak traži — "sva potrebna globalna prava i prava nad kreiranom bazom") je dodela **ALL PRIVILEGES** nad tom konkretnom bazom tom korisniku, čime se pokrivaju sve operacije koje instalacija zahteva.

Ovo se može uraditi i preko SQL upita (umesto kroz phpMyAdmin GUI):

```sql
CREATE DATABASE wordpress_db;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'JakaLozinka123!';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp_user'@'localhost';
FLUSH PRIVILEGES;
```

---

### 7. Šta predstavlja karakter `%` u polju Host name prilikom kreiranja MySQL korisnika?
Polje **Host name** definiše **sa koje mrežne lokacije** korisnik ima dozvolu da se konektuje na MySQL/MariaDB server. Karakter **`%`** je **wildcard** koji znači "**bilo koja adresa**" — korisnik može pristupiti serveru sa bilo kog hosta u mreži (ili interneta, ako server nije zaštićen firewall-om). Ovo je suprotno od npr. `localhost`, gde korisnik može pristupiti **samo sa istog računara** na kome se baza nalazi. U produkcionim okruženjima, `%` se izbegava kad je moguće (manje sigurno), ali se često koristi u laboratorijskim/ispitnim zadacima radi jednostavnosti.

---

### 8. Zašto je obavezno zaštititi pristup serveru baze podataka jakom lozinkom?
Server baze podataka čuva **sve kritične podatke** veb aplikacije (korisnička imena, lozinke — često heširane, sadržaj sajta, konfiguracione podatke). Ako root ili bilo koji privilegovan nalog ima slabu ili nepostojeću lozinku, napadač koji dobije pristup serveru (ili čak samo MySQL portu, **3306**, ako je izložen mreži) može **pročitati, izmeniti ili obrisati kompletnu bazu**, kompromitujući ceo sajt i potencijalno podatke korisnika. Jaka lozinka je osnovna, ali ključna linija odbrane.

---

### 9. Koje su tehnike pravljenja backup-a baze podataka?
Postoji nekoliko pristupa:
1. **Ručno kopiranje fajlova baze** — najbrži način, kopiranjem foldera baze direktno iz **data** direktorijuma servera. Ograničenje: ne radi pouzdano za **InnoDB** tabele, jer se njihovi podaci čuvaju u **zajedničkom fajlu** (`ibdata1`) koji sadrži podatke **svih** InnoDB baza na serveru, ne samo one koju želimo da bekapujemo.
2. **SQL upiti** — napredniji, programerski pristup (van uobičajenog obima administracije).
3. **phpMyAdmin Export** — najpraktičniji način za administratora: izabere se baza, klikne **Export**, izabere **Quick** metoda i format **SQL**, čime se generiše `.sql` fajl sa kompletnom strukturom i podacima baze (**Full Backup**).
4. **mysqldump (komandna linija)** — ekvivalent phpMyAdmin Export-u, ali iz CLI-ja, pogodno za automatizaciju (npr. preko Task Scheduler-a):

mysqldump -u root -p wordpress_db > backup_wordpress.sql

---

### 10. Koja je razlika između Full backup-a i Incremental backup-a?
- **Full (kompletan) backup** — svaki put se eksportuje **cela baza** (struktura + svi podaci), bez obzira na to koliko se promenilo od prethodnog backup-a. Jednostavniji za restauraciju (jedan fajl), ali zahteva više prostora i vremena ako se radi često.
- **Incremental backup** — bekapuju se **samo promene** napravljene od poslednjeg backup-a. Efikasniji u smislu prostora i brzine izvršavanja, ali restauracija je složenija (potrebno je primeniti poslednji full backup, pa zatim sve naredne inkrementalne, redom).

---

### 11. Kako se vraća (restore) baza podataka iz `.sql` backup fajla?
Restauracija se radi **uvozom (import)** prethodno eksportovanog `.sql` fajla, na dva osnovna načina:
- **Kroz phpMyAdmin** — izabere se (ili kreira) ciljna baza, zatim tab **Import**, izabere se `.sql` fajl sa diska i klikne **Go**.
- **Kroz komandnu liniju**:

mysql -u root -p wordpress_db < backup_wordpress.sql

Bitno je da ciljna baza **postoji** (kreirana, prazna) pre uvoza, osim ako `.sql` fajl sam sadrži `CREATE DATABASE` komandu.

---

### 12. Kako se kreira deljeni (shared) direktorijum za smeštaj backup fajlova, i zašto se backup čuva na drugom serveru?
Deljeni direktorijum se kreira na domenskom kontroleru (`server1`) kroz **Properties > Sharing > Advanced Sharing**, gde se definiše **share name** i odgovarajuće **NTFS i share dozvole** za pristup. Backup veb aplikacije i baze se čuva na **drugom serveru** (ne na istom gde se nalazi veb aplikacija) iz razloga **redundantnosti** — ako server na kome je hostovan sajt (`server2`) otkaže (hardverski kvar, ransomware, fizičko oštećenje), rezervna kopija ostaje sačuvana i dostupna za oporavak sa drugog, nezavisnog uređaja.

---

### 13. Kako se instalira i povezuje periferijski uređaj (skener) na domenski kontroler?
Skener se povezuje fizički (USB ili mrežno) na računar koji je deo domena, nakon čega se:
1. Instalira odgovarajući **drajver** za skener (preko Device Manager-a ili instalacionog softvera proizvođača).
2. Provjerava se da li je uređaj prepoznat i funkcionalan kroz **Devices and Printers** (ili **Devices and Scanners** na novijim sistemima) i testira se probnim skeniranjem.
3. Ukoliko se zahteva mrežna dostupnost (deljenje skenera drugim korisnicima domena), konfiguriše se odgovarajuća **WSD (Web Services on Devices)** ili deljena fascikla na koju skener čuva skenirane dokumente.

---

### 14. Analiza problema iz Priloga_1 — zašto korisnik ne može da otvori sajt, a ping radi?
Korisnik ima IP adresu **192.168.1.200** sa **maskom 255.255.255.192 (/26)**, dok je server2 na **192.168.1.10/26**. Mreža `/26` obuhvata adrese **192.168.1.0 – 192.168.1.63** (sa default gateway-em `192.168.1.10` koji je **van** ovog opsega za klijenta!). Problem je u tome što klijent sa adresom **.200** i maskom **/26** zaključuje da je njegova mreža **192.168.1.192 – 192.168.1.255**, dok se gateway (.10) i DNS server (.10) nalaze u **potpuno drugoj** /26 podmreži — pa klijent **ne može direktno da ih kontaktira**, jer ih smatra "udaljenim", a nema definisanu rutu ka njima.

*(Ping ka **192.168.1.10/26** može uspeti jer korisnik to radi sa eksplicitno upisanom **/26 maskom u samoj ping komandi konteksta** ili je u istoj fizičkoj L2 mreži pa ARP/switching i dalje radi na L2 nivou, ali aplikativni saobraćaj/DNS koji zavisi od ispravne L3 logike rutiranja unutar OS-a ima problem.)* Rešenje: ispraviti IP konfiguraciju klijenta tako da koristi ispravnu masku **/26 (255.255.255.192)** i adresu **iz opsega .0–.63** (npr. 192.168.1.20), kako bi gateway i DNS server bili u **istoj logičkoj podmreži**.

---

### 15. Analiza problema iz Priloga_2 — pogrešna maska podmreže
Server2 je na **192.168.1.40/27** (mreža biznis.local obuhvata **192.168.1.32 – 192.168.1.63**), a problematičan klijent ima adresu **192.168.1.50** sa **maskom 255.255.255.0 (/24)**. Klijent sa **/24** maskom misli da je u mreži **192.168.1.0 – 192.168.1.255** (cela "klasa C"), dok je u realnosti pravi gateway (**.63**) deo uže **/27** podmreže. Posledica: klijent **ne pravi razliku** između lokalne i udaljene mreže (smatra celu .0/24 lokalnom), pa **ne koristi gateway** za saobraćaj koji bi (u stvarnoj /27 topologiji) trebalo da ide preko rutera, što remeti komunikaciju i DNS rezoluciju. Rešenje: ispraviti masku klijenta na **255.255.255.224 (/27)**, u skladu sa stvarnom podelom mreže.

---

### 16. Analiza problema iz Priloga_3 — klijent ne može da pinguje NIKOG u lokalnoj mreži
Server2 je na **192.168.1.20/28** (mreža trade.local obuhvata **192.168.1.16 – 192.168.1.31**), a klijent ima adresu **192.168.1.23** sa **maskom 255.255.255.224 (/27)**. Ovo je **potpuno pogrešna, šira maska** — klijent sa `/27` maskom (opseg 192.168.1.0–31, sa svojom granicom na 32) zapravo *jeste* u približno istom opsegu pa bi se očekivalo da L2 radi, ali pošto stvarna mreža je definisana kao **/28** (192.168.1.16–31, gateway .17), pogrešna (šira) maska na klijentu uzrokuje da klijentov OS **drugačije računa broadcast i mrežnu adresu**, što remeti ARP rezoluciju i mogućnost komunikacije sa bilo kojim uređajem — uključujući i gateway. Ovo objašnjava zašto klijent **ne može da pinguje nikog**, ne samo veb server. Rešenje: ispraviti masku na ispravnu vrednost **255.255.255.240 (/28)**, usklađenu sa stvarnom podelom mreže definisanom na domenskom kontroleru.

---

### 17. Koja je opšta lekcija iz sva tri priloga o IP konfiguraciji?
Sva tri scenarija pokazuju isti tip greške — **neusklađenost subnet maske klijenta sa stvarnom podelom mreže** definisanom na ruteru/domenskom kontroleru. Ovo je čest, ali **suptilan** problem u praksi, jer:
- **Ping ka specifičnoj poznatoj adresi** može i dalje funkcionisati u određenim slučajevima (npr. zbog ARP cache-a, ili ako su uređaji na istom fizičkom L2 segmentu bez stvarnog rutiranja između njih), dok se ostala komunikacija (DNS, HTTP) ponaša nepredvidivo.
- **Najpouzdaniji prvi korak dijagnostike** je provera IP konfiguracije klijenta (`ipconfig /all`) i poređenje sa ispravnom mrežnom maskom/opsegom definisanim za tu mrežu, **pre** pretpostavke da je problem u samom serveru, DNS-u ili veb aplikaciji.

---

### 18. Koje su komande za proveru rada veb servera, baze i mreže na klijentu?
Najkorisnije komande za dijagnostiku:

ipconfig /all

ping 192.168.1.10

nslookup www.skola.local

tracert www.skola.local

httpd -t

systemctl status apache2

mysqladmin -u root -p status

- `ipconfig /all` (Windows klijent) — prikazuje trenutnu IP konfiguraciju, masku, gateway i DNS server, ključno za dijagnostiku problema iz priloga.
- `httpd -t` — provjerava sintaksu Apache konfiguracije pre restarta servisa (sprečava pad servera zbog greške u `httpd.conf`).
- `mysqladmin -u root -p status` — prikazuje status MySQL/MariaDB servera (broj konekcija, uptime, itd.), korisno za potvrdu da server baze radi.

---
