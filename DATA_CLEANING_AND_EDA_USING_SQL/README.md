# Laptop Dataset — Data Cleaning & EDA (MySQL)

## Files
| File | Description |
|---|---|
| `cleaning.txt` | Full data cleaning pipeline |
| `eda.txt` | Exploratory data analysis |

---

## Dataset
Raw laptop data imported as `laptopdata` into the `laptops` database via CSV.
~1300 rows, 12 original columns.

---

## laptops_cleaning.sql

### 1. Index Column
- Dropped the auto-generated `Unnamed: 0` column from CSV import
- Added a new `index` column at first position
- Populated with unique sequential values using a MySQL variable `@row`

### 2. Drop Nulls
- Deleted all rows where any column contained a NULL value
- Verified with a COUNT check (should return 0)

### 3. Drop Duplicates
- Added a temporary `uid` AUTO_INCREMENT column as a truly unique key
- Deleted duplicate rows keeping the first occurrence (lowest uid)
- Dropped the `uid` helper column after cleanup
- Verified no duplicates remain

### 4. Column Cleaning

| Column | What was done |
|---|---|
| Ram | Removed `GB` text, converted to INT |
| Weight | Removed `kg` text, set `?` to NULL, converted to DECIMAL |
| Price | Rounded to integer |
| OpSys | Categorised into: Windows, macOS, Linux, Chrome OS, Android, No OS |
| Gpu | Categorised into brands: Intel, Nvidia, AMD, ARM, Other |
| Memory | Split into 3 new columns: `storage_type`, `primary_storage`, `secondary_storage`. TB values converted to GB. Original column dropped |
| Cpu | Split into 3 new columns: `cpu_brand`, `cpu_model`, `cpu_speed`. Original column dropped |
| ScreenResolution | Split into 4 new columns: `touchscreen` (0/1), `resolution_width`, `resolution_height`, `screen_type`. Original column dropped |

### 5. Data Types & Column Order
All columns reordered and given appropriate data types in a single `ALTER TABLE`:

| Column | Type |
|---|---|
| index | INT PRIMARY KEY |
| Company | VARCHAR(50) |
| TypeName | VARCHAR(50) |
| Inches | DECIMAL(4,1) |
| screen_type | VARCHAR(20) |
| resolution_width | SMALLINT UNSIGNED |
| resolution_height | SMALLINT UNSIGNED |
| touchscreen | TINYINT(1) |
| cpu_brand | VARCHAR(10) |
| cpu_model | VARCHAR(20) |
| cpu_speed | DECIMAL(3,1) |
| Ram | TINYINT UNSIGNED |
| Gpu | VARCHAR(20) |
| storage_type | VARCHAR(10) |
| primary_storage | SMALLINT UNSIGNED |
| secondary_storage | SMALLINT UNSIGNED |
| OpSys | VARCHAR(20) |
| Weight | DECIMAL(4,2) |
| Price | FLOAT |

---

## laptops_eda.sql

### 1. Head, Tail, Sample
- First 5 rows, last 5 rows, random 5 rows

### 2. Numerical Column Analysis — Price
- 5 number summary: count, min, max, mean, std
- Missing value count
- Outlier detection using IQR method (ROW_NUMBER based quartiles)
- Horizontal histogram in price buckets of 10,000

### 3. Categorical Column Analysis — Company
- Value counts with percentage share
- Missing value count
- Bar simulation (ASCII)

### 4. Numerical - Numerical — Price vs Ram
- Side by side 5 number summary
- Scatterplot simulation (avg price per RAM value)
- Pearson correlation coefficient

### 5. Categorical - Categorical — Company vs Gpu
- Contingency table (cross tab)
- Stacked bar chart simulation

### 6. Numerical - Categorical — Price vs Company
- Min, max, mean, std of Price grouped by Company

### 8. Missing Value Treatment
- Null count per every column
- Weight nulls filled with median
- storage_type nulls filled with mode

### 9. Feature Engineering
| Feature | Formula |
|---|---|
| `ppi` | `SQRT(width² + height²) / Inches` |
| `price_bracket` | Budget (<25k), Mid (<50k), High (<100k), Premium (100k+) |

### 10. One Hot Encoding
Encoded the following categorical columns into binary (0/1) columns:

| Column | Categories encoded |
|---|---|
| Company | Dell, Lenovo, HP, Asus, Apple, Acer, MSI, Toshiba, Other |
| TypeName | Notebook, Gaming, Ultrabook, 2in1, Workstation, Netbook |
| Gpu | Intel, Nvidia, AMD, ARM, Other |
| OpSys | Windows, macOS, Linux, Chrome OS, Android, No OS |
| cpu_brand | Intel, AMD, Samsung, Other |
| screen_type | IPS Full HD, IPS 4K, IPS Quad HD, IPS Retina, IPS, Full HD, 4K, Quad HD, Standard |
| storage_type | SSD, HDD, Hybrid, Other |
