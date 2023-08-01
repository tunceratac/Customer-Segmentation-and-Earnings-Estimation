# Customer Segmentation and Earnings Estimation

A game company wants to create new customer segments based on certain customer features and estimate the average earnings for potential customers in each segment. The features used for segmentation are:
- `PRICE`: Customer's spending amount
- `SOURCE`: The type of device the customer is connecting to
- `SEX`: Gender of the client
- `COUNTRY`: Country of the customer
- `AGE`: Age of the customer

## Dataset

The data is loaded from the CSV file "persona.csv" into a pandas DataFrame.

```python
import pandas as pd

# Load the data into a DataFrame
df = pd.read_csv("persona.csv")
```

# Exploratory Data Analysis (EDA)
Now, let's explore the dataset to understand its contents and characteristics.

```python
# Check the shape of the DataFrame
print("Shape of the DataFrame:", df.shape)

# Display basic information about the DataFrame
df.info()

# How many unique SOURCE types are there?
num_unique_sources = df["SOURCE"].nunique()
print("Number of unique SOURCE types:", num_unique_sources)

# Frequency count of each SOURCE type
source_frequencies = df["SOURCE"].value_counts()
print("Frequency count of each SOURCE type:\n", source_frequencies)

# How many unique PRICEs are there?
num_unique_prices = df["PRICE"].nunique()
print("Number of unique PRICEs:", num_unique_prices)

# Sales count for each PRICE
price_sales_count = df["PRICE"].value_counts()
print("Sales count for each PRICE:\n", price_sales_count)

# Sales count from each country
sales_by_country = df.pivot_table(values="PRICE", index="COUNTRY", aggfunc="count")
print("Sales count by country:\n", sales_by_country)

# Total earnings from sales by country
earnings_by_country = df.pivot_table(values="PRICE", index="COUNTRY", aggfunc="sum")
print("Total earnings from sales by country:\n", earnings_by_country)

# Sales count for each SOURCE type
sales_by_source = df.pivot_table(values="PRICE", index="SOURCE", aggfunc="count")
print("Sales count by SOURCE type:\n", sales_by_source)

# Average PRICE by country
average_price_by_country = df.pivot_table(values="PRICE", index="COUNTRY", aggfunc="mean")
print("Average PRICE by country:\n", average_price_by_country)

# Average PRICE by SOURCE type
average_price_by_source = df.pivot_table(values="PRICE", index="SOURCE", aggfunc="mean")
print("Average PRICE by SOURCE type:\n", average_price_by_source)

# Average PRICE in the COUNTRY-SOURCE breakdown
average_price_country_source = df.pivot_table(values="PRICE", index=["COUNTRY", "SOURCE"], aggfunc="mean")
print("Average PRICE in the COUNTRY-SOURCE breakdown:\n", average_price_country_source)

# Average earnings breakdown by COUNTRY, SOURCE, SEX, and AGE
average_earnings_breakdown = df.pivot_table(values="PRICE", index=["COUNTRY", "SOURCE", "SEX", "AGE"], aggfunc="mean").head()
print("Average earnings breakdown by COUNTRY, SOURCE, SEX, and AGE:\n", average_earnings_breakdown)
```

# Customer Segmentation and Earnings Estimation
We will create customer segments based on the breakdown of COUNTRY, SOURCE, SEX, and AGE. Then, we will estimate the average earnings for potential customers in each segment.

```
# Convert the numeric variable AGE to a categorical variable with intervals
intervals = [0, 18, 23, 30, 40, 60, df["AGE"].max()]
labels = ["0-18", "19-23", "24-30", "30-40", "40-60", "60+"]
df["CAT_AGE"] = pd.cut(df["AGE"], intervals, labels=labels)

# Create the LEVEL_BASED variable by combining COUNTRY, SOURCE, SEX, and CAT_AGE
df["LEVEL_BASED"] = df[["COUNTRY", "SOURCE", "SEX", "CAT_AGE"]].agg(lambda x: '_'.join(x).upper(), axis=1)

# Drop unnecessary columns
df = df[["LEVEL_BASED", "PRICE"]]

# Calculate average PRICE based on LEVEL_BASED to determine the ranges
agg_df = df.groupby("LEVEL_BASED").agg({"PRICE": "mean"})
agg_df = agg_df.reset_index()

# Check if each LEVEL_BASED user is unique
level_based_count = agg_df["LEVEL_BASED"].value_counts()

# Create 4 separate ranges to segment the averaged PRICE variable
agg_df["SEGMENT"] = pd.qcut(agg_df["PRICE"], 4)

# Define a function to match the range values of a potential customer's age
def age_interval(age):
    if age <= 18:
        return "0-18"
    elif age <= 23:
        return "19-23"
    elif age <= 30:
        return "24-30"
    elif age <= 40:
        return "30-40"
    elif age <= 60:
        return "40-60"
    else:
        return "60+"

# Identify a new potential customer
country = "TUR"
source = "ANDROID"
sex = "MALE"
age = 26
user = country + "_" + source + "_" + sex + "_" + age_interval(age)

# Show expected earnings information from the potential customer
user_average_earnings = agg_df[agg_df["LEVEL_BASED"] == user]["PRICE"]
user_segment = agg_df[agg_df["LEVEL_BASED"] == user]["SEGMENT"]
print("Average earnings expected from this user:", user_average_earnings.to_string(index=False))
print("The expected revenue range from this user:", user_segment.to_string(index=False))
```

# Conclusion
In this analysis, we created customer segments based on the breakdown of COUNTRY, SOURCE, SEX, and AGE. Then, we estimated the average earnings for potential customers in each segment. The game company can use this information to target different customer segments with appropriate marketing strategies and pricing.
