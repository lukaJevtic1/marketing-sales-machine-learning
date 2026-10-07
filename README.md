# Predviđanje prihoda od prodaje – Marketing & Sales

Seminarski rad iz mašinskog učenja u programskom jeziku **R**. Cilj projekta je sveobuhvatna analiza faktora koji utiču na ostvareni prihod od prodaje (`sales_revenue_usd`) i razvoj regresionih modela koji predviđaju prihod na osnovu marketinških ulaganja, kanala prodaje i ponašanja kupaca.

**Autori:** Danilo Novaković (101/2018), Luka Jevtić (64/2017)  
**Fakultet:** Prirodno-matematički fakultet, Univerzitet u Kragujevcu

---

## Sadržaj

1. [Motivacija](#motivacija)
2. [Podaci](#podaci)
3. [Tok analize](#tok-analize)
4. [Rezultati modelovanja](#rezultati-modelovanja)
5. [Zaključak](#zaključak)
6. [Pokretanje projekta](#pokretanje-projekta)
7. [Struktura repozitorijuma](#struktura-repozitorijuma)


## Motivacija

Analiza i predviđanje prihoda od prodaje ključni su za uspešno poslovanje: omogućavaju kompanijama da pametnije rasporede marketinške budžete, optimizuju prodajne kanale i donesu bolje finansijske odluke. Umesto oslanjanja na subjektivne procene, koriste se podaci i algoritmi mašinskog učenja.

Iz ugla nauke o podacima, ovo je problem **regresije** – ciljna promenljiva je kontinuirana vrednost prihoda. Zbog izražene desne asimetrije ciljne promenljive primenjena je logaritamska transformacija:

```r
log_revenue = log(sales_revenue_usd)
```

Transformacija je korišćena radi smanjenja desne asimetrije, ublažavanja uticaja ekstremnih vrednosti i dobijanja pogodnije distribucije za regresiono modelovanje.


## Podaci

Korišćen je skup **Marketing & Sales** sa platforme Kaggle.

- **60.000 instanci** i **23 primarne kolone**.
- Podaci obuhvataju poslovanje na MENA tržištu.
- Podaci obuhvataju period od **2020. do 2023. godine**.
- **Ciljna promenljiva:** `sales_revenue_usd`, izražena u USD i sa izraženom desnom asimetrijom.

### Pregled originalnih promenljivih

| Naziv kolone | Opis promenljive |
| :--- | :--- |
| `id` | Jedinstveni identifikacioni broj transakcije. |
| `date` | Datum izvršene transakcije. |
| `region` | Geografski region u okviru MENA tržišta. |
| `sales_channel` | Kanal prodaje. |
| `product_category` | Kategorija proizvoda. |
| `customer_segment` | Segment kupca. |
| `season` | Kvartal / sezona u godini (Q1, Q2, Q3, Q4). |
| `marketing_budget_usd` | Ukupan opredeljeni marketinški budžet u dolarima. |
| `ad_spend_online_usd` | Sredstva uložena u online oglašavanje. |
| `ad_spend_offline_usd` | Sredstva uložena u offline marketinške kampanje. |
| `num_promotions` | Broj aktivnih promotivnih ponuda. |
| `discount_percentage` | Procenat odobrenog popusta. |
| `num_sales_representatives` | Broj angažovanih prodajnih predstavnika. |
| `customer_age` | Starost kupca u godinama. |
| `customer_satisfaction_score` | Ocena zadovoljstva kupca. |
| `competitor_price_index` | Indeks cene konkurencije u odnosu na naš proizvod. |
| `website_traffic` | Saobraćaj na veb-sajtu. |
| `conversion_rate` | Stopa konverzije posetilaca u kupce. |
| `email_open_rate` | Procenat otvaranja promotivnih mejlova. |
| `social_media_followers` | Broj pratilaca brenda na društvenim mrežama. |
| `days_since_last_purchase` | Broj dana od poslednje kupovine kupca. |
| `num_previous_purchases` | Ukupan broj prethodnih kupovina kupca. |
| `sales_revenue_usd` | Ostvareni prihod od prodaje u dolarima (**ciljna promenljiva**). |


## Tok analize

### 1. Detekcija i imputacija nedostajućih vrednosti

- Nedostajuće vrednosti identifikovane su u 4 kolone: `email_open_rate`, `discount_percentage`, `customer_satisfaction_score` i `days_since_last_purchase`.
- Udeo nedostajućih vrednosti iznosi približno **3% po promenljivoj**.
- Sprovedeno je testiranje mehanizma nedostajanja pomoću **Chi-Square testova** i MCAR analiza.
- Za imputaciju je korišćen **MICE algoritam sa PMM metodom** (`m = 5`, `maxit = 5`, `seed = 123`).
- Nakon imputacije nije ostalo nedostajućih vrednosti u skupu podataka.

### 2. Feature Engineering

Kreirana su izvedena obeležja u finansijskim, vremenskim, digitalnim i drugim kategorijama, uključujući:

- `total_ad_spend`
- `online_share`
- `log_budget`
- `spend_per_rep`
- `spend_per_promotion`
- `budget_per_traffic`
- vremenske karakteristike kao što su `month`, `quarter`, `day_of_week` i `is_weekend`

Programski je provereno da se ciljna promenljiva ne koristi na način koji bi doveo do **target leakage-a**.

### 3. Eksplorativna analiza podataka (EDA) i autlajeri

- Ciljna promenljiva je log-transformisana u `log_revenue`.
- Log-transformacija je smanjila izraženu desnu asimetriju ciljne promenljive i omogućila pogodniju osnovu za regresiono modelovanje.
- Autlajeri su analizirani pomoću IQR metode i grafičkih prikaza.
- Utvrđeno je da identifikovane ekstremne vrednosti predstavljaju realne poslovne slučajeve, zbog čega su zadržane u skupu podataka.
- U vremenskoj analizi uočen je izraženiji **Q4 efekat**, odnosno veći prihod tokom četvrtog kvartala.

### 4. Selekcija obeležja i redukcija dimenzionalnosti

- **Multikolinearnost (GVIF):** analizirani su odnosi između prediktora i uklonjeni redundantni atributi.
- **Statističko rangiranje:** korišćena je Spearman-ova korelacija za numeričke i Eta-Squared ($\eta^2$) analiza za kategorijske atribute.
- `id`, `date` i originalna ciljna promenljiva `sales_revenue_usd` nisu korišćeni kao prediktori u konačnim modelima.
- Redukovani skup prediktora omogućio je smanjenje složenosti modela uz zadržavanje visoke prediktivne moći.

**Konačno izabrani prediktori:**

```text
log_budget
total_ad_spend
spend_per_rep
num_promotions
discount_percentage
conversion_rate
customer_satisfaction_score
num_previous_purchases
product_category
customer_segment
season
sales_channel
```


## Rezultati modelovanja

Podaci su podeljeni na **Train (80%)** i **Test (20%)** skup uz primenu **5-fold unakrsne validacije** nad trening skupom. Test skup od 12.000 instanci korišćen je isključivo za finalnu evaluaciju.

Metrike evaluacije na **test skupu**, na logaritamskoj skali:

| Model | $R^2$ | MAE | RMSE |
| :--- | :---: | :---: | :---: |
| **GBM / XGBoost** | **0.9297** | **0.1458** | **0.1842** |
| **Random Forest** | 0.9039 | 0.1715 | 0.2153 |
| **Linear Regression** | 0.9014 | 0.1710 | 0.2181 |
| **Lasso Regression** | 0.9014 | 0.1711 | 0.2181 |
| **Ridge Regression** | 0.8933 | 0.1788 | 0.2269 |
| **Decision Tree (CART)** | 0.8714 | 0.1997 | 0.2491 |
| **Baseline** (srednja vrednost) | -0.0002 | 0.5540 | 0.6946 |

### Ključni uvidi

1. **Najbolji model:** **Gradient Boosting (GBM/XGBoost)** ostvario je najbolje rezultate sa $R^2 = 0.9297$, MAE = 0.1458 i RMSE = 0.1842.

2. **Poređenje sa linearnim modelom:** u odnosu na linearnu regresiju, GBM je smanjio RMSE sa 0.2181 na 0.1842, što predstavlja smanjenje greške od približno **15.5%**.

3. **Snažan linearni signal:** Linearna regresija i Lasso ostvarili su $R^2 = 0.9014$, dok je Random Forest ostvario $R^2 = 0.9039$.

4. **Važnost varijabli:** analiza značajnosti i permutacione važnosti pokazala je da među najvažnijim prediktorima dominiraju **`customer_segment`**, **`product_category`**, **`log_budget`** i **`total_ad_spend`**.



## Zaključak

Istraživanje je pokazalo da je prihod od prodaje snažno povezan sa karakteristikama kupaca, kategorijom proizvoda i obimom marketinških ulaganja.

Primena kompletnog procesa pripreme podataka, imputacije nedostajućih vrednosti, inženjeringa obeležja, selekcije prediktora i poređenja više regresionih modela omogućila je izgradnju pouzdanih modela za predviđanje prihoda.

Najbolje performanse ostvario je **Gradient Boosting (GBM/XGBoost)** sa $R^2 = 0.9297$ i RMSE = 0.1842, čime je nadmašio ostale testirane modele.

Sa druge strane, **Linearna regresija i Lasso** ostvarile su $R^2 \approx 0.9014$ i predstavljaju dobre alternative kada su jednostavnost i interpretabilnost modela važnije od maksimalne prediktivne preciznosti.


## Pokretanje projekta

### Potrebni paketi

```r
install.packages(c(
  "mice", "tidyverse", "knitr", "lmtest", "car", "randomForest",
  "glmnet", "caret", "rpart", "rpart.plot", "vip"
))
```

### Koraci

1. Klonirajte repozitorijum:
   ```bash
   git clone https://github.com/lukaJevtic1/marketing-sales-machine-learning.git
   ```
2. Otvorite `project/project.Rproj` u RStudiju. Radni direktorijum se tada automatski postavlja na folder `project`.
   *(Ako ne koristite RStudio projekat, postavite ga ručno: `setwd("putanja/do/marketing-sales-machine-learning/project")`.)*
3. Otvorite `project.Rmd` i pokrenite ga (**Knit**), ili izvršavajte blokove koda redom. Skup podataka (`marketing_sales_dataset.csv`) učitava se iz istog foldera.

> Rezultati su ponovljivi jer su korišćeni fiksni seed-ovi (`123`, `42`).

## Struktura repozitorijuma

```
├── project/
│   ├── project.Rmd                   
│   ├── project.Rproj                 
│   ├── project.nb.html                
│   ├── project.nb.pdf                 
│   └── marketing_sales_dataset.csv    
├── .gitignore
└── README.md
```
