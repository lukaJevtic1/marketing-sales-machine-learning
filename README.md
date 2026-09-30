# Predviđanje prihoda od prodaje – Marketing & Sales

Seminarski rad iz mašinskog učenja u programskom jeziku **R**. Cilj projekta je analiza faktora koji utiču na ostvareni prihod od prodaje (`sales_revenue_usd`) i razvoj regresionih modela koji predviđaju prihod na osnovu marketinških ulaganja, kanala prodaje i ponašanja kupaca.

**Autori:** Danilo Novaković (101/2018), Luka Jevtić (64/2017)

---

## Sadržaj

1. [Motivacija](#motivacija)
2. [Podaci](#podaci)
3. [Tok analize](#tok-analize)
4. [Rezultati](#rezultati)
5. [Zaključak](#zaključak)
6. [Pokretanje projekta](#pokretanje-projekta)
7. [Struktura repozitorijuma](#struktura-repozitorijuma)

---

## Motivacija

Analiza i predviđanje prihoda od prodaje ključni su za uspešno poslovanje: omogućavaju kompanijama da pametnije rasporede marketinške budžete i donesu bolje finansijske odluke. Umesto oslanjanja na subjektivne procene, koriste se podaci i algoritmi koji na osnovu ulaganja, ponašanja kupaca i kanala prodaje predviđaju buduće rezultate.

Iz ugla nauke o podacima, ovo je problem **regresije** – ciljna promenljiva (`sales_revenue_usd`) je kontinuirana.

## Podaci

Korišćen je skup **Marketing & Sales** sa platforme Kaggle:
[kaggle.com/datasets/abdelfattahibrahim/marketing-sales-dataset](https://www.kaggle.com/datasets/abdelfattahibrahim/marketing-sales-dataset)

- **60.000 instanci** i **23 kolone** iz poslovanja u MENA regionu (Riyadh, Dubai, Cairo, Alexandria, Amman, Casablanca, Kuwait…)
- Obuhvata period od 2020. do 2023. godine (oko 15.000 transakcija godišnje)
- Tipovi kolona: 6 tekstualnih (`date`, `region`, `sales_channel`, `product_category`, `customer_segment`, `season`), ostale su numeričke
- Ciljna promenljiva: `sales_revenue_usd` (prosek ≈ 5.911 USD, medijana ≈ 4.340 USD, maksimum ≈ 190.377 USD)

| Grupa | Kolone |
| --- | --- |
| Identifikacija i vreme | `id`, `date`, `season` |
| Kategorijske | `region`, `sales_channel`, `product_category`, `customer_segment` |
| Marketing | `marketing_budget_usd`, `ad_spend_online_usd`, `ad_spend_offline_usd`, `num_promotions`, `discount_percentage`, `email_open_rate`, `social_media_followers` |
| Prodaja i tržište | `num_sales_representatives`, `competitor_price_index`, `website_traffic`, `conversion_rate` |
| Kupac | `customer_age`, `customer_satisfaction_score`, `days_since_last_purchase`, `num_previous_purchases` |
| **Cilj** | `sales_revenue_usd` |

> Skup nema duplikata i svi opsezi vrednosti su logički ispravni (starost 18–74, ocena zadovoljstva 2–5, popust do 40%).

## Tok analize

### 1. Analiza i imputacija nedostajućih vrednosti
- NA vrednosti postoje u četiri kolone: `email_open_rate` (1790), `discount_percentage` (1808), `customer_satisfaction_score` (1844) i `days_since_last_purchase` (1836).
- Urađena je vizuelizacija obrazaca nedostajućih vrednosti, indikatori missingness-a, test nezavisnosti i MCAR analiza.
- Za imputaciju je izabran **MICE sa PMM** (`m = 5`, `maxit = 5`, `seed = 123`). Brisanje redova i imputacija srednjom/medijanom odbačeni su jer dovode do gubitka podataka, odnosno veštačkog smanjenja varijanse.

### 2. Feature engineering
Kreirana su nova obeležja:

- **Finansijski i marketinški:** `total_ad_spend`, `online_share`, `budget_utilization`, `unspent_budget`, `spend_per_rep`, `spend_per_promotion`, `budget_per_traffic`
- **Logaritamske transformacije:** `log_budget`, `log_traffic`, `log_followers`
- **Vremenska:** `month`, `quarter`, `day_of_week`, `is_weekend`, `is_year_end`, `days_since_start`, `season`
- **Kupci:** `purchase_frequency`, `is_returning`, `customer_age_group`
- **Tržišna i digitalna:** `price_advantage`, `effective_discount`, `estimated_conversions`, `traffic_per_follower`
- **Interakcije:** `segment_product`, `season_product`, `channel_segment`

Programski je provereno da nijedan novi feature ne koristi ciljnu promenljivu, a korelacije novih obeležja sa targetom ne ukazuju na indirektno curenje podataka (**target leakage**).

### 3. Eksplorativna analiza podataka (EDA)
- Univarijatna analiza budžeta, poseta sajtu, broja pratilaca, regiona i kategorija proizvoda.
- Ciljna promenljiva je jako asimetrična (**skewness ≈ 6.63**), pa je primenjena **log-transformacija** (`log_revenue`), nakon koje je raspodela približno normalna (potvrđeno Q-Q plotom).
- Outlieri po IQR metodi: 3.816 opservacija (≈ 6.36%). **Nisu uklonjeni**, jer predstavljaju stvarne uspešne kampanje, a ne greške.
- Korelaciona analiza i kategorijska analiza.
- Vremenska analiza: prihod je stabilan od 2020. do 2023. (86,6–89,9 miliona USD godišnje), uz izražen **Q4 efekat** (oktobar–decembar preko 36 miliona USD mesečno, prvih šest meseci 22–26 miliona).

### 4. Selekcija feature-a
1. **Multikolinearnost (GVIF):** uklonjeni `marketing_budget_usd`, `ad_spend_online_usd` i `ad_spend_offline_usd`.
2. **Statistička selekcija:** Spearman korelacija za numerička i eta-squared za kategorijska obeležja; uklonjeni slabi prediktori (npr. `region`).
3. **Model-based selekcija:** Random Forest importance i Lasso regularizacija.

**Konačni skup od 8 feature-a:**
`num_promotions`, `customer_satisfaction_score`, `conversion_rate`, `num_previous_purchases`, `product_category`, `season`, `customer_segment`, `sales_channel`

### 5. Modelovanje
- Podela **80 : 20** (`set.seed(123)`): 48.000 instanci za trening i 12.000 za test.
- Test skup je potpuno izolovan; parametri predobrade računaju se samo na trening skupu (**bez data leakage-a**).
- **5-fold unakrsna validacija** za sve modele.
- Modeli: Baseline (srednja vrednost), Linearna regresija, Ridge, Lasso, Decision Tree, Random Forest, Gradient Boosting.
- Modeli predviđaju `log_revenue`, pa su metrike izražene na logaritamskoj skali.

## Rezultati

Performanse na test skupu:

| Model | R² | MAE | RMSE |
| --- | --- | --- | --- |
| Baseline (srednja vrednost) | -0.0002 | 0.5540 | 0.6946 |
| **Linearna regresija** | **0.5522** | **0.3456** | **0.4648** |
| Ridge | 0.5470 | 0.3482 | 0.4674 |
| **Lasso** | **0.5522** | **0.3456** | **0.4648** |
| Decision Tree | 0.5078 | 0.3622 | 0.4873 |
| Random Forest | 0.5318 | 0.3548 | 0.4752 |
| Gradient Boosting | 0.5506 | 0.3464 | 0.4656 |

**Ključna zapažanja:**
- Linearna regresija i Lasso daju najbolje i praktično identične rezultate, uz visoku interpretabilnost.
- Gradient Boosting im je vrlo blizu, dok Decision Tree zaostaje.
- Prosečna greška (bias) na test skupu je zanemarljiva (≈ -0.009).
- Najveće greške javljaju se na ekstremnim outlierima sa vrlo visokim prihodom, koje model sa datim prediktorima ne može da predvidi.
- **Najvažniji prediktori** (Random Forest i permutaciona važnost): `customer_segment` i `product_category`, zatim `num_previous_purchases`, `conversion_rate` i `season`.

## Zaključak

Ključni faktori koji utiču na prihod su **segment kupaca, kategorija proizvoda, istorija kupovina, stopa konverzije i sezonski trendovi**. Linearna regresija i Lasso pokazale su se kao optimalan izbor zbog ravnoteže između jednostavnosti i preciznosti.

Moguća unapređenja: dodatno podešavanje hiperparametara i primena složenijih algoritama.

## Pokretanje projekta

### Potrebni paketi

```r
install.packages(c(
  "tidyverse", "ggplot2", "dplyr", "tidyr", "knitr",
  "mice",          # imputacija (MICE / PMM)
  "car",           # VIF / GVIF
  "caret",         # trening i unakrsna validacija
  "glmnet",        # Ridge i Lasso
  "rpart",         # Decision Tree
  "randomForest",  # Random Forest
  "gbm",           # Gradient Boosting
  "vip"            # permutaciona važnost
))
```

### Koraci

1. Preuzmite ili klonirajte repozitorijum:
   ```bash
   git clone https://github.com/lukaJevtic1/marketing-sales-machine-learning.git
   ```
2. Preuzmite skup podataka sa [Kaggle-a](https://www.kaggle.com/datasets/abdelfattahibrahim/marketing-sales-dataset) i postavite fajl `marketing_sales_dataset.csv` u koren projekta (ako se već ne nalazi u repozitorijumu).
3. Otvorite projekat u RStudiju i postavite radni direktorijum:
   ```r
   setwd("putanja/do/foldera/projekta")
   ```
4. Otvorite `project.Rmd` i pokrenite ga (**Knit**), ili izvršavajte blokove koda redom.

> Rezultati su reproduktivni jer su korišćeni fiksni seed-ovi (`123`, `42`).

## Struktura repozitorijuma

```
├── project.Rmd                    # kompletna analiza (kod + objašnjenja)
├── project.pdf                    # izveštaj (knit)
├── marketing_sales_dataset.csv    # skup podataka
└── README.md
```
