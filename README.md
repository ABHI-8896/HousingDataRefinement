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



