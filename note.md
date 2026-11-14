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