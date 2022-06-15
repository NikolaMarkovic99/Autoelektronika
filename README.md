# Autoelektronika
Upravljanje klimom u automobilu 

# Ideja i zadaci
1.	Podaci o temperaturi u automobilu se dobijaju automatski sa dva senzora, svakih 200 ms, sa kanala 0 i 1 serijske komunikacije. 
2.	Trenutna temperatura se dobija kao srednja vrijednost očitanih temperatura, koje se šalju sa dva senzora.
3.	Preko simulirane serijske komunikacije šaljemo naredbe za AUTOMATSKI ili MANUELNI režim rada, kao i podešavanje željene temperature i histerezisa. Podaci sa senzora se šalju svakih 1000 ms.
4.	U MANUELNOM režimu rada, imamo dva stanja klime – uključeno i isključeno, što se kontroliše preko prekidača. Pomenuti prekidač simuliramo preko donje diode na ulaznom stupcu LED bar-a. Kada je stanje uključeno, dioda svijetli, a kada je isključeno ne svijetli. 
5.	Sledeće tri diode na ulaznom stupcu predstavljaju prekidače za aktiviranje ventilatora u kolima. Ukoliko je klima uključena i ventilator u jednom od pomenuta tri stanja, na terminalu se ispisuje stanje ventilatora. 
6.	U AUTOMATSKOM režimu, klima se kontroliše prema senzorima temperature. Na osnovu zadate željene temperature i histrerezisa, omogućeno je automatsko upravljanje klimom. Histerezis može biti u opsegu od 0.1 do 5, a željena temperatura od 16 do 30. 
7.	Na 7seg displeju se prikazuju vrijednosti trenutne temperature, minimalne ili maksilmalne temperature kao i to da li je klima u MANUELNOM ili AUTOMATSKOM režimu rada. Ukoliko je na prvom displeju 0, režim rada klime je MANUELNI, a ako je 1 onda je AUTOMATSKI. Naredna dva displeja prikazuju trenutnu temperaturu, dok poslednja dva displeja prikazuju minimalnu ili maksimalnu vrijednost temperature (u zavisnosti od stanja gornjeg prekidača na ulaznom stupcu). Brzina osvježavanja displeja je 200 ms.

# Periferije
Periferije koje je potrebno koristiti su LED_bar, 7seg displej i AdvUniCom softver za simulaciju serijske komunikacije. Prilikom pokretanja LED_bars_plus.exe navesti rRR kao argument da bi se dobio LED bar sa jednim ulaznim i 2 izlazna stupca crvene boje. Prilikom pokretanja Seg7_Mux.exe navesti kao argument broj 5, kako bi se dobio 7-seg displej sa 5 cifara. Što se tiče serijske komunikacije, potrebno je otvoriti kanal 0, kanal 1 i kanal 2. Kanal 0 se automatski otvara pokretanjem AdvUniCom.exe, a kanal 1 i kanal 2 otvoriti dodavanjem broja 1 i 2 kao argumenta: AdvUniCom.exe 1 i AdvUniCom.exe 2.

# Kratak pregled taskova
Glavni .c fajl ovog projekta je main_application.c

## void led_bar_tsk( void* pvParameters )
Task koji očitava stanje tastera pritisnutih na LED bar-u, na osnovu kojih se određuje stanje klime i ventilatora, kao i indikatora za prikaz minimalne odnosno maksimalne temperature na 7seg displeju. Takođe, unutar ovog taska vrši se setovanje odnosno resetovanje dioda, kao i smještanje stanja ventilatora u red. 

## void SerialSend_Task(void* pvParameters)
U ovom tasku se vrši ispis poruka na serijsku. Karakter po karakter se ispisuje željena poruka. Kada se poruka ispiše do kraja (kada je brojač stigao do dužine riječi), onda se daje semafor da je ispis završen. Zadatak ovog taska je da primi poruku u niz, primi dužinu poruke i da ispiše poruku od nultog karaktera do karaktera čiji je indeks dužina poruke i da označi završetak radnje.

## void SerialSend_Task0(void* pvParameters)
S obzirom da vrijednost trenutne temperature treba dobiti svakih 200 ms sa kanala 0 serijske komunikacije, od strane FreeRTOS-a je to omogućeno tako što se u ovom tasku svakih 200 ms šalje karakter 'A' preko kanala 0. Kada se pokrene AdvUniCom.exe potrebno je označiti opciju Auto, odnosno svaki put kad stigne karakter 'A', da se pošalje naredba oblika 00xx\0d. (npr 0025\0d). Vrijednost koja se pošalje je zapravo vrijednost trenutne temperature (npr. 25). Povremeno je potrebno ručno u AdvUniCom softveru mijenjati ovu vrijednost kako bi se simulirala promjena temperature.

## void SerialSend_Task1(void* pvParameters)
S obzirom da vrijednost trenutne temperature treba dobiti svakih 200 ms sa kanala 1 serijske komunikacije, od strane FreeRTOS-a je to omogućeno tako što se u ovom tasku svakih 200 ms šalje karakter 'A' preko kanala 1. Kada se pokrene AdvUniCom.exe potrebno je označiti opciju Auto, odnosno svaki put kad stigne karakter 'A', da se pošalje naredba oblika 00xx\0d. (npr 0025\0d). Vrijednost koja se pošalje je zapravo vrijednost trenutne temperature (npr. 25). Povremeno je potrebno ručno u AdvUniCom softveru mijenjati ovu vrijednost kako bi se simulirala promjena temperature.

## void prijem_sa_senzora_tsk(void* pvParameters)
U ovom tasku vrši se očitavanje podataka sa senzora. Na svakih 200 ms, task očitava nove vrijednosti temperature, formira trenutnu temperaturu kao srednju vrijednost pojedinačnih očitavanja senzora i određuje minimum i maksimum mjerenja.

## void Primio_kanal_0(void* pvParameters)
S obzirom da vrijednost trenutne temperature treba dobiti svakih 200 ms sa kanala 0 serijske komunikacije, od strane FreeRTOS-a je to omogućeno tako što se u ovom tasku svakih 200 ms šalju karakteri 'XYZ' preko kanala 0. Kada se pokrene AdvUniCom.exe potrebno je označiti opciju Auto, odnosno svaki put kad stignu karakteri 'XYZ', da se pošalje naredba oblika 00xx\0d (npr. 0025\0d ). Vrijednost koja se pošalje je zapravo vrijednost trenutne temperature (u navedenom primeru 25). Povremeno je potrebno ručno u AdvUniCom softveru mijenjati ovu vrijednost kako bi se simulirala promjena temperature u automobilu.

## void Primio_kanal_1(void* pvParameters) 
S obzirom da vrijednost trenutne temperature treba dobiti svakih 200 ms sa kanala 1 serijske komunikacije, od strane FreeRTOS-a je to omogućeno tako što se u ovom tasku svakih 200 ms šalju karakteri 'XYZ' preko kanala 1. Kada se pokrene AdvUniCom.exe potrebno je označiti opciju Auto, odnosno svaki put kad stignu karakteri 'XYZ', da se pošalje naredba oblika 00xx\0d (npr. 0025\0d). Vrijednost koja se pošalje je zapravo vrijednost trenutne temperature (u navedenom primeru 25). Povremeno je potrebno ručno u AdvUniCom softveru mijenjati ovu vrijednost kako bi se simulirala promjena temperature u automobilu.

## void SerialReceive_Task(void* pvParameters)
Ovaj task ima za zadatak da obradi komandne poruke koje stižu sa kanala 2 serijske komunikacije. Komandu koja stiže kao i njenu dužinu smješta u redove, kako bi ostali taskovi te podatke imali na raspolaganju za dalje računanje. 

## void obrada_podataka_task(void* pvParameters) 
Ovaj task provjerava da li je režim rada klime AUTOMATSKI ili MANUELNO. To provjerava na osnovu dužine riječi koju šaljemo sa kanala 2, što se i ispisuje na kanalu 2. Takođe preko njega ispisujemo željenu temperaturu i histerezis poslate sa kanala 2 u format 0xx\0d (npr. 021\0d za željenu temperaturu i 0.5\0d za histerezis). Pored toga, provjeva još i da li je trenutna temperatura u zadatom opsegu i u zavisnosti od toga pali ili gasi klimu što se prikazuje na prvoj diodi trećeg stupca LED bar-a.

## void Seg7_ispis_task(void* pvParameters)
Ispisuje na 7seg displej informacije o režimu rada klime (0 za MANUELNI režim rada, a 1 za AUTOMATSKI), trenutnu temperaturu kao i maksimalnu ili minimalnu temperaturu, u zavisnosti od toga da li je pritisnut gornji taster prvog stupca LED bar-a. 

## void Serijska_stanje_task(void* pvParameters)
Ovaj task služi za formiranje poruke o stanju sistema. Svakih 1000 ms je potrebno na serijsku ispisati stanje sistema: režim rada, da li je uključen sistem i trenutnu temperaturu. Poruka bi trebalo da bude u sledećoj formi: “Stanje: (AUTOMATSKI/MANUELNO), radi: (UKLJUCENO/ISKLJUCENO), temp: xx”. Da bi se pravilno ispisala poruka, potrebno je voditi računa o dužini poruke i da se naredna riječ nastavlja na prethodni niz, a ne da ga prebriše. U zavisnosti od vrijednosti promjenljivih, formira se poruka i započinje ispis. Kada se ispis završi, čeka se da prođe vrijeme do kreiranja sljedeće poruke.

## void main_demo(void)
U ovoj funkciji se vrši inicijalizacija svih periferija koje se koriste, kreiraju se taskovi, semafori i red, definiše se interrupt za serijsku komunikaciju i poziva
vTaskStartScheduler(), funkcija koja aktivira planer za raspoređivanje taskova.

# Predlog simulacije cijelog sistema

Prvo otvoriti sve periferije na način opisan gore. Pokrenuti program. U prozoru LED_bars_plus donji prekidač u prvom stupcu predstavlja prekidač za uključivanje klime. Treba ga pritisnuti da bi stanje klime prešlo u UKLJUČENO. Tada se u prozoru kanala 2 ispisuje da je klima uključena. Takođe pale se donje diode u drugom i trećem stupcu LED bar-a. Ukoliko pritisnemo jedan od naredna tri tastera na prvom stupcu, aktiviramo ventilator i režim rada ventilatora se takođe ispisuje na kanalu 2. U prozoru AdvUniCom softvera kanala 0 i kanala 1, upisati da kada stignu karakteri 'XYZ', kanal 0 šalje naredbu formata npr. 00xx\0d (npr. 0021\0d gdje 0d označava kraj naredbe). Čekirati Auto i ok1. Na taj način šaljemo trenutne temperature sa kanala 0 i 1 (u ovom slučaju temperatura je 21) svakih 200 ms. Na kanalu 2 serijske komunikacije poslati AUTOMATSKI\0d kako bi aktivirali automatski režim rada klime. Tada na displeju vidimo koji je režim rada klime (1 na prvom displeju), kao i trenutnu temperaturu (vrijednost na drugom i trećem displeju). Poslednja dva displeja prikazuju maksimalnu ili minimalnu temperaturu u zavisnosti od toga da li je pritisnut gornji prekidač na prvom stupcu LED bar-a (kada je pritisnut, prikazuje se maksimalna temperatura i obrnuto). Preko kanala 2 šaljemo željenu temperaturu i histerezis. Potrebno je poslati tri karaktera (npr. 019\0d za željenu temperaturu ili 0.5\0d za histerezis i slično). Željena temperature koja se šalje mora biti iz opsega 16 do 30, a histerezis od 0.1 do 5. Ukoliko je stanje klime uključeno, režim rada AUTOMATSKI i trenutna temperatura veća od željene temperature plus histerezis, pali se prva dioda na trećem stupcu, što označava da se aktivirala klima, u suprotnom dioda ne svijetli. Dioda će svijetliti i ako je stanje uključeno i režim rada MANUELNO. U isključenom stanju, dioda ne svijetli.
