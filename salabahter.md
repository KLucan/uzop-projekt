# ČIŠĆENJE PODATAKA
Q: Koje smo vrste problema u podacima identificirali u prvom dijelu projekta?

A:
- konstantne vrijednosti - uklonili smo tehnički stupac index
- nismo našli monotone vrijednosti
- nadomjestili smo nepostojeće vrijednosti
    - date_unregistration - -1 za nepostojeći zapis
    - datum registracije - zamjenjen medijanom
    - clicks_pre_start, clicks_post_start, clicks_total - ispunjeni s 0
    - first_assignment_score - -1 za nedostjajuću vrijednost + posebna bool varijabla za provjeru je li student predao ili nije prvi zadatak
    - socioekonomski status - zamjenjeni nakčešćom vrijednošću

Q: Kako smo odredili first_assignment?

A: Poredali smo sve zadatke koji nemaju vrstu "exam" po datumu i za svakog studenta uzeli prvi.

Q: Kako smo konstruirali varijablu clicks_pre_start?

A: Zbrojili smo sve klikove za jednog studenta za datume  manje od 0 (0 označava početak studija)

Q: Kako smo učitali podatke iz 2. dijela projekta?

A: Izvezli smo očišćene pdoatke u .csv kojeg smo čitali za 3. dio projekta.

Q: Koje smo začajke transformirali iz kategoričkih u numeričke i kako?

A:
- 2 binarne značajke gender_num i disability_num preslikali u 1/0
- socioekonomski status, stupanj edukacije, dob - jednostavno enumerirali

Q: Kako smo normalizirali podatke?

A: scikit MinMaxScaler pri pripremi podataka, za BART također StandardScaler

- MinMaxScaler - skalira podatke da budu u rasponu [0,1]
- StandardScaler - normalizira podatke da spadaju unutar normalne razdiobe

# Kovarijacijska matrica

Q: Kako ste ispitali linearni odnos između ulaznih varijabli i ciljne varijable?

A: Pearsonova korelacijska analiza

Q: Što ste zaključili iz kovarijacijske matrice?

A: Korelacijski koeficijenti niski, najizraženija korelacija između clicks_pre_start i i ocjene prvog zadatka, no dalje umjerena. Rezultati nam se podudaraju s člankom.

# MODELI I REZULTATI

## Metrike

Q: Koje ste metrike koristili pri evaluaciji modela?

A:
- matrica zabune
    - TP, TN, FP, FN
    - brzi uvid u klasifikaciju skupa
    - vrijednosti korištene za druge metrike
    - preciznost - TP / (TP + FP)
    - odaziv - TP / (TP + FN)
- točnost
    - omjer ispravno klasificiranh primjera u odnosu na ukupan broj 
    - jednostavna za izračun, intuitivna
    - kod neuravnoteženih skupova može zavaravati
- F1-score
    - harmonijska sredina preciznosti i odziva
    - dobra mjera kod neuravnoteženih podataka
- ROC krivulja
    - odnos stope TP i FP za različite pragove klasifikacije
    - uvid u ponašanje pri različitim pragovima klasifikacije
    - iz nje možemo učitati optimalni prag - za BART smo to napravili
- AUC
    - površina ispod ROC krivulje
    - vjerojatnost da će model rangirati nasumično odabrani pozitivni primjer višlje od nasumično izabranog negativnog

## Decision Tree
Q: Kako radi Decision Tree?
A: Dijeli podatke u hijerarhijske čvorove prema kriteriju informacijske dobiti. Jednostavan je, ali sklon prenaučenosti.

Q: Kako se Decision Tree ponašao na našem skupu?
A:
- ukupna točnost: 0.39
- macro F1: 0.37
- najbolja klasa: Withdrawn
- najgora klasa: Distinction

Q: Zašto su rezultati loši za Distinction i Fail?
A: Zbog neuravnoteženosti — stablo preferira većinske klase i teško pronalazi granice za rijetke ishode. (ovo možemo praktički za svaki model reć)

## Random Forest
Q: Zašto smo koristili Random Forest?
A: RF smanjuje varijancu kombiniranjem više stabala i bolje generalizira od pojedinačnog stabla.

Q: Koje su performanse Random Foresta?
A:
- ukupna točnost: 0.52
- macro F1: 0.43
- najbolje klasa: Withdrawn i Pass
- najgora klasa: Distinction

Q: Kako RF radi i zašto je bolji od DT?
A:
- više stabala, uzima se klasa koju bira njih najiše
- tijekom izgradnje stabala izbacuju se neke značajke
- bolja generalizacija, smanjuje overfitting

## BART (Bayesian Additive Regression Trees)
Q: Kako radi BART?
A: BART je Bayesovski ansambl regresijskih stabala koji daje distribuciju predikcija, a ne samo jednu vrijednost. Radi u One-vs-Rest režimu.

Q: Kako smo određivali optimalni threshold?
A: Iz ROC krivulje

Q: Koji su rezultati BART modela?
A: 
- ukupna točnost: 0.69
- macro F1: 0.52
- najbolje klasa: Withdrawn i Pass
- najgora klasa: Distinction

## 📈 Logistička regresija
Q: Zašto smo koristili logističku regresiju?
A: Kao baseline linearni model za usporedbu s nelinearnim modelima.

Q: Kako se LR ponašala na našem skupu?
A:
- ukupna točnost: 0.57
- macro F1: 0.37
- najbolje klasa: Withdrawn i Pass
- najgora klasa: Distinction

Q: Zašto LR loše razdvaja Distinction i Pass?
A: Granica između tih klasa nije linearna — potrebni su nelinearni modeli poput RF, BART ili XGBoost.

## XGBoost - naš pokušaj korištenja novog modela
Q: Zašto smo koristili XGBoost
A: Slična metoda metodama iz članka, još uvijek stvara stabla, htjeli smo usporediti ovu noviju metodu s već korišetnima

Q: Kako se LR ponašala na našem skupu?
A:
- ukupna točnost: 0.57
- macro F1: 0.44
- najbolje klasa: Withdrawn i Pass
- najgora klasa: Distinction

Q: Što smo zaključili iz rezultata

A: Uspjeli smo postići bolje rezultate od svih modela osim BART-a. Optimizacijom hiperparametara (Karlo) nismo uspjeli doći do velikog poboljšanja. Ipak, treba napomenuti da je XGBoost puno brži od BART-a, sličniji jednostavnijim modelima. Kada bi proveli više vremena uz optimizaciju hipermparametara i možda dodatno uredili podatke kako bi više odgovarali modelu, vjerojatno bi mogli postići slične rezultate BART-u, ali puno brže.

📉 METRIKE I ANALIZA
## 🎯 Accuracy
Q: Zašto accuracy nije dobra metrika u ovom projektu?
A: Zbog neuravnoteženosti — model može ignorirati male klase i svejedno imati visoku točnost.

## 🎯 Precision
Q: Što znači visoka precision?
A: Model rijetko daje lažno pozitivne predikcije.

Q: Koja klasa ima najbolju precision?
A: Withdrawn (posebno kod Random Foresta).

## 🎯 Recall
Q: Što znači visok recall?
A: Model uspješno pronalazi većinu stvarnih pozitivnih primjera.

Q: Koja klasa ima najniži recall u svim modelima?
A: Fail — najteža klasa za predikciju.

## 🎯 F1-score
Q: Zašto je F1-score važan?
A: Kombinira precision i recall, posebno koristan kod neuravnoteženih klasa.

## 🎯 ROC i AUC
Q: Što prikazuje ROC krivulja?
A: Odnos između TPR i FPR pri različitim pragovima.

Q: Što znači AUC = 0.8?
A: Model će u 80% slučajeva rangirati pozitivni primjer iznad negativnog.

Q: Koji model ima najbolje AUC rezultate?
A: BART

🧠 ZAVRŠNA PITANJA ZA PREDAJU
Q: Koji model biste odabrali kao najbolji?
A: BART, ali je dosta spor i zauzima puno memorije.

Q: Koja je najveća slabost svih modela?
A: Neuravnoteženost klasa, posebno Fail i Distinction.

Q: Kako biste poboljšali rezultate?
A:

koristiti SMOTE (oversampling) - probali neuspješno
optimizirati hiperparametre (posebno za XGBoost) - probali neuspješno
dodati nove značajke (npr. dinamika klikova kroz vrijeme) - nismo imali vremena