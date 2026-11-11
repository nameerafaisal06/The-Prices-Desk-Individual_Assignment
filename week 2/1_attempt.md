**Attempt before AI****
__Diagnosis:__
Before running any AI tools to clean the data, I audited the raw dataset "teaching_prices_messy"

1) I imported the dataset as pandas and saved it as variable named 'raw'.

2) Inspected data structure and shape using raw.info(), raw.shape , raw.head() , and raw.tail()

3) I discovered there were exact duplicate row, using raw.duplicated().sum(), which returned 1 duplicate row (row 96). I planned to drop exact duplicate using clean = raw.copy() then clean = clean.drop_duplicates(). I did this to remove identical inputs which could lead to distorted analysis

4)Checked for missing values using raw.isna().sum() which showed 1 missing value in "price" column in row 5. I took the decision to not invent missing data to prevent jumbling of data in the stats.

5) Corrected inconsistent item formatting using clean["item"] = clean["item"].str.strip().str.title() and then checked by clean["item"].value_counts() . Also, checked subset duplicates across ["date","item","city"] using clean.duplicated(subset=["date","item","city"]).sum() and dropped extra using clean = clean.drop_duplicates(subset=["date","item","city"])

6)Fixed string price formatting in row 17 (1,250) by stripping the comma using clean ["price"] =pd.to_numeric(clean ["price"].str.replace(",",""), errors = "coerce") to store it as a numeric float, and converted the data columns to datetime by using clean ["date"] =pd.to_datetime(clean ["date"])
