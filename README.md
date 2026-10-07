# Predviđanje prihoda od prodaje – Marketing & Sales

Seminarski rad iz mašinskog učenja u programskom jeziku **R**. Cilj projekta je sveobuhvatna analiza faktora koji utiču na ostvareni prihod od prodaje (`sales_revenue_usd`) i razvoj regresionih modela koji predviđaju prihod na osnovu marketinških ulaganja, kanala prodaje i ponašanja kupaca.

**Autori:** Danilo Novaković (101/2018), Luka Jevtić (64/2017)  
**Fakultet:** Prirodno-matematički fakultet, Univerzitet u Kragujevcu  


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

Iz ugla nauke o podacima, ovo je problem **regresije** – ciljna promenljiva je kontinuirana vrednost prihoda, pri čemu je u cilju stabilizacije varijanse i postizanja normalnosti primenjena logaritmatska transformacija (`log_revenue = log(sales_revenue_usd)`).



## Podaci

Korišćen je skup **Marketing & Sales** sa platforme Kaggle:  
[kaggle.com/datasets/abdelfattahibrahim/marketing-sales-dataset](https://www.kaggle.com/datasets/abdelfattahibrahim/marketing-sales-dataset)

- **60.000 instanci** i **23 primarne kolone** iz poslovanja na MENA tržištu (Cairo, Riyadh, Dubai, Abu Dhabi, Jeddah, Amman, Alexandria).
- Obuhvata trogodišnji period od 2020. do 2023. godine (stabilnih ~15.000 transakcija godišnje).
- **Ciljna promenljiva:** `sales_revenue_usd` (izražena u USD; izrazito desno asimetrična sa skewness ≈ 6.63).

### Pregled originalnih promenljivih

| Naziv kolone | Opis promenljive |
| :--- | :--- |
| `id` | Jedinstveni identifikacioni broj transakcije. |
| `date` | Datum izvršene transakcije. |
| `region` | Geografski region u okviru MENA tržišta. |
| `sales_channel` | Kanal prodaje (Online, Retail Store, Direct Sales, itd.). |
| `product_category` | Kategorija proizvoda (Electronics, Cosmetics, Clothing, Food & Beverage, Home & Kitchen). |
| `customer_segment` | Segment kupca (Regular, New, Corporate, VIP). |
| `season` | Kvartal / sezona u godini (Q1, Q2, Q3, Q4). |
| `marketing_budget_usd` | Ukupan opredeljeni marketinški budžet u dolarima. |
| `ad_spend_online_usd` | Sredstva uložena u online oglašavanje. |
| `ad_spend_offline_usd` | Sredstva uložena u offline marketinške kampanje. |
| `num_promotions` | Broj aktivnih promotivnih ponuda tokom transakcije. |
| `discount_percentage` | Procenat odobrenog popusta. |
| `num_sales_representatives` | Broj angažovanih prodajnih predstavnika. |
| `customer_age` | Starost kupca u godinama. |
| `customer_satisfaction_score` | Ocena zadovoljstva kupca (skala 1–5). |
| `competitor_price_index` | Indeks cene konkurencije u odnosu na naš proizvod. |
| `website_traffic` | Saobraćaj na veb-sajtu (broj poseta). |
| `conversion_rate` | Stopa konverzije posetilaca u kupce. |
| `email_open_rate` | Procenat otvaranja promotivnih mejlova. |
| `social_media_followers` | Broj pratilaca brenda na društvenim mrežama. |
| `days_since_last_purchase` | Broj dana od poslednje kupovine kupca. |
| `num_previous_purchases` | Ukupan broj prethodnih kupovina kupca. |
| `sales_revenue_usd` | Ostvareni prihod od prodaje u dolarima (**ciljna promenljiva**). |



## Tok analize

### 1. Detekcija i imputacija nedostajućih vrednosti
- Nedostajuće vrednosti identifikovane su u 4 kolone: `email_open_rate`, `discount_percentage`, `customer_satisfaction_score` i `days_since_last_purchase` (svaka sa ~3% missing-a).
- Sprovedeno je testiranje mehanizma nedostajanja (Chi-Square testovi, MCAR analiza po Hawkins/Anderson-Darling metodologiji).
- Imputacija je izvršena korišćenjem **MICE algoritma sa PMM metodom** (`m = 5`, `maxit = 5`, `seed = 123`).

### 2. Feature Engineering
Kreirana su izvedena obeležja u finansijskim, vremenskim, digitalnim i interakcionim kategorijama (npr. `total_ad_spend`, `online_share`, `log_budget`, `spend_per_rep`, `price_advantage`, `customer_age_group`, `is_weekend`, `season_product`).  
*Programski je verifikovano da nema curenja podataka (target leakage) prema ciljnoj promenljivoj.*

### 3. Eksplorativna analiza podataka (EDA) i autlajeri
- Ciljna promenljiva je log-transformisana u `log_revenue`, čime je postignuta kontinuirana raspodela bliska normalnoj (potvrđeno Q-Q plotom).
- Identifikovani autlajeri po IQR metodi (~6.36%) zadržani su u skupu jer predstavljaju realne visoke transakcije i uspešne kampanje.
- Otkriven je izraženi **Q4 efekat** u vremenskoj analizi (znatno viši prihod krajem godine).

### 4. Selekcija obeležja i redukcija dimenzionalnosti
- **Multikolinearnost (GVIF):** Uklonjeni su kolinearni ad-spend atributi.
- **Statističko rangiranje:** Spearman-ova korelacija za numeričke i Eta-Squared ($\eta^2$) iz ANOVA modela za kategorijske atribute.
- **Poređenje modela bez curenja podataka:** Izbačeni su ID, datum i izvorni `sales_revenue_usd` iz punog modela. Redukovani model sa **12 ključnih ulaznih atributa** zadržao je visoku objašnjivost uz drastično manju složenost.
- **Konačno izabrani prediktori (12):** `log_budget`, `total_ad_spend`, `spend_per_rep`, `num_promotions`, `discount_percentage`, `conversion_rate`, `customer_satisfaction_score`, `num_previous_purchases`, `product_category`, `customer_segment`, `season`, `sales_channel`.



## Rezultati modelovanja

Podaci su podeljeni na **Train (80%)** i **Test (20%)** skup uz primenu **5-fold unakrsne validacije** nad trening setom. Test skup od 12.000 instanci korišćen je isključivo za finalnu evaluaciju.

Metrike evaluacije na **Test skupu** (na logaritamskoj skali):

| Model | $R^2$ | MAE | RMSE |
| :--- | :---: | :---: | :---: |
| **GBM / XGBoost** | **0.9297** | **0.1458** | **0.1842** |
| **Random Forest** | 0.9039 | 0.1715 | 0.2153 |
| **Linear Regression** | 0.9014 | 0.1710 | 0.2181 |
| **Lasso Regression** | 0.9014 | 0.1711 | 0.2181 |
| **Ridge Regression** | 0.8933 | 0.1788 | 0.2269 |
| **Decision Tree (CART)** | 0.8714 | 0.1997 | 0.2491 |
| **Baseline** (Srednja vrednost) | -0.0002 | 0.5540 | 0.6946 |

### Ključni uvidi:
1. **Apsolutni pobednik:** **Gradient Boosting (GBM/XGBoost)** ubedljivo zauzima prvo mesto sa $R^2 = 0.9297$ i najnižim greškama ($\text{MAE} = 0.1458$, $\text{RMSE} = 0.1842$), čime je smanjio grešku predikcije za preko 14% u odnosu na linearni benchmark.
2. **Snažan linearni signal:** Linearna regresija i Lasso postavljaju odličan baseline sa $R^2 = 0.9014$, dok Random Forest ostvaruje $R^2 = 0.9039$.
3. **Važnost varijabli:** Analiza permutacione važnosti i značajnosti atributa pokazala je da su ubedljivo najjači pokretači prihoda **`customer_segment`** (naročito Novi i VIP kupci), **`product_category`**, budžet (`log_budget`), kao i stopa konverzije (`conversion_rate`).

## Zaključak

Istraživanje je pokazalo da je prihod u poslovanju najsnažnije određen kategorijom kupaca, tipom proizvoda, budžetom i stopom konverzije. Napredne metode, prvenstveno Gradient Boosting (GBM), ostvarile su vrhunske performanse sa preko 92.9% objašnjene varijanse na neviđenom test skupu. S druge strane, Linearna i Lasso regresija ($R^2 \approx 0.9014$) nude odličnu alternativu za produkciju kada je primarna jednostavnost i interpretabilnost.

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
