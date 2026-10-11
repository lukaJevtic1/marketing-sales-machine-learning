# Predviđanje prihoda od prodaje – Marketing & Sales

Seminarski rad iz mašinskog učenja u programskom jeziku **R**. Cilj projekta je analiza faktora koji utiču na ostvareni prihod od prodaje (`sales_revenue_usd`) i razvoj regresionih modela koji predviđaju prihod na osnovu marketinških ulaganja, kanala prodaje i ponašanja kupaca.

**Autori:** Danilo Novaković (101/2018), Luka Jevtić (64/2017)

**Fakultet:** Prirodno-matematički fakultet, Univerzitet u Kragujevcu


## Sadržaj

1. [Motivacija](#motivacija)
2. [Podaci](#podaci)
3. [Tok analize](#tok-analize)
4. [Rezultati modelovanja](#rezultati-modelovanja)
5. [Ograničenja](#ograničenja)
6. [Zaključak](#zaključak)
7. [Pokretanje projekta](#pokretanje-projekta)
8. [Struktura repozitorijuma](#struktura-repozitorijuma)


## Motivacija

Analiza i predviđanje prihoda od prodaje važni su za uspešno poslovanje: omogućavaju kompanijama da pametnije rasporede marketinške budžete, optimizuju prodajne kanale i donesu bolje finansijske odluke. Umesto oslanjanja na subjektivne procene, koriste se podaci i algoritmi mašinskog učenja.

Iz ugla nauke o podacima, ovo je problem **regresije** – ciljna promenljiva je kontinuirana vrednost prihoda. Zbog izražene desne asimetrije (skewness ≈ 6.63) primenjena je logaritamska transformacija:

```r
log_revenue = log(sales_revenue_usd)
```

Transformacija smanjuje asimetriju, ublažava uticaj ekstremnih vrednosti i daje pogodniju raspodelu za regresiono modelovanje.


## Podaci

Korišćen je skup **Marketing & Sales** sa platforme Kaggle: [kaggle.com/datasets/abdelfattahibrahim/marketing-sales-dataset](https://www.kaggle.com/datasets/abdelfattahibrahim/marketing-sales-dataset)

- **60.000 instanci** i **23 primarne kolone** (7 celobrojnih, 10 numeričkih i 6 karakternih).
- Podaci obuhvataju poslovanje na MENA tržištu (Alexandria, Amman, Cairo, Casablanca, Dubai, Kuwait, Riyadh).
- Period: **2020–2023**, oko 15.000 transakcija godišnje.
- **Ciljna promenljiva:** `sales_revenue_usd` (USD), prosek ≈ 5.911, medijana ≈ 4.340, maksimum ≈ 190.377, sa izraženom desnom asimetrijom.
- Nema potpuno dupliranih redova, a opsezi vrednosti su logički ispravni (starost 18–74, ocena zadovoljstva 2–5, popust do 40%).
- Skup deluje **sintetički** (veoma pravilni sezonski obrasci kroz godine, glatke raspodele), pa se zaključci ne bi smeli direktno prenositi na realno poslovanje.

### Pregled originalnih promenljivih

| Naziv kolone | Opis promenljive |
| :--- | :--- |
| `id` | Jedinstveni identifikacioni broj transakcije. |
| `date` | Datum izvršene transakcije. |
| `region` | Geografski region u okviru MENA tržišta. |
| `sales_channel` | Kanal prodaje (Online, Retail Store, Direct Sales, Social Media, Wholesale). |
| `product_category` | Kategorija proizvoda. |
| `customer_segment` | Segment kupca (Regular, New, Corporate, VIP). |
| `season` | Kvartal u godini (Q1, Q2, Q3, Q4). |
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

### 1. Nedostajuće vrednosti i imputacija

- Nedostajuće vrednosti postoje u 4 kolone: `email_open_rate` (1.790), `discount_percentage` (1.808), `customer_satisfaction_score` (1.844) i `days_since_last_purchase` (1.836), odnosno oko **3% po promenljivoj**. Samo 0.52% redova ima dve ili više nedostajućih vrednosti.
- Mehanizam nedostajanja ispitan je vizuelizacijom obrazaca, indikatorima nedostajanja, **χ² testovima** i MCAR testom. Hawkins test odbacuje MCAR, dok Anderson-Darling rank test ne, pa je MCAR korišćen samo kao **radna pretpostavka**. Izabrana metoda je opravdana i u slučaju šireg MAR mehanizma.
- Za imputaciju je korišćen **MICE algoritam sa PMM metodom** (`m = 5`, `maxit = 5`, `seed = 123`); za dalji rad uzet je prvi imputirani skup. Brisanje redova i imputacija srednjom vrednošću ili medijanom odbačeni su zbog gubitka podataka, odnosno veštačkog smanjenja varijanse.
- Nakon imputacije nije ostalo nedostajućih vrednosti.

### 2. Feature Engineering

Kreirana su izvedena obeležja:

- **Finansijska i marketinška:** `total_ad_spend`, `online_share`, `budget_utilization`, `unspent_budget`, `spend_per_rep`, `spend_per_promotion`, `budget_per_traffic`
- **Logaritamske transformacije:** `log_budget`, `log_traffic`, `log_followers`
- **Vremenska:** `month`, `quarter`, `day_of_week`, `is_weekend`, `is_year_end`, `days_since_start`
- **Kupci:** `purchase_frequency`, `is_returning`, `customer_age_group`
- **Tržišna i digitalna:** `price_advantage`, `effective_discount`, `estimated_conversions`, `traffic_per_follower`
- **Interakcije:** `segment_product`, `season_product`, `channel_segment`

Nijedno novo obeležje se ne računa iz ciljne promenljive, a korelacije novih obeležja sa ciljem ne ukazuju na očigledno curenje podataka (**target leakage**). Treba imati u vidu da nova finansijska obeležja pojačavaju međusobnu povezanost (Spearman 0.91–0.99), pa je multikolinearnost rešavana naknadno, u fazi selekcije.

### 3. Eksplorativna analiza podataka (EDA) i outlier-i

- Ciljna promenljiva je jako asimetrična (**skewness ≈ 6.63**), pa je log-transformisana u `log_revenue`. Nakon transformacije raspodela je približno normalna (Q-Q plot).
- Outlier-i su analizirani IQR metodom (gornja granica 13.612,81 USD). Identifikovano je **3.816 opservacija (≈ 6.36%)** u gornjem repu. Zadržani su jer ne deluju kao greške u merenju, a njihovo uklanjanje bi osiromašilo skup.
- Urađene su univarijatna, korelaciona (Spearman) i kategorijska analiza. Prihod je najviše povezan sa budžetskim promenljivama (Spearman oko 0.5), dok su saobraćaj na sajtu, pratioci i popust gotovo nepovezani sa prihodom.
- U vremenskoj analizi prihod je stabilan od 2020. do 2023. (oko 86,6–89,9 miliona USD godišnje), uz izražen **Q4 efekat**: oktobar–decembar donose preko 36 miliona USD mesečno, dok se u prvih šest meseci prihod kreće oko 22–28 miliona.

### 4. Feature Selection i redukcija dimenzionalnosti

1. **Multikolinearnost (GVIF):** redundantni atributi (`marketing_budget_usd`, `ad_spend_online_usd`, `ad_spend_offline_usd`) isključeni su iz modela za proveru, a nad ostalim prediktorima GVIF je ispod praga 5.
2. **Statističko rangiranje:** Spearman-ova korelacija za numerička i eta-kvadrat (η²) za kategorijska obeležja; slabi prediktori (npr. `region`, `discount_percentage`) su uklonjeni.
3. **Model-based potvrda:** Random Forest importance i Lasso regularizacija (u ovom koraku nijedno obeležje nije dodatno izbačeno).
4. `id`, `date` i originalna ciljna promenljiva `sales_revenue_usd` nisu korišćeni kao prediktori.

Linearni model sa svim obeležjima (46 prediktora, R²_adj = 0.943) i model sa izabranim skupom (13 prediktora, R²_adj = 0.902) razlikuju se za oko 4 procentna poena objašnjene varijanse, što je cena redukcije složenosti.

**Konačno izabrani prediktori (13: 9 numeričkih i 4 kategorijska):**

```text
log_budget
total_ad_spend
spend_per_rep
spend_per_promotion
num_promotions
customer_satisfaction_score
conversion_rate
num_previous_purchases
purchase_frequency
product_category
customer_segment
season
sales_channel
```


## Rezultati modelovanja

Podaci su podeljeni na **Train (80%, 48.000 instanci)** i **Test (20%, 12.000 instanci)** skup (`set.seed(123)`), uz **5-fold unakrsnu validaciju** nad trening skupom. Skaliranje (z-score) i nivoi faktora računati su samo na trening skupu. Modeli predviđaju `log_revenue`, pa su sve metrike izražene na **logaritamskoj skali**.

Metrike na **test skupu**:

| Model | R² | MAE | RMSE |
| :--- | :---: | :---: | :---: |
| **Gradient Boosting (GBM)** | **0.9302** | **0.1453** | **0.1835** |
| Random Forest | 0.9016 | 0.1737 | 0.2179 |
| Linear Regression | 0.9014 | 0.1709 | 0.2181 |
| Lasso Regression | 0.9014 | 0.1710 | 0.2181 |
| Ridge Regression | 0.8933 | 0.1788 | 0.2269 |
| Decision Tree (CART) | 0.8714 | 0.1996 | 0.2491 |
| Baseline (srednja vrednost) | -0.0002 | 0.5540 | 0.6946 |

### Ključni uvidi

1. **Najbolji model:** Gradient Boosting ostvario je R² = 0.9302, MAE = 0.1453 i RMSE = 0.1835. Prosečna greška (bias) na test skupu je zanemarljiva (≈ -0.0013), a najveća apsolutna greška iznosi 0.7105.

2. **Poređenje sa ostalim modelima:** u odnosu na linearnu regresiju GBM smanjuje RMSE sa 0.2181 na 0.1835 (≈ **15.9%**), a u odnosu na Random Forest sa 0.2179 (≈ **15.8%**).

3. **Linearni signal je jak, ali ne i jedini:** linearni modeli objašnjavaju oko 90% varijanse, dok GBM dodaje još oko 2.9 procentnih poena, što ukazuje na nelinearne efekte i interakcije. Random Forest je tek neznatno bolji od linearne regresije po R² i RMSE (a po MAE nije), što može biti posledica skromnog podešavanja hiperparametara.

4. **Važnost varijabli:** Random Forest importance i permutaciona važnost (obe računate nad Random Forest modelom, a ne nad GBM-om) slažu se da dominiraju **`customer_segment`**, **`product_category`**, **`log_budget`** i **`total_ad_spend`**. Permutacija `customer_segment` povećava RMSE za više od 0.35. Ovo su statističke povezanosti, a ne dokaz uzročnosti.


## Zaključak

Prihod od prodaje je u ovom skupu najjače povezan sa segmentom kupaca, kategorijom proizvoda i obimom marketinških ulaganja, uz izražen sezonski efekat u četvrtom kvartalu.

Kompletan proces (imputacija, inženjering obeležja, selekcija prediktora i poređenje više regresionih modela) doveo je do modela koji dobro predviđaju prihod na ovom skupu. Najbolje performanse ostvario je **Gradient Boosting** sa R² = 0.9302 i RMSE = 0.1835 (na logaritamskoj skali, što otprilike odgovara relativnoj grešci od 18–20% u originalnim jedinicama).

**Linearna regresija i Lasso** (R² ≈ 0.9014) ostaju dobre alternative kada su jednostavnost i interpretabilnost važniji od maksimalne preciznosti.


## Pokretanje projekta

### Potrebni paketi

```r
install.packages(c(
  "mice", "tidyverse", "knitr", "car", "randomForest",
  "glmnet", "caret", "rpart", "rpart.plot", "vip", "gbm", "scales"
))
```

> Za MCAR test koristi se funkcija `mice::mcar`, pa je potrebna novija verzija paketa `mice`.

### Koraci

1. Klonirajte repozitorijum:
```bash
  git clone https://github.com/lukaJevtic1/marketing-sales-machine-learning.git
```
2. Otvorite `project/project.Rproj` u RStudiju. Radni direktorijum se tada automatski postavlja na folder `project`.
   *(Ako ne koristite RStudio projekat, postavite ga ručno: `setwd("putanja/do/marketing-sales-machine-learning/project")`.)*
3. Otvorite `project.Rmd` i pokrenite ga (**Knit**), ili izvršavajte blokove koda redom. Skup podataka (`marketing_sales_dataset.csv`) učitava se iz istog foldera.

> Rezultati su ponovljivi jer su korišćeni fiksni seed-ovi (`123`, `42`). Imputacija i modeli sa Random Forest-om mogu trajati nekoliko minuta.


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
