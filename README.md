# Helsinki University SQL Basics Spring 2021

This repo contains the second task for the course. It's a schema and a database running it, you can find the database
diagram here: https://dbdiagram.io/d/603f931ffcdcb6230b226465

In the file: 6 `Tietokannan suunnittelu - Tietokantojen perusteet kevät 2021.html` there are some resources on how to design
a database, written in finnish.

Source of course material: https://tikape.mooc.fi/kevat-2021/

## Instructions for the assignment, in finnish:

```
Tehtäväsi on suunnitella tietokanta, jota voitaisiin käyttää verkkokaupan taustalla. Suunnittele tietokanta niin, että siihen voidaan tallentaa tietosisältö seuraavia toimintoja varten:

1. Verkkokaupassa on tuotteita, joilla on nimi, kuvaus ja hinta.
2. Käyttäjä pystyy etsimään tuotteita ja hänellä on ostoskori, johon voi lisätä ja poistaa tuotteita.
3. Tuotteita voidaan luokitella tuoteryhmiin. Sama tuote voi kuulua useaan tuoteryhmään.
4. Kun tilaus on valmis, käyttäjä pystyy antamaan yhteystietonsa ja pankkikorttinsa tiedot, minkä jälkeen tilaus siirtyy käsittelyyn.
5. Käyttäjä pystyy arvostelemaan tuotteita ja lukemaan muiden käyttäjien antamia arvosteluja.
6. Tuotteista voidaan muodostaa tarjouspaketteja, joissa tietyt tuotteet saa tiettyyn hintaan pakettina.
7. Käyttäjä näkee tuotteen kuvauksen yhteydessä, paljonko tuotetta on jäljellä missäkin verkkokaupan varastossa.
8. Käyttäjä voi saada alennuskoodin, jonka voi syöttää tilauksen yhteydessä. Alennuskoodin voi käyttää vain kerran.
9. Käyttäjä näkee tuotteen yhteydessä esimerkkejä, mitä tuotteita muut tämän tuotteen ostaneen asiakkaat ovat ostaneet.

Tehtäväsi on suunnitella, mitä tauluja tietokannassa on, mitä sarakkeita kussakin taulussa on ja miten taulut viittaavat toisiinsa. Jos yllä oleva kuvaus ei kerro jotain yksityiskohtaa, tee jokin järkevä päätös tämän asian suhteen.
Suunnittele tietokanta luvun 6 periaatteiden mukaisesti ja esitä tietokannan rakenne SQL-skeemana ja tietokantakaaviona. Selosta lisäksi jokaisesta yllä olevasta vaatimuksesta (1–9), mitä tietoa tietokantaan tallennetaan, jotta kyseinen vaatimus saadaan toteutettua.

```

## My explanations on how the system works, in finnish

```
1. Tuotteet tallennetaan tietokannan tauluun Tuotteet. Jokaiselle tuotteelle määritellään sarakkeet id, nimi, kuvaus sekä hinta.

2. Käyttäjät tallennetaan tietokannan tauluun Kayttajat. Jokaiselle käyttäjälle voidaan määritellään ostoskori lisäämällä uusi rivi Ostoskorit tauluun.

Ostoskoriin voidaan lisätä uusi tuote, lisäämällä uusi rivi tauluun OstoskoriTuote. Tässä taulussa on viitesarakkeet ostoskori_id ja tuote_id, joiden avulla voimme määrittää mitä tuotteita mihinkin ostoskoriin kuuluu. Tuotteen voi poistaa, poistamalla kyseisen rivin taulusta.

3. Tietokantaan on lisätty taulu TuoteRyhma, joka määrittää kaikki tuoteryhmät. Tuotteen voi lisätä tuoteryhmään tekemällä siitä tuoteryhmäyksen tauluun TuoteRyhmaus. Tämä toimii samalla periaatteella kuin 2 kohdan OstoskoriTuote.

4. Käyttäjien yhteys- ja pankkitiedot tallennetaan tauluun KayttajaTiedot. Taulussa on viitesarake itse käyttäjään. Kun käyttäjä on lisännyt tietonsa voidaan tilausta vastaavat rivit taulussa TilausTuote asettaa käsittelyyn muuttamalla sen saraketta tila. Sarake tila on merkkijono ja se voidaan tarpeen mukaan asettaa mihin tahansa haluttuun arvoon. Se mahdollistaa myös sen että tilaus voi olla osittain käsitelty tai kokonaan, jos kaikkien tilausta vastaavien rivien tila on asetettu haluttuun arvoon.

5. Arvostelu voidaan lisätä, lisäämällä uusi rivi tauluun Arvostelut. Uudelle riville tulee asettaa  kommentti, käyttäjän id- ja tuotteen id viitesarakeen arvo.

6. Tarjouspaketit määritellään taulussa Tarjoukset, jonka uusille riveille lisätään hinta. Kun tarjous on luotu, voidaan siihen lisätä tuotteita lisäämällä rivi TarjousTuote tauluun.

Kun käyttäjä lisää tarjouksen ostoskoriin, 2 kohdassa mainittuun OstoskoriTuote tauluun listätään uusi rivi. Tällä kertaa tuote_idn sijaan, täytetään sarakkeen tarjous_id arvo vastaamaan kyseisen tarjouksen idtä. Tämä tarkoittaa sitä että toinen näistä viitearvoista tulee olemaan arvoltaan NULL. Mutta, tämä helpottaa kyselyiden suorittamista.

7. Aloitetaan lisäämällä uusi rivi Varastot tauluun. Tämän jälkeen varastoon voidaan lisätä tuote, lisäämällä uuden rivin VarastoTuote tauluun. Varasto tuotteella on viitesarakkeet itse varastoon, sekä kyseiseen tuotteeseen. Varastossa tulee olemaan useita samoja tuotteita joka kasvattaa tietokannan kokoa, mutta tämä tapa vastaa paremmin oikeaa tilannetta, joka vuorostaan on helpompi ymmärtää.

8. Uuden alennuskoodin, eli kupongin, voi luoda lisäämällä uuden rivin tauluun Kupongit. Kun käyttäjä syöttää kupongin koodin, kupongin alennus otetaan huomioon lopullisessa tuotteiden hinnassa. Kun maksusuoritus on onnistunut, kuponki voidaan asettaa käytetyksi muuttamalla sen käytetty saraketta 'false' arvoon.

Toistaiseksi myös käytetyt kupongit säilytetään kannassa jotta verkkokaupan asiakaspalvelijoiden olisi helpompi selvittää mahdolliset ongelmatilanteet.

9. Kun joku käyttäjistä tekee ostoksen, jokaista ostettua tuotetta kohden lisätään jokaiselle tuotteen kanssa ostetulle tuotteelle rivi tauluun TuoteSuositus. Jokainen rivi kertoo ostetun tuotteen ja tuotteen joka ostettiin sen kanssa.

TuoteSuosituksista voidaan hakea ne tuotteet, jotka ovat yleisimmin ostettu katselussa olevan tuotteen kanssa.

```
