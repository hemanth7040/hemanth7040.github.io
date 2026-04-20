---
title: "Phase 1: Replace Your Shell Scripts With Python — Data Literacy for DevOps Engineers"
date: 2026-04-19T08:00:00+05:30
draft: false
categories: ["python"]
description: "Learn NumPy, Pandas, Matplotlib, Scikit-learn, error handling, and file I/O — the six tools every DevOps engineer needs to work with data, logs, and AI pipelines."
series: ["Python Zero to MLOps"]
series_order: 4
tags: ["NumPy", "DevOps", "DataLiteracy", "Beginners", "Ai"]
---

> **Who this is for:** You finished Phase 0 and know Python basics. Now we learn the tools
> that turn your Python knowledge into real DevOps and data work. No data science background needed.

---

## Why Does a DevOps Engineer Need Data Tools?

You already write Bash scripts to parse logs, count errors, and monitor systems.
Python's data tools do the same things — but faster, with less code, and on much larger files.

Here is a real comparison:

**Bash** — count how many times "ERROR" appears in a log file:
```bash
grep -c "ERROR" app.log
```

**Python + Pandas** — count errors, group them by hour, and find the busiest hour:
```python
import pandas as pd

df = pd.read_csv("app.log")
df[df["level"] == "ERROR"].groupby("hour").size().idxmax()
```

One does counting. The other does analysis. Both are useful — but as you move toward MLOps,
you need the analysis power.

---

## What We Cover in Phase 1

| Tool | What It Does | Why You Need It |
|------|-------------|----------------|
| NumPy | Fast number arrays | Speeds up any script that processes large numbers |
| Pandas | Read and filter data | Replaces grep/awk for log and CSV analysis |
| Matplotlib / Seaborn | Draw charts | Spot patterns visually — like data drift in ML pipelines |
| Scikit-learn | Machine learning basics | Understand what data scientists hand you |
| Error Handling | Catch failures gracefully | Stop production pipelines from crashing silently |
| File I/O | Read YAML, JSON, TXT | Parse config files and write output logs |

---

## Setting Up

Install everything in one command using uv (as we set up in Phase 0 setup post):

```bash
uv pip install numpy pandas matplotlib seaborn scikit-learn
```

Or with pip:

```bash
pip install numpy pandas matplotlib seaborn scikit-learn
```

---

## 1. NumPy — Fast Number Arrays

### What Is NumPy?

NumPy stands for **Numerical Python**. It gives Python a special data structure called an
**array** that is much faster than a regular Python list when working with numbers.

Think of it this way:
- Python list = a shopping bag (flexible, slow to search)
- NumPy array = a neatly packed box (strict, very fast)

### Why Is It Faster?

A regular Python list can hold anything — strings, numbers, booleans all mixed together.
Because of this flexibility, Python checks every item individually when doing math.

A NumPy array holds **only one type** (like only integers, or only floats).
Because of this, NumPy can process all items at once — the same way a CPU processes
multiple instructions in a single clock cycle.

### Your First NumPy Array

```python
import numpy as np

# Create a regular Python list
python_list = [10, 20, 30, 40, 50]

# Create a NumPy array from that list
np_array = np.array([10, 20, 30, 40, 50])

print(python_list)   # [10, 20, 30, 40, 50]
print(np_array)      # [10 20 30 40 50]
```

Notice NumPy does not show commas. That is just how it displays — the data is the same.

### The Big Difference — Math on a Whole Array

```python
# With a Python list — you need a loop
python_list = [10, 20, 30, 40, 50]
doubled = [x * 2 for x in python_list]
print(doubled)   # [20, 40, 60, 80, 100]

# With NumPy — one line, no loop needed
np_array = np.array([10, 20, 30, 40, 50])
doubled = np_array * 2
print(doubled)   # [20 40 60 80 100]
```

This is called **vectorization** — doing an operation on the whole array at once.
It is not just shorter code — it is genuinely 10x to 100x faster on large datasets.

### Common NumPy Operations

```python
import numpy as np

# Create arrays
a = np.array([1, 2, 3, 4, 5])
b = np.array([10, 20, 30, 40, 50])

# Basic math — all done element by element
print(a + b)    # [11 22 33 44 55]
print(a * b)    # [10 40 90 160 250]
print(b / a)    # [10. 10. 10. 10. 10.]

# Statistics
print(np.mean(a))    # 3.0  — average
print(np.sum(a))     # 15   — total
print(np.max(b))     # 50   — highest value
print(np.min(b))     # 10   — lowest value
print(np.std(a))     # 1.41 — standard deviation (how spread out the values are)
```

### Creating Arrays Without Typing Each Number

```python
import numpy as np

# A range of numbers (like Python's range(), but returns an array)
print(np.arange(0, 10, 2))      # [0 2 4 6 8]
# arange(start, stop, step) — stop is not included

# 5 evenly spaced numbers between 0 and 1
print(np.linspace(0, 1, 5))     # [0.   0.25 0.5  0.75 1.  ]
# linspace(start, stop, count) — stop IS included

# An array of all zeros (useful as a starting placeholder)
print(np.zeros(5))              # [0. 0. 0. 0. 0.]

# An array of all ones
print(np.ones(5))               # [1. 1. 1. 1. 1.]
```

### 2D Arrays — Working With Tables of Numbers

```python
import numpy as np

# A 2D array is like a table — rows and columns
matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
])

print(matrix.shape)   # (3, 3) — 3 rows, 3 columns

# Access a specific value: [row, column] — both start at 0
print(matrix[0, 0])   # 1  — first row, first column
print(matrix[1, 2])   # 6  — second row, third column

# Access a full row
print(matrix[0])      # [1 2 3]

# Access a full column
print(matrix[:, 1])   # [2 5 8]  — all rows, second column
```

### Real DevOps Use Case — Analyzing Response Times

```python
import numpy as np

# Imagine these are API response times in milliseconds from your logs
response_times = np.array([120, 145, 98, 210, 134, 88, 310, 102, 145, 99])

print(f"Average response time: {np.mean(response_times):.1f}ms")
print(f"Slowest request: {np.max(response_times)}ms")
print(f"Fastest request: {np.min(response_times)}ms")

# Find requests slower than 200ms (potential issues)
slow_requests = response_times[response_times > 200]
print(f"Slow requests: {slow_requests}")        # [210 310]
print(f"Slow request count: {len(slow_requests)}")   # 2
```

The line `response_times[response_times > 200]` is called **boolean indexing**.
`response_times > 200` creates a True/False array, and NumPy returns only the items
where the value is True. No loop needed.

---

## 2. Pandas — The Swiss Army Knife for Data

### What Is Pandas?

Pandas gives Python a data structure called a **DataFrame** — think of it as a spreadsheet
that lives in your code. It has rows, columns, headers, and powerful filtering built in.

As a DevOps engineer, any time you have a CSV log file, a metrics export, or a JSON report,
Pandas is the right tool to read and analyze it.

### The Two Core Pandas Objects

**Series** — a single column of data (like one column in a spreadsheet):
```python
import pandas as pd

# A Series is like a labeled NumPy array
temperatures = pd.Series([22.5, 24.1, 19.8, 26.3, 21.0],
                          index=["Mon", "Tue", "Wed", "Thu", "Fri"])

print(temperatures)
# Mon    22.5
# Tue    24.1
# Wed    19.8
# Thu    26.3
# Fri    21.0
# dtype: float64

print(temperatures["Wed"])   # 19.8
print(temperatures.mean())   # 22.74
```

**DataFrame** — multiple columns together (a full table):
```python
import pandas as pd

# Create a DataFrame from a dictionary
# Each key becomes a column name
# Each list becomes the column's data
data = {
    "name":    ["Alice", "Bob", "Charlie", "Diana"],
    "role":    ["DevOps", "MLOps", "DevOps", "MLOps"],
    "exp":     [3, 5, 2, 7],
    "salary":  [80000, 95000, 70000, 110000]
}

df = pd.DataFrame(data)
print(df)
```

Output:
```
      name   role  exp  salary
0    Alice  DevOps    3   80000
1      Bob   MLOps    5   95000
2  Charlie  DevOps    2   70000
3    Diana   MLOps    7  110000
```

The numbers on the left (0, 1, 2, 3) are the **index** — the row numbers.

### Reading Real Files

In real work you will rarely type data by hand. You will read files:

```python
import pandas as pd

# Read a CSV file — the most common format
df = pd.read_csv("app_logs.csv")

# Read a JSON file
df = pd.read_json("metrics.json")

# Read an Excel file (needs openpyxl: pip install openpyxl)
df = pd.read_excel("report.xlsx")
```

### Exploring a DataFrame — Always Do This First

Before analyzing any data, explore it to understand what you are working with:

```python
import pandas as pd

df = pd.read_csv("app_logs.csv")

print(df.shape)        # (1000, 6) — 1000 rows, 6 columns

print(df.head())       # first 5 rows — quick preview
print(df.tail(3))      # last 3 rows

print(df.columns)      # list of all column names
print(df.dtypes)       # data type of each column (int, float, object, etc.)

print(df.info())       # full summary — columns, types, non-null counts

print(df.describe())   # statistics for number columns (mean, min, max, etc.)

print(df.isnull().sum())   # count of missing values per column
```

### Selecting Columns and Rows

```python
import pandas as pd

data = {
    "service": ["api", "db", "cache", "api", "db"],
    "level":   ["INFO", "ERROR", "INFO", "ERROR", "INFO"],
    "latency": [45, 320, 12, 89, 156]
}
df = pd.DataFrame(data)

# Select one column — returns a Series
print(df["service"])

# Select multiple columns — pass a list of names
print(df[["service", "latency"]])

# Select rows by index number using iloc (integer location)
print(df.iloc[0])      # first row
print(df.iloc[1:3])    # rows 1 and 2 (row 3 not included)

# Select rows by label using loc
print(df.loc[0])       # row with index label 0
```

### Filtering Rows — Like grep But Smarter

```python
import pandas as pd

data = {
    "service": ["api", "db", "cache", "api", "db"],
    "level":   ["INFO", "ERROR", "INFO", "ERROR", "INFO"],
    "latency": [45, 320, 12, 89, 156]
}
df = pd.DataFrame(data)

# Filter rows where level is ERROR
errors = df[df["level"] == "ERROR"]
print(errors)
#   service  level  latency
# 1      db  ERROR      320
# 3     api  ERROR       89

# Filter rows where latency is over 100ms
slow = df[df["latency"] > 100]
print(slow)

# Combine two conditions — use & for AND, | for OR
# Must wrap each condition in parentheses
slow_errors = df[(df["level"] == "ERROR") & (df["latency"] > 100)]
print(slow_errors)
```

### Adding and Removing Columns

```python
import pandas as pd

data = {"latency": [45, 320, 12, 89, 156]}
df = pd.DataFrame(data)

# Add a new column — calculated from existing data
df["latency_sec"] = df["latency"] / 1000
print(df)
#    latency  latency_sec
# 0       45        0.045
# 1      320        0.320
# 2       12        0.012

# Add a column with a fixed value
df["environment"] = "production"

# Remove a column
df = df.drop(columns=["latency_sec"])
```

### Grouping and Aggregating Data

This is where Pandas really beats Bash. Group rows by a column and calculate stats for each group:

```python
import pandas as pd

data = {
    "service": ["api", "db", "cache", "api", "db", "api"],
    "level":   ["INFO", "ERROR", "INFO", "ERROR", "INFO", "INFO"],
    "latency": [45, 320, 12, 89, 156, 67]
}
df = pd.DataFrame(data)

# Count rows per service
print(df.groupby("service").size())
# service
# api      3
# cache    1
# db       2

# Average latency per service
print(df.groupby("service")["latency"].mean())
# service
# api      67.0
# cache    12.0
# db      238.0

# Multiple stats at once
print(df.groupby("service")["latency"].agg(["mean", "max", "count"]))
```

### Handling Missing Values

Real data always has gaps. Pandas makes it easy to find and fix them:

```python
import pandas as pd
import numpy as np

data = {
    "service": ["api", "db", None, "api"],
    "latency": [45, np.nan, 12, 89]
}
df = pd.DataFrame(data)

print(df.isnull())          # True/False for each cell
print(df.isnull().sum())    # count of missing values per column

# Drop rows that have any missing value
df_clean = df.dropna()

# Fill missing values with a default
df_filled = df.fillna({"service": "unknown", "latency": 0})
```

### Sorting and Saving

```python
import pandas as pd

data = {
    "service": ["api", "db", "cache"],
    "latency": [89, 320, 12]
}
df = pd.DataFrame(data)

# Sort by latency — highest first
df_sorted = df.sort_values("latency", ascending=False)
print(df_sorted)
#   service  latency
# 1      db      320
# 0     api       89
# 2   cache       12

# Save to CSV (no index column written to file)
df_sorted.to_csv("sorted_report.csv", index=False)

# Save to JSON
df_sorted.to_json("sorted_report.json", orient="records", indent=2)
```

---

## 3. Matplotlib and Seaborn — See Your Data

### What Are These Libraries?

**Matplotlib** is Python's foundational charting library. Everything builds on it.
**Seaborn** sits on top of Matplotlib and produces prettier charts with less code.

As a DevOps engineer moving toward MLOps, you need charts to:
- Spot unusual patterns in metrics (anomalies)
- Check whether data has changed over time (data drift)
- Show stakeholders what is happening in a system

### Your First Chart — A Line Chart

```python
import matplotlib.pyplot as plt

# Hourly error counts from your log pipeline
hours = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
errors = [2, 1, 0, 3, 1, 0, 5, 12, 18, 22, 19, 15]

plt.figure(figsize=(10, 4))
# figsize sets the width and height in inches

plt.plot(hours, errors, marker="o", color="red", linewidth=2)
# marker="o" puts a dot at each data point
# color sets the line color
# linewidth sets how thick the line is

plt.title("Hourly Error Count")
plt.xlabel("Hour of Day")
plt.ylabel("Number of Errors")
plt.grid(True)
# grid adds the faint background lines — makes it easier to read values

plt.tight_layout()
# tight_layout prevents labels from getting cut off

plt.savefig("errors.png")   # saves the chart as an image file
plt.show()                   # displays it if running interactively
```

### Bar Chart — Compare Categories

```python
import matplotlib.pyplot as plt

services = ["api", "db", "cache", "queue"]
error_counts = [45, 12, 3, 28]

plt.figure(figsize=(8, 4))
plt.bar(services, error_counts, color=["#e74c3c", "#3498db", "#2ecc71", "#f39c12"])
# Each bar gets a different color — makes it easy to tell apart

plt.title("Errors by Service (Last 24 Hours)")
plt.xlabel("Service")
plt.ylabel("Error Count")

# Add the number on top of each bar
for i, count in enumerate(error_counts):
    plt.text(i, count + 0.5, str(count), ha="center", fontweight="bold")
    # ha="center" means the text is centered above the bar

plt.tight_layout()
plt.savefig("service_errors.png")
```

### Seaborn — Better Charts With Less Code

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd

data = {
    "service": ["api", "db", "cache", "api", "db", "api", "cache"],
    "latency": [45, 320, 12, 89, 156, 67, 18]
}
df = pd.DataFrame(data)

plt.figure(figsize=(8, 4))

# Box plot — shows the spread of latency per service
# The box shows the middle 50% of values
# The line in the box is the median
# The dots outside are outliers
sns.boxplot(data=df, x="service", y="latency", palette="Set2")

plt.title("Latency Distribution by Service")
plt.ylabel("Latency (ms)")
plt.tight_layout()
plt.savefig("latency_boxplot.png")
```

### Histogram — Understand Data Distribution

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

# Simulate 1000 response times
response_times = np.random.normal(loc=150, scale=40, size=1000)
# normal distribution centered at 150ms with spread of 40ms

plt.figure(figsize=(8, 4))
sns.histplot(response_times, bins=30, kde=True, color="steelblue")
# bins=30 means divide the range into 30 buckets
# kde=True adds a smooth curve showing the shape of the distribution

plt.title("Response Time Distribution")
plt.xlabel("Response Time (ms)")
plt.ylabel("Count")
plt.tight_layout()
plt.savefig("response_distribution.png")
```

The **KDE curve** (the smooth line) shows you the shape of your data.
If it shifts to the right over time — your service is getting slower. That is data drift.

---

## 4. Scikit-learn — Understand What Data Scientists Hand You

### Why Does a DevOps Engineer Need This?

You do not need to train machine learning models.
But you do need to understand what a model is, what it expects, and what it returns —
so you can write the pipeline that feeds it data and serves its predictions.

Scikit-learn is the most common Python ML library. Knowing it lets you:
- Read and understand a data scientist's code
- Write validation scripts that check model inputs
- Test that a model returns expected output types

### The Three Steps Every ML Model Follows

Every model in Scikit-learn (and in ML in general) follows the same three steps:

1. **Fit** — show the model training data so it can learn patterns
2. **Transform** — use what it learned to process new data
3. **Predict** — ask the model to make a decision on new data

```python
from sklearn.preprocessing import StandardScaler
import numpy as np

# Training data — imagine these are CPU usage percentages from your servers
training_data = np.array([[45], [72], [88], [55], [61], [90], [48]])

# Step 1: Fit — the scaler learns the mean and standard deviation
scaler = StandardScaler()
scaler.fit(training_data)

print(f"Mean learned: {scaler.mean_[0]:.1f}")   # 65.6
print(f"Std learned: {scaler.scale_[0]:.1f}")   # 17.8

# Step 2: Transform — scale the data so all values are on the same scale
scaled = scaler.transform(training_data)
print(scaled[:3])
# [[-1.16]
#  [ 0.36]
#  [ 1.26]]
# Values are now centered around 0
```

**Why scale data?** ML models are sensitive to the size of numbers.
If one feature is "CPU usage (0-100)" and another is "memory (0-65536 MB)",
the model pays too much attention to memory just because the numbers are bigger.
Scaling makes everything comparable.

### A Simple Prediction Example

```python
from sklearn.linear_model import LinearRegression
import numpy as np

# Training data: hours of load vs CPU temperature
# X = input (hours under load), y = output (temperature)
X_train = np.array([[1], [2], [3], [4], [5], [6], [7], [8]])
y_train = np.array([35, 38, 42, 47, 53, 60, 68, 77])

# Create the model
model = LinearRegression()

# Step 1: Fit — model learns the relationship between load hours and temperature
model.fit(X_train, y_train)

# Step 3: Predict — what temperature after 10 hours of load?
prediction = model.predict([[10]])
print(f"Predicted temperature at 10 hours: {prediction[0]:.1f}°C")
# Predicted temperature at 10 hours: 94.5°C
```

This is the **exact same pattern** a data scientist uses for complex models.
You do not need to understand the math — you need to know the pattern: fit → predict.

### The Train/Test Split — Why It Matters

```python
from sklearn.model_selection import train_test_split
import numpy as np

# Imagine 100 rows of log data
X = np.random.rand(100, 3)   # 100 rows, 3 features each
y = np.random.randint(0, 2, 100)   # 100 labels (0 or 1)

# Split into 80% training, 20% testing
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

print(f"Training rows: {len(X_train)}")   # 80
print(f"Testing rows:  {len(X_test)}")    # 20
```

**Why split?** If you test the model on the same data it trained on, it will look
artificially perfect — like a student who memorized answers instead of understanding them.
The test set is data the model has never seen, so the score is honest.

As a DevOps engineer, your job is to make sure this split is **reproducible** — that is
what `random_state=42` does. It seeds the random number generator so the split is
identical every time the pipeline runs.

---

## 5. Error Handling — Stop Pipelines Crashing Silently

### The Problem With Silent Failures

In Bash, a failed command often silently continues to the next line.
In Python, an unhandled error crashes the entire script — but only if you let it reach that point uncaught.

In a production pipeline, you want neither:
- **Silent failure** — data gets corrupted without anyone knowing
- **Hard crash** — entire pipeline stops for a recoverable issue

Error handling lets you respond intelligently to failures.

### Basic Try / Except

```python
# Without error handling — crashes the whole script
result = 10 / 0
print(result)   # ZeroDivisionError — script stops here

# With error handling — controlled response
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero — using default value")
    result = 0

print(f"Result: {result}")   # Result: 0
# Script continues normally
```

**How it works:**
- Python runs the code inside `try`
- If an error happens, it immediately jumps to `except`
- If no error happens, `except` is skipped entirely

### Catching Multiple Error Types

```python
def read_config(filepath):
    try:
        with open(filepath, "r") as f:
            import json
            config = json.load(f)
            return config

    except FileNotFoundError:
        # The file does not exist at that path
        print(f"Config file not found: {filepath}")
        return {}

    except json.JSONDecodeError as e:
        # The file exists but is not valid JSON
        print(f"Config file has invalid JSON: {e}")
        return {}

    except PermissionError:
        # The file exists but we do not have read access
        print(f"No permission to read: {filepath}")
        return {}

# Usage — safe, never crashes
config = read_config("/etc/myapp/config.json")
```

### The `else` and `finally` Blocks

```python
def process_log_file(filepath):
    file = None
    try:
        file = open(filepath, "r")
        data = file.read()

    except FileNotFoundError:
        print(f"Log file missing: {filepath}")
        data = ""

    else:
        # This block only runs if NO error happened in try
        print(f"Successfully read {len(data)} characters")

    finally:
        # This block ALWAYS runs — error or no error
        # Use it for cleanup
        if file:
            file.close()
            print("File closed.")

    return data
```

`else` — runs only on success. Good for logging "it worked".
`finally` — always runs. Good for closing files, database connections, or network sockets.

### Raising Your Own Errors

```python
def validate_latency(value):
    if not isinstance(value, (int, float)):
        raise TypeError(f"Latency must be a number, got {type(value).__name__}")

    if value < 0:
        raise ValueError(f"Latency cannot be negative: {value}")

    if value > 10000:
        raise ValueError(f"Latency too high — possible bad data: {value}ms")

    return value

# Test it
try:
    validate_latency(-50)
except ValueError as e:
    print(f"Validation failed: {e}")

try:
    validate_latency("fast")
except TypeError as e:
    print(f"Wrong type: {e}")
```

Raising your own errors is how you make pipelines **self-documenting**.
When something goes wrong, the error message tells you exactly what and why.

### Real Pipeline Pattern

```python
import pandas as pd

def load_and_validate(filepath):
    """Load a CSV log file and validate it has the columns we expect."""

    try:
        df = pd.read_csv(filepath)

    except FileNotFoundError:
        raise FileNotFoundError(f"Log file not found: {filepath}")

    except pd.errors.EmptyDataError:
        raise ValueError(f"Log file is empty: {filepath}")

    # Validate required columns exist
    required_columns = ["timestamp", "service", "level", "latency"]
    missing = [col for col in required_columns if col not in df.columns]

    if missing:
        raise ValueError(f"Missing required columns: {missing}")

    # Validate no nulls in critical columns
    if df["service"].isnull().any():
        raise ValueError("Found null values in 'service' column")

    print(f"Loaded {len(df)} rows from {filepath}")
    return df
```

This pattern — load, validate, raise clear errors — is what makes production pipelines trustworthy.

---

## 6. File I/O — Reading YAML, JSON, and Text Files

### Why This Matters for DevOps

Every DevOps tool uses config files:
- Kubernetes uses YAML
- Docker Compose uses YAML
- APIs return JSON
- Logs are plain text

Python can read, modify, and write all of these — making it perfect for config automation.

### Reading Plain Text Files

```python
# Read the whole file as one string
with open("app.log", "r") as f:
    content = f.read()
    print(f"Total characters: {len(content)}")

# Read line by line — memory-efficient for large files
with open("app.log", "r") as f:
    for line in f:
        line = line.strip()   # remove the newline character at the end
        if "ERROR" in line:
            print(line)
```

**Why use `with`?** The `with` statement automatically closes the file when the block ends —
even if an error happens inside. Without it, you must call `file.close()` manually,
and if an error happens first, the file stays open.

### Writing Text Files

```python
report_lines = [
    "=== Daily Error Report ===",
    "Date: 2026-04-20",
    "Total errors: 47",
    "Critical: 3",
    "High: 12",
    "Medium: 32",
]

# "w" = write mode — creates the file, or overwrites it if it exists
with open("daily_report.txt", "w") as f:
    for line in report_lines:
        f.write(line + "\n")   # \n adds a line break after each line

# "a" = append mode — adds to the end without deleting existing content
with open("daily_report.txt", "a") as f:
    f.write("Generated by Python pipeline\n")
```

### Reading and Writing JSON

JSON is the most common format for APIs and data exchange:

```python
import json

# --- Reading JSON ---
with open("config.json", "r") as f:
    config = json.load(f)
    # json.load() converts JSON text into a Python dictionary

print(config["model"])          # access values like a dict
print(config.get("timeout", 30))   # .get() with a default if key is missing

# --- Writing JSON ---
pipeline_config = {
    "name": "log-processor",
    "version": "1.0.0",
    "settings": {
        "batch_size": 1000,
        "timeout_seconds": 30,
        "retry_attempts": 3
    },
    "enabled": True
}

with open("pipeline_config.json", "w") as f:
    json.dump(pipeline_config, f, indent=2)
    # indent=2 makes the file human-readable with proper indentation

# --- Converting JSON string (from an API response) ---
json_string = '{"status": "ok", "count": 42}'
data = json.loads(json_string)   # loads() for string, load() for file
print(data["count"])   # 42
```

### Reading and Writing YAML

YAML is what Kubernetes, Docker Compose, and GitHub Actions use:

```python
# First install: pip install pyyaml
import yaml

# --- Reading YAML ---
with open("deployment.yaml", "r") as f:
    manifest = yaml.safe_load(f)
    # safe_load() is safer than load() — use it always

print(manifest["metadata"]["name"])
print(manifest["spec"]["replicas"])

# --- Writing YAML ---
new_config = {
    "apiVersion": "apps/v1",
    "kind": "Deployment",
    "metadata": {
        "name": "my-app",
        "namespace": "production"
    },
    "spec": {
        "replicas": 3
    }
}

with open("new_deployment.yaml", "w") as f:
    yaml.dump(new_config, f, default_flow_style=False)
    # default_flow_style=False makes it write block style (readable)
    # default_flow_style=True writes it all on one line (compact)
```

### Reading CSV Files With and Without Pandas

```python
# With Pandas — recommended for data analysis
import pandas as pd

df = pd.read_csv("metrics.csv")
print(df.head())

# Without Pandas — for simple scripts where you don't want the dependency
import csv

with open("metrics.csv", "r") as f:
    reader = csv.DictReader(f)
    # DictReader uses the first row as column names
    for row in reader:
        print(f"Service: {row['service']}, Latency: {row['latency']}")
```

Use Pandas when you need filtering, grouping, or analysis.
Use the `csv` module when you just need to loop through rows without extra dependencies.

---

## Putting It All Together — A Real DevOps Script

Here is a script that uses everything from Phase 1 together.
It reads a log CSV, analyzes errors, generates a report, and saves it.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import json
from datetime import datetime

def analyze_logs(filepath):
    """
    Read a log file, find errors, calculate stats,
    save a chart, and write a JSON report.
    """

    # --- Load and validate ---
    try:
        df = pd.read_csv(filepath)
    except FileNotFoundError:
        raise FileNotFoundError(f"Log file not found: {filepath}")

    required = ["timestamp", "service", "level", "latency"]
    missing = [c for c in required if c not in df.columns]
    if missing:
        raise ValueError(f"Missing columns: {missing}")

    print(f"Loaded {len(df)} log entries")

    # --- Analysis ---
    total_errors = len(df[df["level"] == "ERROR"])
    error_rate = (total_errors / len(df)) * 100

    avg_latency = np.mean(df["latency"].values)
    p95_latency = np.percentile(df["latency"].values, 95)
    # p95 = 95th percentile — 95% of requests are faster than this value

    errors_by_service = df[df["level"] == "ERROR"].groupby("service").size()

    # --- Chart ---
    plt.figure(figsize=(8, 4))
    errors_by_service.plot(kind="bar", color="crimson")
    plt.title("Errors by Service")
    plt.ylabel("Error Count")
    plt.tight_layout()
    plt.savefig("error_chart.png")
    plt.close()

    # --- Report ---
    report = {
        "generated_at": datetime.now().isoformat(),
        "total_requests": len(df),
        "total_errors": total_errors,
        "error_rate_pct": round(error_rate, 2),
        "avg_latency_ms": round(avg_latency, 1),
        "p95_latency_ms": round(p95_latency, 1),
        "errors_by_service": errors_by_service.to_dict()
    }

    with open("log_report.json", "w") as f:
        json.dump(report, f, indent=2)

    print(f"Error rate: {error_rate:.1f}%")
    print(f"Average latency: {avg_latency:.1f}ms")
    print(f"P95 latency: {p95_latency:.1f}ms")
    print("Report saved to log_report.json")
    print("Chart saved to error_chart.png")

    return report

# Run it
analyze_logs("app_logs.csv")
```

---

## Summary — What You Learned in Phase 1

| Tool | Key Skill | DevOps Use |
|------|-----------|-----------|
| NumPy | Fast array math, boolean indexing | Analyze metrics without loops |
| Pandas | Read CSV/JSON, filter rows, group data | Replace grep/awk for log analysis |
| Matplotlib | Line charts, bar charts | Visualize error trends |
| Seaborn | Box plots, histograms | Spot anomalies and data drift |
| Scikit-learn | Fit / transform / predict pattern | Understand ML pipelines |
| Error handling | try/except, raise, finally | Build pipelines that fail gracefully |
| File I/O | JSON, YAML, CSV, TXT | Automate config and report generation |

---

## What Comes Next — Phase 2

You now have the data literacy layer. The next phase is **Engineering and Automation** —
where you turn this knowledge into production-grade services:

- `subprocess` and `os` — call shell commands from Python
- `uv` virtual environments — reproducible, isolated environments
- **Pydantic** — strict data validation for APIs
- **FastAPI** — serve predictions and trigger DevOps tasks over HTTP
- **Pytest** — test your scripts in CI/CD pipelines
- **Asyncio** — handle many requests at the same time

> **Practice before moving on:** Take the final script above and modify it to read your own
> log files. Change the column names to match your data. Add one more chart. That is how
> this knowledge becomes muscle memory.