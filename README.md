select Address, city
From PortfolioProject..HousingData


--Removing time from saledate

select SaleDate
From PortfolioProject..HousingData

Update PortfolioProject..HousingData
Set SaleDate = Convert(date,saledate)

Alter Table PortfolioProject..HousingData
Add SaleDateOnly Date;

Update PortfolioProject..HousingData
Set Saledateonly = Convert(date,saledate)

select SaleDateOnly
From PortfolioProject..HousingData

--Seperating Property adress from city

select PropertyAddress
From PortfolioProject..HousingData


--Some addresses are NULL that can be filled by the parcelID as same parcelID leads to same address

select a.ParcelID, a.PropertyAddress, b.ParcelID, b. PropertyAddress, ISNULL(a.PropertyAddress, b. PropertyAddress)
From PortfolioProject..HousingData as a
JOIN PortfolioProject..HousingData as b
ON a.ParcelID = b.ParcelID
AND a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is NULL


Update a
SET PropertyAddress = ISNULL(a.PropertyAddress, b. PropertyAddress)
From PortfolioProject..HousingData as a
JOIN PortfolioProject..HousingData as b
ON a.ParcelID = b.ParcelID
AND a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is NULL

--Going Back to the task at hand

Select Substring(PropertyAddress, 1,CHARINDEX(',', PropertyAddress)-1) as Address,
Substring(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress)) as City
From PortfolioProject..HousingData

Alter Table PortfolioProject..HousingData
Add Address nvarchar(255);

Update PortfolioProject..HousingData
Set Address = Substring(PropertyAddress, 1,CHARINDEX(',', PropertyAddress)-1)

Alter Table PortfolioProject..HousingData
Add City nvarchar(255);

Update PortfolioProject..HousingData
Set City = Substring(PropertyAddress, CHARINDEX(',', PropertyAddress)+1, LEN(PropertyAddress))

--Seperating Owner Address from City and State

Select owneraddress
From PortfolioProject..HousingData

Select 
PARSENAME(Replace(Owneraddress, ',','.'),3) as HomeAdress,
PARSENAME(Replace(Owneraddress, ',','.'),2) as CityName,
PARSENAME(Replace(Owneraddress, ',','.'),1) as StateName
From PortfolioProject..HousingData

Alter Table PortfolioProject..HousingData
Add OwnerHomeAddress nvarchar(255);

Update PortfolioProject..HousingData
Set OwnerHomeAddress = PARSENAME(Replace(Owneraddress, ',','.'),3)

Alter Table PortfolioProject..HousingData
Add Owner_City nvarchar(255);

Update PortfolioProject..HousingData
Set Owner_City = PARSENAME(Replace(Owneraddress, ',','.'),2)

Alter Table PortfolioProject..HousingData
Add Owner_State nvarchar(255);

Update PortfolioProject..HousingData
Set Owner_state = PARSENAME(Replace(Owneraddress, ',','.'),1)

--Changing Y and N to Yes and No to make all data same 

Select SoldasVacant, Case
When SoldAsVacant = 'Y' Then 'Yes'
When SoldAsVacant = 'N' Then 'No'
ELSE SoldAsVacant
End
From PortfolioProject..HousingData


Update PortfolioProject..HousingData
Set SoldasVacant =  Case
When SoldAsVacant = 'Y' Then 'Yes'
When SoldAsVacant = 'N' Then 'No'
ELSE SoldAsVacant
End
From PortfolioProject..HousingData 

--Removing Duplicates

With RowNUMCTE AS (
Select *, 
	ROW_Number() over (
	Partition by ParcelID, PropertyAddress, 
	             SalePrice, SaleDate,
				 LegalReference
				 Order by 
				 UniqueID) rownum
From PortfolioProject..HousingData

)

Select *
From RownumCTE
where rownum > 1 

--REmoving Unimportant Columns

ALter Table PortfolioProject..HousingData
Drop Column SaleDate
