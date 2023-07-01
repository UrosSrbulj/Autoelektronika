# Autoelektronika
Provera stanja vrata
# Uvod
Ideja ovog projekta da se napise softver koji ce da proverava stanja vrata na automobilu.Sofvter treba da proverava da li su vrata otvorena,koja su vrata i da li se automobil krece brzinom vecom od dozvoljene maskimalne brzine pri otvorenim(zatvorenim)vratima.Preko simulatora serijske komunikacije (kanal 0)salju se podaci o vratima i brizni vozila.Ako je ispunjen uslov da su vrata otvorena i brzina veca od maksimalne, u zavisnosti koja su vrata otvorena program ce raditi sledece:

1.Vazi za sva vrata osim za gepek - ispisuje na displeju koja su vrata ,dolazi do blinkanja dioda i ispusuje se poruka upozorenja na PC(kanal) kao  i koja su vrata otvorena.

2.Vazi samo za gepek- pali se alarm koji se kotrolise pomocu tastera.Kad je alarm ukljucen treba da se upale sve diode,a kad je iskljucen diode treba da se ugase i na displeju  da se ispise broj vrata.Takodje i u ovom slucaju ispisuje se poruka upozorenja na PC.Postoji opcija da se alarm ugasi pomocu simulatora serijske komunikacije(preko kanala 1) tako sto se posalje odredjena poruka.
# Periferije 
1.AdvUniCom - softver za serijsku komunikaciju(koriste se kanali 0 i 1)

2.LED_bars_plus-prvi stubac je ulazni(taster),ostali izlazni-pokretanjem komandom LED_bars_plus.exe rRGB(jedan ulazni i tri izlazna stubca)

3.Seg7_Mux - displej - pokretanje komandom Seg7_Mux.exe 5 (5 je broj polja na displeju)

# Nacin testiranja programa
Prvo se preko serijske komunikacije na kanalu 0 salju podaci o vratima .Na pocetku poruke se nalazi start bajt(0xef) ,prvi karakter posle start bajt moze biti 1 ili 0 (1 – vrata otvorena,0 -zatvorena),drugi kartker je koja su vrata otvorena(1-prednja leva,2-prednja desna,3-zadnja leva,4 -zadnja desna ,5 gepek) I na kraju poruke ide stop bajt(0xff).Zatim se salju podaci o brzini takodje na istom kanalu.Takodje se na pocetku poruke nalazi start bajt (0xfe) ,prvi karakter su stotine,drugi desetice i treci jedinice(pr.poslato \1\2\3 – 123km/h) na kraju ide stop bajt(0xee).Podaci o vratima smestaju se u strukturu pod nazivom “Mystruct”.Polje koje opisuje stanje vrata naziva se “stanje.vrata”,a polje koje opisuje koja su vrata naziva se “vrata”,dok se podaci o brzni smestaju u baffer. Ukoliko dodje do slucaja da su vrata otvorena i brzina veca od maksimalne (5km/h) program se deli na dva dela u zavisnosti koja su vrata otvorena.Ako su otvorena vrata 1,2,3 ili 4 dolazi do blinkanja  izlaznih dioda na Led baru ,ispisuje  poruka na displej(door1,door2,door3,door4 u zavisnosti od broja vrata) i salje se poruka upozorenja na PC  kao i koja su vrata otvorena.
Drugi slucaj je kad je otovren gepek(vrata 5),tada imamo alarm koji se ukljucuje  ili iskljucuje pomocu tastera na Led baru. Kad je taster pritisnut ,alarm je ukljucen I  sve diode na led baru su ukljucene.Kad je taster ugasen ,alarm je iskljucen ,diode su ugasene ,a na Seg7 displeju  je ispisano door5.Takodje se I u ovom slucaju na kanalu 1 ispisuje poruka upozorenja.Alarm se moze ugasiti I preko serijske komunikacije (preko kanala1 ) tako sto ce se poslati komanda 0x0d.
# Kratak opis taskova
SerialReceive_Task0(void* Parameters) – task sluzi da preuzme podatke sa kanala 0 serijske komunikacije ,da ih stavi u zasebne redove  i posalje dalje na obradu.(podatke o vratima I brzini).

Obrada_vrata_Task(void* Parameters) – task koji sluzi da obradi podatke koje je poslao SerialReceive_Task0  ,ispise ih na ekranu i posalje dalje na obradu.

Obrada_brzine_Task(void* Parameters)- task sluzi da primi podatke poslate od strane taska SerialReceive_Task0(brzina) i taska Obrada_vrata_Task(stanje vrata).Racuna trenutnu brzinu i proverava uslov da li je trenutna brzina veca od maksimalne.Ako je uslov zadovoljen i ako su vrata otvorena salje  semafore i podatke koji su smesteni u red drugim taskovima.

SerialSend_Task(void* pvParameters) - task sluzi da ispise  poruku upozorenja kao i  koja su vrata otvorena na simulator serijske komunikacije (PC).

Led_Displej_Task(void* pvParameters) - task sluzi da ispise koja su vrata otvorena na seg7 displeju i da ukljuci diode da blinkaju.

Alarm_Task(void* pvParameters) - task sluzi da u zavisnosti da od stanja tastera ukljuci ili iskljuci alarm.

Stop_Alarm_Task(void* pvParameters)- task sluzi da ugasi alarm kad stigne poruka sa PC kanala 1.
 
SerialReceive_Task1(void* pvParameters) -  task sluzi da preuzme podatke sa kanala 1 serijske komunikacije i posalje ih odgovarajucem tasku na dalju obradu.






