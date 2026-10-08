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

Iz ugla nauke o podacima, ovo je problem **regresije** – ciljna promenljiva je kontinuirana vrednost prihoda. Zbog izražene desne asimetrije ciljne promenljive primenjena je logaritamska transformacija:

```r
log_revenue = log(sales_revenue_usd)
```

Transformacija je korišćena radi smanjenja desne asimetrije, ublažavanja uticaja ekstremnih vrednosti i dobijanja pogodnije distribucije za regresiono modelovanje.


## Podaci

Korišćen je skup **Marketing & Sales** sa platforme Kaggle: [kaggle.com/datasets/abdelfattahibrahim/marketing-sales-dataset](https://www.kaggle.com/datasets/abdelfattahibrahim/marketing-sales-dataset)

- **60.000 instanci** i **23 primarne kolone**.
- Podaci obuhvataju poslovanje na MENA tržištu (Riyadh, Dubai, Cairo, Alexandria, Amman, Casablanca, Kuwait…).
- Period: **2020–2023**, oko 15.000 transakcija godišnje.
- **Ciljna promenljiva:** `sales_revenue_usd` (USD), prosek ≈ 5.911, medijana ≈ 4.340, maksimum ≈ 190.377, sa izraženom desnom asimetrijom.
- Skup nema duplikata, a opsezi vrednosti su logički ispravni (starost 18–74, ocena zadovoljstva 2–5, popust do 40%).

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

- Nedostajuće vrednosti postoje u 4 kolone: `email_open_rate` (1.790), `discount_percentage` (1.808), `customer_satisfaction_score` (1.844) i `days_since_last_purchase` (1.836), odnosno oko **3% po promenljivoj**.
- Mehanizam nedostajanja ispitan je vizuelizacijom obrazaca, indikatorima nedostajanja, **Chi-Square testovima** i MCAR analizom.
- Za imputaciju je korišćen **MICE algoritam sa PMM metodom** (`m = 5`, `maxit = 5`, `seed = 123`). Brisanje redova i imputacija srednjom vrednošću ili medijanom odbačeni su zbog gubitka podataka, odnosno veštačkog smanjenja varijanse.
- Nakon imputacije nije ostalo nedostajućih vrednosti.

### 2. Feature Engineering

Kreirana su izvedena obeležja u finansijskim, vremenskim, digitalnim i drugim kategorijama, uključujući:

- **Finansijska i marketinška:** `total_ad_spend`, `online_share`, `budget_utilization`, `unspent_budget`, `spend_per_rep`, `spend_per_promotion`, `budget_per_traffic`
- **Logaritamske transformacije:** `log_budget`, `log_traffic`, `log_followers`
- **Vremenska:** `month`, `quarter`, `day_of_week`, `is_weekend`, `is_year_end`, `days_since_start`
- **Kupci:** `purchase_frequency`, `is_returning`, `customer_age_group`
- **Tržišna i digitalna:** `price_advantage`, `effective_discount`, `estimated_conversions`, `traffic_per_follower`
- **Interakcije:** `segment_product`, `season_product`, `channel_segment`

Programski je provereno da nijedno novo obeležje ne koristi ciljnu promenljivu, a korelacije novih obeležja sa ciljem ne ukazuju na indirektno curenje podataka (**target leakage**).

### 3. Eksplorativna analiza podataka (EDA) i autlajeri

- Ciljna promenljiva je jako asimetrična (**skewness ≈ 6.63**), pa je log-transformisana u `log_revenue`. Nakon transformacije raspodela je približno normalna (potvrđeno Q-Q plotom).
- Autlajeri su analizirani IQR metodom i grafički: **3.816 opservacija (≈ 6.36%)**. Zadržani su jer predstavljaju realne uspešne kampanje, a ne greške u podacima.
- Urađene su univarijatna, korelaciona i kategorijska analiza.
- U vremenskoj analizi prihod je stabilan od 2020. do 2023. (oko 86,6–89,9 miliona USD godišnje), uz izražen **Q4 efekat**: oktobar–decembar donose preko 36 miliona USD mesečno, dok je u prvih šest meseci prihod 22–26 miliona.

### 4. Selekcija obeležja i redukcija dimenzionalnosti

1. **Multikolinearnost (GVIF):** analizirani su odnosi između prediktora i uklonjeni redundantni atributi (`marketing_budget_usd`, `ad_spend_online_usd`, `ad_spend_offline_usd`).
2. **Statističko rangiranje:** Spearman-ova korelacija za numerička i Eta-Squared ($\eta^2$) za kategorijska obeležja. Uklonjeni su slabi prediktori (npr. `region`).
3. **Model-based potvrda:** Random Forest importance i Lasso regularizacija.
4. `id`, `date` i originalna ciljna promenljiva `sales_revenue_usd` nisu korišćeni kao prediktori u konačnim modelima.

Poređenje linearnog modela sa svim obeležjima (46 prediktora, $R^2_{adj}$ = 0.943) i sa izabranim skupom (12 prediktora, $R^2_{adj}$ = 0.902) pokazuje da redukcija znatno smanjuje složenost uz mali gubitak objašnjene varijanse.

**Konačno izabrani prediktori (12):**

```text
log_budget
total_ad_spend
spend_per_rep
spend_per_promotion
num_promotions
customer_satisfaction_score
conversion_rate
num_previous_purchases
product_category
customer_segment
season
sales_channel
```


## Rezultati modelovanja

Podaci su podeljeni na **Train (80%, 48.000 instanci)** i **Test (20%, 12.000 instanci)** skup (`set.seed(123)`), uz **5-fold unakrsnu validaciju** nad trening skupom. Test skup je korišćen za finalnu evaluaciju modela. Modeli predviđaju `log_revenue`, pa su sve metrike izražene na **logaritamskoj skali**.

Hiperparametri su podešavani na malim gridovima: Random Forest (`mtry` ∈ {2, 4}, `ntree = 150`) i Gradient Boosting (`interaction.depth` ∈ {3, 5}, `n.trees = 150`, `shrinkage = 0.1`, `n.minobsinnode = 10`).

Metrike na **test skupu**:

| Model | $R^2$ | MAE | RMSE |
| :--- | :---: | :---: | :---: |
| **Gradient Boosting (GBM)** | **0.9297** | **0.1458** | **0.1842** |
| Random Forest | 0.9039 | 0.1715 | 0.2153 |
| Linear Regression | 0.9014 | 0.1710 | 0.2181 |
| Lasso Regression | 0.9014 | 0.1711 | 0.2181 |
| Ridge Regression | 0.8933 | 0.1788 | 0.2269 |
| Decision Tree (CART) | 0.8714 | 0.1997 | 0.2491 |
| Baseline (srednja vrednost) | -0.0002 | 0.5540 | 0.6946 |

### Ključni uvidi

1. **Najbolji model:** Gradient Boosting ostvario je $R^2 = 0.9297$, MAE = 0.1458 i RMSE = 0.1842. Njegova prosečna greška (bias) na test skupu je zanemarljiva (≈ -0.0006), a najveća apsolutna greška iznosi 0.7485.

2. **Poređenje sa ostalim modelima:** u odnosu na linearnu regresiju GBM smanjuje RMSE sa 0.2181 na 0.1842 (≈ **15.5%**), a u odnosu na Random Forest sa 0.2153 (≈ **14.4%**).

3. **Linearni signal je jak, ali ne i jedini:** linearni modeli već objašnjavaju oko 90% varijanse, dok GBM dodaje još oko 2.8 procentnih poena, što ukazuje na nelinearne efekte i interakcije. Random Forest je tek neznatno bolji od linearne regresije, što može biti posledica ograničenog podešavanja hiperparametara.

4. **Važnost varijabli:** Random Forest importance i permutaciona važnost slažu se da dominiraju **`customer_segment`**, **`product_category`**, **`log_budget`** i **`total_ad_spend`**. Permutacija `customer_segment` povećava RMSE za više od 0.35.


## Zaključak

Istraživanje je pokazalo da je prihod od prodaje snažno povezan sa profilom kupaca (segment, istorija kupovina), kategorijom proizvoda i obimom marketinških ulaganja, uz izražen sezonski efekat u četvrtom kvartalu.

Primena kompletnog procesa pripreme podataka, imputacije nedostajućih vrednosti, inženjeringa obeležja, selekcije prediktora i poređenja više regresionih modela omogućila je izgradnju pouzdanih modela za predviđanje prihoda. Najbolje performanse ostvario je **Gradient Boosting** sa $R^2 = 0.9297$ i RMSE = 0.1842 (na logaritamskoj skali, što otprilike odgovara relativnoj grešci od 18–20% u originalnim jedinicama).

**Linearna regresija i Lasso** ($R^2 \approx 0.9014$) ostaju dobre alternative kada su jednostavnost i interpretabilnost važniji od maksimalne prediktivne preciznosti.


## Pokretanje projekta

### Potrebni paketi

```r
install.packages(c(
  "mice", "tidyverse", "knitr", "car", "randomForest",
  "glmnet", "caret", "rpart", "rpart.plot", "vip", "gbm"
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
