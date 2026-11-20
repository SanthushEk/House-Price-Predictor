# Day 1 Notes

## Dataset
- 20,640 rows, 9 columns
- Target: MedHouseVal (house value in $100,000s)
- No missing values

## Columns
- MedInc: median income of the area (in $10,000s)
- HouseAge: median house age
- AveRooms / AveBedrms: average rooms / bedrooms per house
- Population: people living in the area
- AveOccup: average people per house
- Latitude / Longitude: location

## Things I noticed
- MedHouseVal is capped at 5.0 ($500k)
- HouseAge capped at 52, MedInc capped at 15
- Big outliers in AveRooms (max 141) and AveOccup (max 1,243)
- Features have very different scales

## What I learned
- Regression = predicting a number. This project is regression.

## Day 2 – Charts and correlations (branch: day-2-eda)

### What we did and why
1. **Created a branch `day-2-eda`** (`git switch -c day-2-eda`)
   - Why: work on a copy so `main` stays clean and working.
2. **Made `02_eda.ipynb`** and loaded `train.csv` (20,640 rows, 9 columns)
   - Why: a separate notebook keeps Day 1 (numbers) apart from Day 2 (charts).
3. **Histogram of house value**
   - Why: shows how prices are spread out. Found a spike at 5.0 (the cap).
4. **Correlation table + heatmap**
   - Why: correlation (-1 to +1) shows which features move with price.
5. **Scatter: income vs price**
   - Why: shows the shape of the relationship, not just one number.
6. **California price map** (longitude/latitude, colour = price, size = population)
   - Why: location matters a lot for house prices.
7. **Boxplots + IQR outlier count**
   - Why: see and count extreme values before deciding what to do on Day 5.

### Answers
**1. Strongest feature?**
`MedInc` (median income), correlation **0.69** with price. Richer areas have more
expensive houses. The next feature is only 0.15 (`AveRooms`), so income is far ahead
of the rest. Latitude (-0.14) and HouseAge (0.11) are weak.

**2. Most expensive areas?**
The coast, mainly the Bay Area (average value about 2.74, i.e. ~$274k)
and the Los Angeles area (about 2.40). The Central Valley inland is much cheaper
(about 0.96). The overall average is 2.07.

**3. How many outliers? Should we worry?**
| Feature | Outliers |
|---------|----------|
| MedInc | 681 |
| AveRooms | 511 |
| AveBedrms | 1,424 |
| AveOccup | 711 |
| Population | 1,196 |
- **992 houses (4.8%) are capped at 5.0.** Their real price is unknown, so the
  model will under-predict the most expensive houses.
- Yes, worth watching. Extreme values (like AveOccup = 1,243) can distort Linear
  Regression, but tree models (Random Forest) handle them better. I'll test this
  on Day 3–5 instead of deleting them now.

### Other things I noticed
- `AveRooms` and `AveBedrms` are strongly linked (0.85), so they carry similar information.
- `Latitude` and `Longitude` are strongly linked (-0.92) because California runs diagonally.
- Features have different scales, so some models will need scaling later.

### What I learned
- Correlation shows how strongly two things move together, but not why.
- A chart can show things a table hides (like the 5.0 cap line).
- Outliers are counted first and handled later.
- Branch workflow: `switch -c` → work → `push -u origin` → pull request → merge.

## Day 3 – First model: Linear Regression (branch: day-3-linear-regression)

### What we did and why
1. **Created a branch `day-3-linear-regression`**
   - Why: keep Day 3 work separate so `main` stays clean.
2. **Split the data into X (features) and y (price)**
   - Why: X is what the model sees (8 columns). y is the answer it must predict.
3. **Train/test split (80% / 20%, `random_state=42`)**
   - Why: test the model on houses it has never seen, like an exam.
     16,512 houses to train, 4,128 to test. `random_state` makes the split repeatable.
4. **Baseline model (always predicts the average price)**
   - Why: a score means nothing without something to compare to.
5. **Trained Linear Regression** (`fit` then `predict`)
   - Why: it learns one weight per feature, then applies them to new houses.
6. **Measured with MAE, RMSE and R²**
   - Why: three ways to see how wrong the model is.
7. **Looked at the weights and a predicted vs actual chart**
   - Why: to understand what the model learned and where it fails.

### Results
| Model | MAE | RMSE | R² |
|-------|-----|------|----|
| Baseline (average) | 0.906 | 1.145 | 0.00 |
| Linear Regression | 0.533 | 0.746 | 0.576 |

- **MAE 0.533 means the model is about $53,000 off on average** (prices are in $100,000s).
- It beats the baseline (R² 0.58 vs 0.00), so it did learn something real.
- But it explains only about 58% of the price changes. A lot is still missing.

### What the model learned (weights)
- Positive: `MedInc` (0.449), `AveBedrms` (0.783)
- Negative: `Latitude` (-0.42), `Longitude` (-0.434), `AveRooms` (-0.123)
- `AveBedrms` is large and positive while `AveRooms` is negative. This looks odd,
  but those two features are strongly linked (0.85), so the model splits their
  effect strangely. Weights can't always be trusted when features overlap.
- The two location weights are both negative and large. Price depends on location
  in a curved way (coast vs inland), and a straight-line formula can't capture that.

### What the chart showed
- Dots should sit on the red line. They spread out a lot.
- The 184 houses capped at 5.0 in the test set are predicted at only about 3.85 on
  average. The model can't reach the top of the price range.
- A few predictions are impossible: 15 houses get a negative price, and the highest
  prediction is about 11.5 (above the 5.0 cap). Linear Regression doesn't know
  prices have limits.