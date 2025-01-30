# Data Cleaning in SQL Project
---
## Project Overview

This project is designed to demonstrate SQL skills and techniques typically used by data analysts to explore and clean data. The project involved setting up a database, creating a table, importing data from an excel worksheet, and cleaning the data.

## Objectives

1. **Set up a PortfolioProject database**: Create and populate the database with the provided data from the excel file.
2. **Data Cleaning**: Identify and remove any records with missing or null values and check if any duplicates exist and remove them where necessary.

## Project Structure
---
### 1. Database Setup

- **Database Creation**: The project starts by creating a database named PortfolioProject.
- **Table Creation**: A table named `NashvilleHousing` is created to store the housing data. The table structure includes columns for UniqueID, ParcelID, LandUse, PropertyAddress, SaleDate, SalePrice etc. 

```sql
CREATE TABLE NashvilleHousing (
    UniqueID INT PRIMARY KEY,
    ParcelID VARCHAR(100),
    LandUse VARCHAR(100),
    PropertyAddress TEXT,
    SaleDate VARCHAR(50),
    SalePrice VARCHAR(50),
    LegalReference VARCHAR(100),
    SoldAsVacant VARCHAR(10),
    OwnerName TEXT,
    OwnerAddress TEXT,
    Acreage FLOAT,
    TaxDistrict VARCHAR(100),
    LandValue FLOAT,
    BuildingValue FLOAT,
    TotalValue FLOAT,
    YearBuilt FLOAT,
    Bedrooms FLOAT,
    FullBath FLOAT,
    HalfBath FLOAT
);

```
### 2. Data Exploration & Cleaning

- **Row Count**: Determine the total number of records in the dataset.
- **Distinct Row Count**: Find out how many unique rows are in the dataset.

The following SQL queries were used to clean the data

**STEP 1: Standardize Date Format in SaleDate Column**

```sql
UPDATE NashvilleHousing
SET SaleDate = TO_CHAR(TO_DATE(SaleDate::TEXT,'Month DD, YYYY'), 'YYYY-MM-DD');

```


**STEP 2: Populate Property Address data**

```sql
SELECT *
FROM PortfolioProject.NashvilleHousing
--WHERE PropertyAddress IS NULL
ORDER BY ParcelID;

SELECT 
    a.ParcelID, 
    a.PropertyAddress, 
    b.ParcelID, 
    b.PropertyAddress, 
    COALESCE(a.PropertyAddress, b.PropertyAddress)
FROM NashvilleHousing a
JOIN NashvilleHousing b
    ON a.ParcelID = b.ParcelID
    AND a.UniqueID <> b.UniqueID
WHERE a.PropertyAddress IS NULL;

UPDATE NashvilleHousing a
SET PropertyAddress = COALESCE(a.PropertyAddress, b.PropertyAddress)
FROM NashvilleHousing b
WHERE a.ParcelID = b.ParcelID
  AND a.UniqueID <> b.UniqueID
  AND a.PropertyAddress IS NULL;

```

**STEP 3: Breaking out Address into Individual Columns (Address, City, State)**

```sql
ALTER TABLE NashvilleHousing
ADD PropertySplitAddress VARCHAR(255);

UPDATE NashvilleHousing
SET PropertySplitAddress = SPLIT_PART(PropertyAddress, ',', 1);

ALTER TABLE NashvilleHousing
ADD PropertySplitCity VARCHAR(255);

UPDATE NashvilleHousing
SET PropertySplitCity = TRIM(SPLIT_PART(PropertyAddress, ',', 2));


ALTER TABLE NashvilleHousing
ADD OwnerSplitAddress VARCHAR(255);

UPDATE NashvilleHousing
SET OwnerSplitAddress = SPLIT_PART(OwnerAddress, ',', 1);

ALTER TABLE NashvilleHousing
ADD OwnerSplitCity VARCHAR(255);

UPDATE NashvilleHousing
SET OwnerSplitCity = SPLIT_PART(OwnerAddress, ',', 2);

ALTER TABLE NashvilleHousing
ADD OwnerSplitState VARCHAR(255);

UPDATE NashvilleHousing
SET OwnerSplitState = SPLIT_PART(OwnerAddress, ',', 3);

```

**STEP 4: Change Y and N to Yes and No in "Sold as Vacant" field**
```sql
UPDATE NashvilleHousing
SET SoldAsVacant = CASE 
    WHEN SoldAsVacant = 'Y' THEN 'Yes'
    WHEN SoldAsVacant = 'N' THEN 'No'
    ELSE SoldAsVacant
    END;

```
**STEP 5: Remove Duplicates**
```sql
WITH RowNumCTE AS (
    SELECT uniqueid,
        ROW_NUMBER() OVER (
            PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference
            ORDER BY UniqueID
        ) AS row_num
    FROM NashvilleHousing
),
RowNumCTE2 AS
(SELECT *
FROM RowNumCTE
WHERE row_num > 1)

DELETE FROM nashvillehousing
WHERE uniqueid IN (SELECT uniqueid FROM RowNumCTE2)
```

**STEP 6: Delete Unused Columns**
```sql
ALTER TABLE NashvilleHousing
DROP COLUMN OwnerAddress;

ALTER TABLE NashvilleHousing
DROP COLUMN TaxDistrict; 

ALTER TABLE NashvilleHousing
DROP COLUMN PropertyAddress; 
```
