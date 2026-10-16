# Basic Operations with Pandas

Pandas is a Python library for working with tabular data. Its `DataFrame` structure is similar to a database table and supports operations on rows and columns.

## Install and import

Install pandas (and NumPy, used later for a missing-value example):

```bash
python -m pip install pandas numpy
```

Import the libraries and check the pandas version:

```python
import pandas as pd
import numpy as np

print(pd.__version__)
```

## Create a DataFrame

An empty DataFrame can be created with `pd.DataFrame()`. More commonly, create one from a list or a dictionary of equally sized lists:

```python
empty_df = pd.DataFrame()
numbers_df = pd.DataFrame([1, 2, 3, 4, 5], columns=["numbers"])

data = {
	"name": ["Himanshu", "kaluBhai", "nerrerendr", "daksh"],
	"age": [33, 34, 35, 36],
	"salary": [23000, 34000, 45000, 24000],
}

df = pd.DataFrame(data)
print(df)
print(type(df))
```

## Inspect and rename columns

Use these attributes and methods to inspect a DataFrame:

```python
print(df.head())       # first five rows
print(df.tail(2))      # last two rows
print(df.shape)        # (number of rows, number of columns)
print(df.columns)      # column labels
df.info()              # column types and non-null counts
print(df.describe())   # summary statistics for numeric columns
```

Rename a column with `rename`. By default, it returns a new DataFrame; use the result or set `inplace=True` to change the existing one. Here, the rest of the examples use `monthly_salary`:

```python
df.rename(columns={"salary": "monthly_salary"}, inplace=True)
```

## Select rows and columns

Select one column by its label, or a list of columns using double brackets:

```python
print(df["name"])
print(df[["name", "monthly_salary"]])
```

Use `.loc` for label-based selection and filtering, and `.iloc` for position-based selection. The end position in an `.iloc` slice is excluded:

```python
print(df.loc[df["name"] == "Himanshu"])
print(df.loc[(df["name"] == "Himanshu") & (df["monthly_salary"] >= 20000)])
print(df.iloc[1])       # row at position 1
print(df.iloc[0:2])     # rows at positions 0 and 1
```

Boolean conditions can also filter a DataFrame directly. Wrap each condition in parentheses when combining conditions with `&` (and) or `|` (or):

```python
adults = df[df["age"] >= 18]
well_paid = df[(df["age"] >= 18) & (df["monthly_salary"] >= 25000)]
print(adults)
print(well_paid)
```

`where` keeps values where the condition is true and replaces values where it is false. It preserves the DataFrame's shape:

```python
print(df.where(df["age"] >= 35, other="not eligible"))
```

## Add and update data

Create columns by assigning a value or a list with one value per row. Add a row with `.loc` at a new index:

```python
df["bonus"] = df["monthly_salary"] * 0.20
df["department"] = ["hr", "management", "security", "technology"]

df.loc[len(df)] = ["adarsh", 19, 45000, 9000, "supply"]
```

Update a value by row and column, or update selected rows using a condition:

```python
df.loc[0, "monthly_salary"] = 65000
df.loc[df["name"] == "adarsh", "monthly_salary"] = 95000
```

## Work with dates

Add date columns with one date per row, then convert strings to pandas datetime values with `pd.to_datetime`:

```python
df["doj"] = pd.to_datetime([
	"2024-01-01", "2025-02-01", "2026-03-01", "2024-04-01", "2025-05-01"
])
df["dor"] = pd.to_datetime(
	["01-01-2026", "01-01-2027", "01-01-2028", "01-01-2029", "01-01-2030"],
	format="%d-%m-%Y",
)

print(df["doj"].dtype)
print(df["doj"].dt.year)
print(df["doj"].dt.month)
print(df["doj"].dt.day)
print(df["doj"].dt.day_name())
```

Use `Timedelta` for a fixed duration and `DateOffset` for calendar-based changes such as months or years:

```python
print(df["doj"] + pd.Timedelta(days=90))
print(df["doj"] + pd.DateOffset(months=3))
print(df["doj"] + pd.DateOffset(years=2))
```

For dates with mixed input formats, recent pandas versions support `format="mixed"`. Prefer an explicit format when all strings follow the same pattern:

```python
mixed_dates = pd.to_datetime(["1/2/2026", "2026-03-01"], format="mixed")
```

## Handle missing values

`isnull()` marks missing values, and summing its result counts them per column. Fill only the column that needs a replacement to avoid changing unrelated data types:

```python
df.loc[df["name"] == "Himanshu", "bonus"] = np.nan

print(df.isnull())
print(df.isnull().sum())

df["bonus"] = df["bonus"].fillna(0)
print(df.isnull().sum())
```

## Group, aggregate, sort, and count

Group rows by a value and aggregate a selected column. For datetime columns, `.dt.month` provides the month number:

```python
print(df.groupby(df["doj"].dt.month)["bonus"].sum())
print(df.groupby(df["doj"].dt.month)["bonus"].mean())
print(df[df["doj"].dt.month == 1].value_counts())
```

Sort by a column in ascending or descending order:

```python
print(df.sort_values(by="age", ascending=False))
print(df.sort_values(by="age"))
```

## Delete rows and columns

Drop columns by name with `columns=` and rows by index with `index=`. These calls return new DataFrames unless `inplace=True` is used:

```python
df.drop(columns=["monthly_salary"], inplace=True)
df.drop(index=df.index[df["name"] == "daksh"], inplace=True)
df.drop(index=1, inplace=True)
```

## Save and load CSV files

Use `to_csv` to save a DataFrame and `read_csv` to load it. Set `index=False` when the DataFrame index should not become an extra CSV column:

```python
df.to_csv("employees.csv", index=False)
loaded_data = pd.read_csv("employees.csv", parse_dates=["doj", "dor"])
print(loaded_data)
```

## Combine DataFrames

`concat` stacks DataFrames by rows by default (`axis=0`). `merge` joins them using a shared key, similar to a SQL join:

```python
dff = pd.DataFrame({
	"id": [1, 2, 3, 4],
	"sector": ["hr", "swipper", "tech_expert", "supporter"],
})
dff1 = pd.DataFrame({
	"id": [1, 2, 3, 4],
	"sector": ["hr1", "swipper1", "tech_expert1", "supporter1"],
})

stacked = pd.concat([dff, dff1], ignore_index=True)
joined = pd.merge(dff, dff1, how="inner", on="id", suffixes=("_left", "_right"))

print(stacked)
print(joined)
```