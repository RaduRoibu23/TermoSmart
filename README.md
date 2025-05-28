# TermoSmart

====== TermoSmart Scheduler ======

===== Introducere =====

<note tip>

**Proiectul** vizeaza dezvoltarea unui termostat inteligent care monitorizeaza continuu temperatura ambientala si, in functie de necesar, porneste un ventilator compact (care imita aerul conditionat) pentru racire sau activeaza rezistoare de incalzire (care imita caloriferul), mentinand astfel permanent un confort termic optim in incapere si optimizand consumul de energie.
</note>
 **Ce face:** Masoara temperatura ambientala si controleaza automat centrala si aerul conditionat.

 **Care este scopul lui:** Ofera economii de energie si confort optim prin pornirea/opirea HVAC conform programarilor.

 **Care a fost ideea de la care ati pornit:** Am plecat de la ideea de a simplifica controlul temperaturii, eliminând nevoia de a jongla între termostat și telecomanda aerului condiționat. Un singur dispozitiv integrat permite menținerea usoara a temperaturii ideale și optimizeaza consumul de energie.

 **De ce credeti ca este util pentru altii si pentru voi:** Simplifica administrarea climatizarii in locuinte si reduce costurile cu incalzirea si racirea.  

===== Descriere generala =====

<note tip>
**Functionalitati:**
 **Incalzire si racire automate** – Porneste incalzirea sau racirea pana la atingerea setpoint-ului definit sau pana la oprire manuala.

 **Mentinere temperatura constanta** – Odata atins pragul setat se comuta in modul „mentinere” pentru a pastra temperatura dorita pana la primirea altei comenzi.

 **Alarma de blocaj** – Semnaleaza daca sistemul functioneaza prea mult timp exclusiv in modul incalzire sau racire. Se semnaleaza printr-un semafor in functie de nivelul de blocaj si un buzzer.

 **Interfata utilizator intuitiva** – Meniu cu butoane dedicate („incalzire”, „racire”, „mentinere”) si afisaj digital al deciziei curente.

</note>

===== Laboratoare folosite =====


 **Laboratorul 0 (GPIO)** – Citirea semnalelor de la butoanele de selectare a modului (incalzire, racire, mentinere) si controlul semaforului cu LED-uri (verde, galben, rosu) prin software, pe baza pragurilor de temperatura.

 **Laboratorul 1 (UART)** – Interfata seriala cu PC-ul pentru configurarea pragurilor de temperatura si afisarea in timp real a starii sistemului.
  
 **Laboratorul 3 (Timere & PWM)** – Generarea intreruperilor periodice (ex. la fiecare 1 s) necesare esantionarii senzorului de temperatura si modularea vitezei ventilatorului prin semnal PWM.  

 **Laboratorul 4 (ADC)** – Citirea semnalului analogic de la senzorul NTC 10 kΩ si conversia lui in valori digitale de temperatura.  


===== Hardware Design =====

{{:pm:prj2025:rnedelcu:poza_pm_circuit_hard.jpg?450|}}

<note tip>

 **BOM (Bill Of Materials)**

 Arduino Uno R3 : Creierul proiectului; alimentare 9V extern - [[https://ardushop.ro/ro/plci-de-dezvoltare/2282-placa-de-dezvoltare-uno-r3-compatibil-arduino-6427854027122.html|Link]]

 Semafor LED (Roșu/Galben/Verde) : Semnalizare stare/eroare; D9–D11 - [[https://www.emag.ro/mini-semafor-cu-led-5v-ai0005-s30/pd/DY8G1GMBM/?ref=similar_products_7_1&provider=rec&recid=rec_4_cbc5b68a802be6e2535b086cb0fdc960007238ca187d8edb40b54d368f82eb48_1747216087&scenario_ID=4|Link]]

 Buzzer Activ : Semnalizare stare/eroare; - [[https://www.optimusdigital.ro/ro/audio-buzzere/12247-buzzer-pasiv-de-33v-sau-3v.html|Link]]

 Senzor temperatura : Senzorul de detectare al temperaturii; - [[https://www.optimusdigital.ro/ro/senzori-senzori-de-temperatura/114-modul-senzor-de-temperatura-cu-termistor.html|Link]]

 Ventilator : Pentru a raci senzorul de temperatura, imita aerul conditionat; - [[https://ardushop.ro/ro/raspberry-pi/475-ventilator-5v-30mm-raspberry-pi-6427854005489.html|Link]]

 Butoane : Cate un buton pentru fiecare mod (incalzire/racire/mentinere) si 2 butoane pentru a creste sau a scadea in mentinere; - [[https://ardushop.ro/ro/butoane--switch-uri/713-buton-mic-push-button-trough-hole-6427854009050.html|Link]]

 Breadbord : Montaj prototip; alimentare rails și grupuri de câte 5 orizontal; - [[https://ardushop.ro/ro/electronica/84-breadboard-400-6427854020949.html|Link]]
 
 Adaptor DC 9 V : Alimentare externă pentru Arduino (în jack); - [[https://www.emag.ro/adaptor-baterie-9v-rosfix-conector-dc-2-1-5-5-lungime-cablu-15cm-ibtl-3jcb/pd/DRL8PSYBM/?gQT=1|Link]]



</note>


{{:pm:prj2025:rnedelcu:poza_buna_hard.png?450|}}{{:pm:prj2025:rnedelcu:schemablockradu.png?450|}}

===== Software Design =====

**Descrierea codului:**

* **Mediu de dezvoltare:** Arduino IDE

* **Librarii si surse 3rd-party:**

  * Functii native Arduino (`analogRead()`, `digitalWrite()`, `tone()`, `noTone()`, `Serial`, `millis()`, `delay()`)

* **Algoritmi si structuri planificate:**

  * Citire analogica de la senzorul de temperatura si conversie ADC→°C (Laboratorul 4 - ADC)
  * Gestionarea starii si a modului de functionare prin intermediul a 5 butoane configurate cu `INPUT_PULLUP` (Laboratorul 0 - GPIO)
  * Semaforizare stari sistem (verde, galben, rosu) folosind LED-uri RGB (Laboratorul 0 - GPIO)
  * Control ventilator si incalzire (logica digitala) in functie de temperatura si setari
  * Alarma progresiva indicand starea curenta prin LED-uri si semnal sonor cu tonalitati diferite
  * Interfata UART pentru afisarea starii curente si actualizarea temperaturii dorite in timp real (Laboratorul 1 - UART)

* **Surse si functii implementate:**

  * `setup()` – configurare initiala a pinilor GPIO si initializare UART
  * `loop()` – orchestrare citire temperatura, verificare butoane, actualizare stari, control semafor si ventilator
  * Citire analogica `analogRead(A0)` si conversie din tensiune in °C (Laboratorul 4 - ADC)
  * `handleSemafor(bool activeFan)` – gestioneaza activarea semaforului si controlul ventilatorului (Laboratorul 0 - GPIO)
  * Logica pentru ajustare dinamica temperatura dorita (`BUTTON4` si `BUTTON5`)
  * Feedback in timp real prin UART privind temperatura actuala si actiunile intreprinse (Laboratorul 1 - UART)

===== Rezultate Obtinuite =====

In urma realizarii proiectului, software-ul implementat functioneaza conform specificatiilor si indeplineste obiectivele stabilite. Cu toate acestea, am intampinat dificultati minore in gestionarea ventilatorului, cauzate de limitarile hardware, ceea ce a determinat ca ventilatorul sa nu functioneze intotdeauna optim.

===== Concluzii =====

* Proiectul s-a dovedit provocator din punct de vedere hardware si mediu din punct de vedere software. Cu toate acestea, am reusit sa finalizam ambele componente cu succes. Am dedicat aproximativ 45 de ore acestui proiect si nu am intampinat probleme majore, cum ar fi defectarea componentelor hardware. In concluzie, am obtinut un proiect functional la standardele dorite.
===== Download =====

<note warning>
O arhivă (sau mai multe dacă este cazul) cu fişierele obţinute în urma realizării proiectului: surse, scheme, etc. Un fişier README, un ChangeLog, un script de compilare şi copiere automată pe uC crează întotdeauna o impresie bună ;-).

Fişierele se încarcă pe wiki folosind facilitatea *Add Images or other files. Namespace-ul în care se încarcă fişierele este de tipul **:pm:prj20??:c?* sau *:pm:prj20??:c?:nume_student* (dacă este cazul). *Exemplu:* Dumitru Alin, 331CC -> *:pm:prj2009:cc:dumitru_alin*.
</note>

===== Jurnal =====

<note tip>
Puteți avea și o secțiune de jurnal în care să poată urmări asistentul de proiect progresul proiectului.
</note>

===== Bibliografie/Resurse =====

<note>
Listă cu documente, datasheet-uri, resurse Internet folosite, eventual grupate pe *Resurse Software* şi *Resurse Hardware*.
</note>

<html><a class="media mediafile mf_pdf" href="?do=export_pdf">Export to PDF</a></html>
