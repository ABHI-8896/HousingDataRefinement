# SQL Data Cleaning Scripts

This repository contains SQL scripts used for cleaning and standardizing data in the `NashvilleHousing` table from the `PortfolioProject` database.

## Overview

The scripts perform the following tasks:
1. Standardize date formats
2. Populate missing property addresses
3. Break out address into individual columns
4. Clean and split owner address
5. Change 'Y' and 'N' to 'Yes' and 'No' in the "Sold as Vacant" field
6. Remove duplicates
7. Delete unused columns
8. Import data using `OPENROWSET` and `BULK INSERT`

## Table of Contents

1. [Standardize Date Format](#standardize-date-format)
2. [Populate Property Address Data](#populate-property-address-data)
3. [Break Out Address into Individual Columns](#break-out-address-into-individual-columns)
4. [Clean and Split Owner Address](#clean-and-split-owner-address)
5. [Change 'Y' and 'N' to 'Yes' and 'No'](#change-y-and-n-to-yes-and-no)
6. [Remove Duplicates](#remove-duplicates)
7. [Delete Unused Columns](#delete-unused-columns)
8. [Import Data](#import-data)
## How It Works

The SQL scripts provided in this repository are designed to clean and standardize data in the `NashvilleHousing` table from the `PortfolioProject` database. Here’s a step-by-step overview of how the scripts work:

1. **Standardize Date Format:**
   - The script converts the `SaleDate` column to a consistent `Date` format and handles cases where the conversion might fail by creating a new column `SaleDateConverted`.

    ```sql
    SELECT saleDateConverted, CONVERT(Date, SaleDate)
    FROM PortfolioProject.dbo.NashvilleHousing;

    UPDATE NashvilleHousing
    SET SaleDate = CONVERT(Date, SaleDate);
    
    ALTER TABLE NashvilleHousing
    ADD SaleDateConverted Date;

    UPDATE NashvilleHousing
    SET SaleDateConverted = CONVERT(Date, SaleDate);
    ```

2. **Populate Property Address Data:**
   - This part of the script populates missing property addresses by joining records with the same `ParcelID` but different `UniqueID` values and updating the missing `PropertyAddress` values.

    ```sql
    SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, ISNULL(a.PropertyAddress, b.PropertyAddress)
    FROM PortfolioProject.dbo.NashvilleHousing a
    JOIN PortfolioProject.dbo.NashvilleHousing b
        ON a.ParcelID = b.ParcelID
        AND a.[UniqueID] <> b.[UniqueID]
    WHERE a.PropertyAddress IS NULL;

    UPDATE a
    SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
    FROM PortfolioProject.dbo.NashvilleHousing a
    JOIN PortfolioProject.dbo.NashvilleHousing b
        ON a.ParcelID = b.ParcelID
        AND a.[UniqueID] <> b.[UniqueID]
    WHERE a.PropertyAddress IS NULL;
    ```

3. **Break Out Address into Individual Columns:**
   - This script extracts address components from a combined `PropertyAddress` column into separate columns for address and city.

    ```sql
    SELECT
        SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1) AS Address,
        SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress)) AS Address
    FROM PortfolioProject.dbo.NashvilleHousing;

    ALTER TABLE NashvilleHousing
    ADD PropertySplitAddress Nvarchar(255);

    UPDATE NashvilleHousing
    SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1);

    ALTER TABLE NashvilleHousing
    ADD PropertySplitCity Nvarchar(255);

    UPDATE NashvilleHousing
    SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress));
    ```

4. **Clean and Split Owner Address:**
   - This part cleans and splits the `OwnerAddress` into separate columns for address, city, and state.

    ```sql
    SELECT
        PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3) AS OwnerSplitAddress,
        PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2) AS OwnerSplitCity,
        PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1) AS OwnerSplitState
    FROM PortfolioProject.dbo.NashvilleHousing;

    ALTER TABLE NashvilleHousing
    ADD OwnerSplitAddress Nvarchar(255);

    UPDATE NashvilleHousing
    SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3);

    ALTER TABLE NashvilleHousing
    ADD OwnerSplitCity Nvarchar(255);

    UPDATE NashvilleHousing
    SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2);

    ALTER TABLE NashvilleHousing
    ADD OwnerSplitState Nvarchar(255);

    UPDATE NashvilleHousing
    SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1);
    ```

5. **Change 'Y' and 'N' to 'Yes' and 'No':**
   - This script updates the `SoldAsVacant` field to replace 'Y' with 'Yes' and 'N' with 'No'.

    ```sql
    UPDATE NashvilleHousing
    SET SoldAsVacant = CASE 
        WHEN SoldAsVacant = 'Y' THEN 'Yes'
        WHEN SoldAsVacant = 'N' THEN 'No'
        ELSE SoldAsVacant
    END;
    ```

6. **Remove Duplicates:**
   - Identifies and removes duplicate records based on key fields such as `ParcelID`, `PropertyAddress`, `SalePrice`, `SaleDate`, and `LegalReference`.

    ```sql
    WITH RowNumCTE AS (
        SELECT *,
            ROW_NUMBER() OVER (
                PARTITION BY ParcelID,
                             PropertyAddress,
                             SalePrice,
                             SaleDate,
                             LegalReference
                ORDER BY UniqueID
            ) AS row_num
        FROM PortfolioProject.dbo.NashvilleHousing
    )
    SELECT *
    FROM RowNumCTE
    WHERE row_num > 1
    ORDER BY PropertyAddress;
    ```

7. **Delete Unused Columns:**
   - Removes columns that are no longer needed from the `NashvilleHousing` table.

    ```sql
    ALTER TABLE PortfolioProject.dbo.NashvilleHousing
    DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate;
    ```

8. **Import Data:**
   - Provides examples of importing data into the `NashvilleHousing` table using `BULK INSERT` and `OPENROWSET`.

    ```sql
    -- Using BULK INSERT
    -- BULK INSERT nashvilleHousing
    -- FROM "D:\Nashville Housing Data.xlsx"
    -- WITH (
    --    FIELDTERMINATOR = ',',
    --    ROWTERMINATOR = '\n'
    --);

    -- Using OPENROWSET
    -- SELECT * INTO nashvilleHousing
    -- FROM OPENROWSET('Microsoft.ACE.OLEDB.12.0',
    --    'Excel 12.0; Database="D:\Nashville Housing Data.xlsx", [Sheet1$]');
    ```

## Summary

This repository contains SQL scripts for data cleaning and standardization of the `NashvilleHousing` dataset within the `PortfolioProject` database. The scripts address several key data quality issues:

1. **Date Format Standardization:** Converts and standardizes date formats in the `SaleDate` column to ensure consistency across the dataset.

2. **Address Data Population:** Updates missing `PropertyAddress` values by cross-referencing records with the same `ParcelID`, and splits combined addresses into individual components for better analysis.

3. **Owner Address Cleanup:** Splits the `OwnerAddress` field into separate columns for address, city, and state, improving the granularity of address data.

4. **Value Standardization:** Converts 'Y' and 'N' values in the `SoldAsVacant` field to 'Yes' and 'No' for clearer data interpretation.

5. **Duplicate Removal:** Identifies and removes duplicate records based on key attributes to ensure the dataset contains unique entries.

6. **Column Removal:** Drops unused columns from the dataset to streamline data structure and improve query performance.

7. **Data Import:** Provides examples of importing data into the `NashvilleHousing` table using `BULK INSERT` and `OPENROWSET` methods, which may require server configuration.

These scripts help maintain data integrity, improve data quality, and facilitate more accurate analyses of the housing data. They are essential for preparing the dataset for further processing and analysis.



