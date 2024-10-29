# Nashville Housing Data Cleaning Project

## Project Overview
This project demonstrates data cleaning techniques applied to the Nashville Housing dataset using SQL. The goal is to standardize, fill missing values, split fields, and remove duplicates to ensure the dataset is consistent, reliable, and ready for further analysis.

## Dataset
The dataset used in this project is stored in the `NashvilleHousing` table within the `PortfolioProject` database. It contains housing information such as property addresses, sale dates, sale prices, and other details relevant to housing transactions in Nashville.

## Project Structure
The project is structured as follows:

1. **Standardize Date Formats**
2. **Fill Missing Property Address Data**
3. **Break Down Addresses into Individual Columns**
4. **Convert Boolean Values**
5. **Remove Duplicate Records**
6. **Drop Unused Columns**

---

## Steps and Code

### 1. Standardize Date Formats
Convert inconsistent `SaleDate` values to a standard `YYYY-MM-DD` format and, if necessary, add a new `SaleDateConverted` column to store the converted dates.
```sql
ALTER TABLE NashvilleHousing ADD SaleDateConverted Date;

UPDATE NashvilleHousing
SET SaleDateConverted = CONVERT(Date, SaleDate);
```

### 2. Fill Missing Property Address Data
Populate missing `PropertyAddress` values by using other records with the same `ParcelID`.
```sql
UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM NashvilleHousing a
JOIN NashvilleHousing b ON a.ParcelID = b.ParcelID AND a.[UniqueID] <> b.[UniqueID]
WHERE a.PropertyAddress IS NULL;
```

### 3. Break Down Addresses into Individual Columns
Separate `PropertyAddress` into individual columns for address, city, and state.
```sql
ALTER TABLE NashvilleHousing ADD PropertySplitAddress NVARCHAR(255);
ALTER TABLE NashvilleHousing ADD PropertySplitCity NVARCHAR(255);
ALTER TABLE NashvilleHousing ADD PropertySplitState NVARCHAR(255);

UPDATE NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) -1);
```

### 4. Convert Boolean Values in "Sold as Vacant" Field
Convert `Y/N` values in the `SoldAsVacant` column to `Yes/No`.
```sql
UPDATE NashvilleHousing
SET SoldAsVacant = CASE
    WHEN SoldAsVacant = 'Y' THEN 'Yes'
    WHEN SoldAsVacant = 'N' THEN 'No'
    ELSE SoldAsVacant
END;
```

### 5. Remove Duplicate Records
Identify and delete duplicate records based on unique identifiers, retaining the first occurrence of each.
```sql
WITH RowNumCTE AS (
    SELECT *, ROW_NUMBER() OVER (
        PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference 
        ORDER BY UniqueID
    ) AS row_num
    FROM NashvilleHousing
)
DELETE FROM RowNumCTE
WHERE row_num > 1;
```

### 6. Drop Unused Columns
Remove columns that are no longer necessary for analysis.
```sql
ALTER TABLE NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate;
```

---

## Additional Notes

### Data Import Options
For data import, `BULK INSERT` and `OPENROWSET` provide efficient methods but require specific server configuration:
- **BULK INSERT** imports from CSV.
- **OPENROWSET** enables importing from Excel files.

Ensure that the server permissions and configurations are set for ad-hoc queries if you choose to use these methods.

---

## Requirements
- SQL Server
- SQL Server Management Studio (SSMS)
- Permission to configure `OPENROWSET` or `BULK INSERT` options (if importing data)

## Conclusion
This project demonstrates best practices in SQL for data cleaning. By applying these techniques, the Nashville Housing dataset is structured, cleaned, and ready for deeper data analysis. Feel free to use, adapt, or build upon this workflow for other datasets.

---

## Author
- Mohamed, a Business Analytics student working on enhancing data-processing skills.

---