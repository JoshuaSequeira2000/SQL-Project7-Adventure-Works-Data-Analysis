# SQL-Project 7- Exploratory Data Analysis - Adventure Works Database Analysis

## Complete code attached - AdventureWorks Database Analysis.sql

## Data Insights Using SQL.

#### 1) Stored procedure to find the number of employees along with the employee details working under each shift.
```
create proc Employee_Shift_Details(@ShiftID tinyint, @Output int output) -- ShiftID value to be passed, output used to store the ShiftID value passed.
as 
begin
	if exists(select * from [HumanResources].[Shift] where @ShiftID = ShiftID) -- Checks IF the ShiftID passed exists in the database. If not, then Else condition will be triggered.
		begin
		    -- Stores employee details.
			select edh.BusinessEntityID, p.FirstName, p.LastName from [HumanResources].[Shift] s
			inner join [HumanResources].[EmployeeDepartmentHistory] edh on s.ShiftID = edh.ShiftID
			inner join [Person].[Person] p on edh.BusinessEntityID = p.BusinessEntityID
			where s.ShiftID = @ShiftID and edh.EndDate is null -- Filters results based on the ShiftID value passed. Also filtered for employees who are currently working in the shift by using EndDate as NULL.

			-- Stores number of employees working each shift.
			select count(edh.BusinessEntityID) as Number_Of_Employees from [HumanResources].[Shift] s -- Count used to find the number of employees for the particular SHiftID value passed.
			inner join [HumanResources].[EmployeeDepartmentHistory] edh on s.ShiftID = edh.ShiftID
			inner join [Person].[Person] p on edh.BusinessEntityID = p.BusinessEntityID
			where s.ShiftID = @ShiftID and edh.EndDate is null -- Filters results based on the ShiftID value passed. Also filtered for employees who are currently working in the shift by using EndDate as NULL.

			-- Stores the shift ID.
			select @Output = ShiftID from [HumanResources].[Shift] -- OUTPUT used to store the ShiftID value passed.
			where @ShiftID = ShiftID
			return 0 -- Used to identity successful result.
		end
	else 
		begin
			select 'Shift ID does not exist' as ERROR -- If shiftID which does not exist in the database is passed, then an Error message will be displayed.
			set @Output = NULL -- Output will be set to NULL once error message is displayed.
			return 1 -- Used to identitfy failed result.
		end
end
GO

-- Executing the stored procedure.
declare @Output as int, @ReturnStatus as int -- declaring output and return status as variables.
execute @ReturnStatus = Employee_Shift_Details 3, @Output output
select @Output as ShiftID, @ReturnStatus as ReturnStatus
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/7aebc3b6-b352-476c-9d15-369ec76f31b6)

#### 2) Stored procedure to check sales target and sales achieved for each sales employees. Also rank employees based on total sales achieved.
```
create proc Employee_Sales_Achieved(@BusinessEntityID int, @Output int output) -- BusinessEntityID to be passed. Output used to store the BusinessEntityID.
as
begin
	if exists(select * from [Sales].[SalesPerson] where BusinessEntityID = @BusinessEntityID) -- Checks if the BusinessEntityID passed exists in the database. If not, then Else condition will be triggered.
		begin
			-- Storing the sales person details.
			select BusinessEntityID, FirstName as First_Name, LastName as Last_Name
			from [Person].[Person] p 
			where BusinessEntityID = @BusinessEntityID -- Filter used to get information of the sales person based on BusinessEntityID passed.
			
			-- Storing sales target, salesYTD, extra sales achived for the year and whether this years sales were greater than last years sales. 
			select SalesQuota as Sales_Target, SalesYTD,
			case when (SalesYTD - SalesQuota) is null then 0.00 else (SalesYTD - SalesQuota) end as Extra_Sales_Achieved, -- stores the difference between the SalesYTD and SalesQuota to give the extra sales achieved for the year.
			case when SalesYTD > SalesLastYear and SalesLastYear != 0.00 then 'Higher sales achieved than last year' -- Only considers employees who have a history of previous years sales.
				 when SalesYTD > SalesLastYear and SalesLastYear = 0.00 then 'New Employee' -- Only considers employees who do not have any previous year sales.
				 when SalesYTD < SalesLastYear then 'Lower sales achieved than last year' end as Sales_Status
			from [Sales].[SalesPerson]
			where BusinessEntityID = @BusinessEntityID

			-- Storing 'Yes' and 'No' values whether sales target for the year have been achieved or not.
			select case when SalesYTD > SalesQuota then 'Yes'
			            when SalesYTD > isnull(SalesQuota, 1) and SalesQuota is null then 'No target assigned for employee' -- Checks if SalesYTD is greater than SalesQuota. Since Sales quota is null, ISNULL function is used to get the output where SalesQuota is null.
						when SalesYTD < SalesQuota then 'No' end as Is_Sales_Target_Achieved 
			from [Sales].[SalesPerson] 
			where BusinessEntityID = @BusinessEntityID

			-- Storing the rank of employees based on SalesYTD.
			select @Output = tbl1.rnk from
			(select BusinessEntityID ,DENSE_RANK() over(order by SalesYTD desc) as Rnk from [Sales].[SalesPerson]) as tbl1 -- derived table used to store the rank of the employees based on SalesYTD.
			where tbl1.BusinessEntityID = @BusinessEntityID
			return 0 -- Used to identity successful result.
		end
	else
		begin
			select 'Business Entity ID does not exisit' as Error
			set @Output = NULL
			return 1 -- Used to identity failed result.
		end	
end
GO

-- Executing the stored procedure.
declare @output int, @ReturnStatus int -- -- declaring output and return status as variables.
execute @ReturnStatus = Employee_Sales_Achieved 283, @output output
select @output as Sales_Achieved_Rank, @ReturnStatus as Return_Status
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/ca49b192-117d-431c-9a93-34e504f7120d)

#### 3) Stored procedure to update the employee's hire information.
```
create proc Update_Employee_Title_Pay_Details
(@BusinessEntityID int, @JobTitle varchar(50), @RateChangeDate datetime, @Rate money) as -- Stores the necessary information needed to update employee hire information.
	begin
		if exists(select * from [HumanResources].[Employee] where BusinessEntityID = @BusinessEntityID) -- Checks if the BusinessEntityID exists in the database.
			begin
				-- Updating JobTitle from employee table.
				update [HumanResources].[Employee] 
				set JobTitle = @JobTitle -- Sets the job title based on the value passed.
				output inserted.* -- Displays the new records inserted.
				from [HumanResources].[Employee]
				where BusinessEntityID = @BusinessEntityID -- Sets the BusinessEntityID based on the value passed.

				-- Updating RateChangeDate and Rate value from EmployeePayHistory table.
				update [HumanResources].[EmployeePayHistory]
				set RateChangeDate = @RateChangeDate, Rate = @Rate -- Sets the RateChangeDate and the Rate based on the value passed.
				output inserted.* -- Displays the new records inserted.
				from [HumanResources].[EmployeePayHistory]
				where BusinessEntityID = @BusinessEntityID
			end
		else
			begin
				select 'Invalid BusinessEntityID' as Error -- Else condition which gets triggered when the IF statement fails.
			end
	end
GO

-- Executing the stored procedure.
execute Update_Employee_Title_Pay_Details 290, 'Sales Representative Lead', '2023-07-09 00:00:00.000', 25
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/b1fb1de4-9f35-439f-ab80-2cee9b231b20)

#### 4) Stored procedure to update the employee login information. 
```
create proc Update_Employee_Login_Info(@BusinessEntityID int, @LoginID nvarchar(50)) as -- Stores the necessary information needed to update the employee login information.
begin
	if exists(select * from [HumanResources].[Employee] where BusinessEntityID = @BusinessEntityID)
		begin
			update [HumanResources].[Employee]
			set LoginID = @LoginID -- Sets the =LoginID title based on the value passed.
			output inserted.*, deleted.* -- Displays the newly added LoginID and the deleted LoginID
			from [HumanResources].[Employee]
			where BusinessEntityID = @BusinessEntityID
		end
	else
		begin
			select 'Invalid BusinessEntityID' as Error -- ELSE condition is triggered if the IF condition fails.
		end
end
GO

-- Executing the stored procedure.
exec Update_Employee_Login_Info 290, 'adventure-works\ranjit1'
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/cb57d4cf-c2d6-49b5-926d-4d27487a8d5a)

-- 5) Stored procedure to update the employee's personal information.
```
create proc Update_Employee_Personal_Info
(@BusinessEntityID int, @NationalIDNumber int, @BirthDate date, @MaritalStatus nchar(1), @Gender nchar(1)) as -- Stores the necessary information needed to update employee personal information.
begin
	if exists(select * from [HumanResources].[Employee] where BusinessEntityID = @BusinessEntityID) -- Checks if the BusinessEntityID exists within the database. If it does not, then the Else condition gets triggered.
		begin
			update [HumanResources].[Employee]
			set NationalIDNumber = @NationalIDNumber, BirthDate = @BirthDate, MaritalStatus = @MaritalStatus,
				Gender = @Gender
			output inserted.*  -- Displays the newly added Details.
			from [HumanResources].[Employee]
			where BusinessEntityID = @BusinessEntityID
		end
	else
		begin
			select 'Invalid BusinessEntityID' as Error -- ELSE condition is triggered if the IF condition fails.
		end
end
GO

-- Executing the stored procedure.
execute Update_Employee_Personal_Info 290, 961931711, '1975-06-20', 'M', 'M'
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/d4d05e88-8057-460c-a1f7-6126eb0da6c4)

#### 6) Stored procedure to update the available VacationHours and SickLeaveHours for employees.
```
create proc Update_VacationHours_SickLeaveHours
(@BusinessEntityID int, @VacationHours smallint, @SickLeaveHours smallint) as -- Stores the necessary information needed to update employee personal information.
begin
	if exists(select * from [HumanResources].[Employee] where BusinessEntityID = @BusinessEntityID) -- Checks if the BusinessEntityID exists within the database. If it does not, then the Else condition gets triggered.
		begin
			update [HumanResources].[Employee]
			set VacationHours = @VacationHours, SickLeaveHours = @SickLeaveHours
			output inserted.* -- Displays the newly added details.
			from [HumanResources].[Employee]
			where BusinessEntityID = @BusinessEntityID
		end
	else
		begin
			select 'Invalid BusinessEntityID' as Error -- ELSE condition is triggered if the IF condition fails.
		end
end
GO

-- Executing the stored procedure.
execute Update_VacationHours_SickLeaveHours 290, 26, 26
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/a9684fa5-aca9-465d-b98f-acdb501a8141)

#### 7) Stored procedure to insert new records inside the table [Person].[Person].
```
create proc Insert_Into_PersonPerson
(@BusinessEntityID int, @PersonType nchar(2), @NameStyle bit, @title nvarchar(8), @FirstName nvarchar(50),
 @MiddleName nvarchar(50), @LastName nvarchar(50), @Suffix nvarchar(10), @EmailPromotion int) as
begin
	insert into [Person].[Person]([BusinessEntityID], [PersonType], [NameStyle], [Title], [FirstName], [MiddleName],
								  [LastName], [Suffix], [EmailPromotion])
	values(@BusinessEntityID, @PersonType, @NameStyle, @title, @FirstName, @MiddleName, @LastName, @Suffix, 
	       @EmailPromotion)

	select * from [Person].[Person]
	where BusinessEntityID = @BusinessEntityID -- Select statement to view the newly inserted record based on BusinessEntityID.
end
GO

-- Executing stored procedure
execute Insert_Into_PersonPerson 292, 'EM', 0, 'Mr.', 'Joshua', null, 'Sequeira', null, 2
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/57c3ec4f-8e64-438d-9f53-6cf95c524d10)

#### Views
#### 8) View to return all the employee's personal details and addresses.
```
create view Employee_Personal_Info as
select e.BusinessEntityID, e.HireDate, e.JobTitle, p.Title, p.FirstName, p.MiddleName, p.LastName, e.BirthDate, e.Gender, 
	   e.MaritalStatus, pp.PhoneNumber, pnt.Name as Phone_Number_Type, a.AddressLine1, a.AddressLine2, 
	   addt.Name as Address_Type, a.City, a.PostalCode, sp.StateProvinceCode, sp.Name as State_Name, 
	   sp.CountryRegionCode, cr.Name as Country
from [HumanResources].[Employee] e 
left join [Person].[Person] p on e.BusinessEntityID = p.BusinessEntityID
left join [Person].[BusinessEntityAddress] bea on p.BusinessEntityID = bea.BusinessEntityID
left join [Person].[Address] a on bea.AddressID = a.AddressID
left join [Person].[AddressType] addt on bea.AddressTypeID = addt.AddressTypeID
left join [Person].[StateProvince] sp on a.StateProvinceID = sp.StateProvinceID
left join [Person].[CountryRegion] cr on sp.CountryRegionCode = cr.CountryRegionCode
left join [Person].[PersonPhone] pp on e.BusinessEntityID = pp.BusinessEntityID
left join [Person].[PhoneNumberType] pnt on pp.PhoneNumberTypeID = pnt.PhoneNumberTypeID
order by e.BusinessEntityID offset 0 rows -- Offset is used to skip rows to be shown in the output. Since OFFSET 0 is used, all rows are shown in the order specified in order by clause.
GO

-- Executing the view.
select * from Employee_Personal_Info
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/64acd741-e3f0-4737-b299-176330557467)

#### 9) View to return the employee's department history.
```
create view Employee_Department_History as
select e.BusinessEntityID, p.Title, p.FirstName, p.MiddleName, p.LastName, p.Suffix, e.JobTitle, 
       d.Name as Department_Name, d.GroupName, edh.StartDate, edh.EndDate
from [HumanResources].[Employee] e
inner join [HumanResources].[EmployeeDepartmentHistory] edh on e.BusinessEntityID = edh.BusinessEntityID
inner join [HumanResources].[Department] d on d.DepartmentID = edh.DepartmentID
inner join [Person].[Person] p on e.BusinessEntityID = p.BusinessEntityID
inner join [HumanResources].[Shift] s on edh.ShiftID = s.ShiftID
order by e.BusinessEntityID offset 0 rows
GO

-- Executing the view.
select * from Employee_Department_History
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/e1ebef08-77cc-4b5b-a447-28b417d163c3)

#### 10) The total number of Male and Female employees, along with the total number of single males, single females, married males, and married females.
```
select case when Gender = 'F' then 'Female' end as EmployeeGenderAndStatus, -- code to find number of female employees.
COUNT(BusinessEntityID) as NumberOfEmployees -- 
from [HumanResources].[Employee] 
where Gender = 'F' 
group by Gender
UNION ALL
select case when Gender = 'M' then 'Male' end, COUNT(BusinessEntityID) -- code to find number of male employees.
from [HumanResources].[Employee] 
where Gender = 'M' 
group by Gender
UNION ALL
select case when MaritalStatus = 'S' then 'Single Male' end, COUNT(BusinessEntityID) -- code to find number of single male employees
from [HumanResources].[Employee] 
where MaritalStatus = 'S' and Gender = 'M' 
group by MaritalStatus
UNION ALL
select case when MaritalStatus = 'M' then 'Married Male' end, COUNT(BusinessEntityID) -- code to find number of married male employees.
from [HumanResources].[Employee] 
where MaritalStatus = 'M' and Gender = 'M' 
group by MaritalStatus
UNION ALL
select case when MaritalStatus = 'S' then 'Single Female' end, COUNT(BusinessEntityID) -- code to find number of single female employees.
from [HumanResources].[Employee] 
where MaritalStatus = 'S' and Gender = 'F' 
group by MaritalStatus
UNION ALL -- Union all used to merge all the tables together regardless if the columns match or not.
select case when MaritalStatus = 'M' then 'Married Female' end, COUNT(BusinessEntityID) -- code to find number of married female employees
from [HumanResources].[Employee] 
where MaritalStatus = 'M' and Gender = 'F' 
group by MaritalStatus
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/0429016e-1c99-4825-8752-0f8d847af867)

#### 11) All the departments having < 10 employees.
```
with DepartmentEmployeeCount as
(select d.Name as Department_Name, COUNT(e.BusinessEntityID) as Number_Of_Employees 
from [HumanResources].[Employee] e
left join [HumanResources].[EmployeeDepartmentHistory] edh on e.BusinessEntityID = edh.BusinessEntityID
left join [HumanResources].[Department] d on edh.DepartmentID = d.DepartmentID
where edh.EndDate is null -- Filtering for employees currently working in the department.
group by d.Name) -- temporary table to store the number of employees in each department.
select * from DepartmentEmployeeCount
where Number_Of_Employees < 10 -- Filtering departments having less than 10 employees.
order by Department_Name
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/791b06e0-b225-4e05-944d-babff04bf068)


#### 12) Number of employees working under each job title.
```
select JobTitle, COUNT(BusinessEntityID) as Number_Of_Employees 
from [HumanResources].[Employee]
group by JobTitle
order by JobTitle  asc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/a7bd7ec7-12f1-4d2a-ad55-161b04196588)

### 13) Find the most senior employees under each job title.
```
with cte as
	(select e.JobTitle, p.FirstName, p.LastName, e.Gender, e.BirthDate,
		   DENSE_RANK() over(partition by e.JobTitle order by e.BirthDate) as DnsRnk
	from [HumanResources].[Employee] e
	inner join [Person].[Person] p on e.BusinessEntityID = p.BusinessEntityID)
select JobTitle, FirstName, LastName, Gender, BirthDate from cte
where DnsRnk = 1
order by JobTitle
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/780e4119-8faf-471c-9bb1-0c10b15402cc)

#### 14) Average, Minimum and Maximum salary for each job title under each department.
```
select d.Name as Department_Name, e.JobTitle as Job_Title, MIN(eph.Rate) as Min_Salary, -- Function to find the minimum salary.
														   MAX(eph.Rate) as Max_Salary, -- Function to find the maximum salary.
														   AVG(eph.Rate) as Avg_Salary  -- Function to find the average salary.
from [HumanResources].[Department] d
inner join [HumanResources].[EmployeeDepartmentHistory] edh on d.DepartmentID = edh.BusinessEntityID
inner join [HumanResources].[Employee] e on edh.BusinessEntityID = e.BusinessEntityID
inner join [HumanResources].[EmployeePayHistory] eph on e.BusinessEntityID = eph.BusinessEntityID
group by d.Name, e.JobTitle
order by d.Name
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/e94eab15-6f42-40a8-9097-af660a328527)


#### 15) Total number of employees currently working in each department.
```
select case when d.Name is null then 'TOTAL' else d.Name end Name, count(dh.BusinessEntityID) as NumberOfEmployees
from [HumanResources].[EmployeeDepartmentHistory] dh
inner join [HumanResources].[Department] d on dh.DepartmentID = d.DepartmentID
where dh.EndDate is null
group by rollup(d.Name)
order by NumberOfEmployees asc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/6bf1f5d9-96b0-44b5-a736-2c0b8cb9a42c)

### 16) Using a pivot table to find the number of employees hired each month for all the years.
```
with cte as
	(select BusinessEntityID, year(HireDate) as HireYear, month(HireDate) as TheMonth  from [HumanResources].[Employee])
select HireYear, [1] as January, [2] as February, [3] as March, [4] as April, [5] as May, [6] as June, 
				 [7] as July, [8] as August, [9] as September, [10] as October, [11] as November, [12] as December from cte
pivot(count(BusinessEntityID) for TheMonth in ([1], [2], [3], [4], [5], [6], [7], [8], [9], [10], [11], [12])) NewPivot
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/bc6c59d1-f28b-49a9-9f4f-dcb3ba22cc27)

-- 17) Using a pivot table to find the number of employees under each sales YTD range based on TerritoryID.
```
with cte as
			(select distinct sp.BusinessEntityID, cr.name,
			case when SalesYTD between 0 and 100000 then 1 when SalesYTD between 100001 and 2000000 then 2
				 when SalesYTD between 2000001 and 3500000 then 3 when SalesYTD between 3500001 and 5000000 then 4
					when SalesYTD >= 5000001 then 5 end SalesRange from [Sales].[SalesPerson] sp 
			inner join [Person].[Person] p on sp.BusinessEntityID = p.BusinessEntityID
			inner join [HumanResources].[EmployeeDepartmentHistory] edh on sp.BusinessEntityID = edh.BusinessEntityID
			inner join [HumanResources].[Department] d on edh.DepartmentID = d.DepartmentID
			inner join [Person].[StateProvince] stp on sp.TerritoryID = stp.TerritoryID
			inner join [Person].[CountryRegion] cr on stp.CountryRegionCode = cr.CountryRegionCode)
select name as Country, [1] as LowestSales, [2] as BelowAverageSales, [3] as AverageSales, [4] as AboveAverageSales, 
             [5] as HighestSales from cte
pivot(count(BusinessEntityID) for SalesRange in ([1], [2], [3], [4], [5])) pvt 
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/cefdf80e-ddec-4e19-a536-0dfde260b6c6)

#### Using Functions. Scalar Function.
#### 1) Function to find the number of employees in each department.
```
create function NumberOfEmployeesInEachDepartment1(@DepartmentID smallint) -- User input value to be passed.
returns smallint -- Scalar function returns a single value.
as
begin
			declare @NumberOfEmployeesInEachDepartment int -- Declaring a varibale.
			select @NumberOfEmployeesInEachDepartment = count(ed.BusinessEntityID) -- Assigning this variable a value.
			from [HumanResources].[EmployeeDepartmentHistory] ed
			inner join [HumanResources].[Department] d on ed.DepartmentID = d.DepartmentID
			where ed.EndDate is null and ed.DepartmentID = @DepartmentID
			group by d.DepartmentID, d.name
			return @NumberOfEmployeesInEachDepartment -- Returning the variable.
end
GO

declare @var int
exec @var = [dbo].[NumberOfEmployeesInEachDepartment1] 4
select @var as NumberOfEmployees
GO

select *, [dbo].[NumberOfEmployeesInEachDepartment1](4) as NumberOfEmployees from [HumanResources].[Department] -- using the function in a select statement.
where DepartmentID = 4
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/ef01df78-6fcc-494c-a445-7f3003154f3a)
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/4675cc59-73ec-48d1-92bf-6a09b3bbe865)

#### Using Functions. Inline Table Functions
#### 2) Find the number of employees based on the shiftID value passed.
```
create function EmployeeShiftCount(@ShiftID tinyint)
returns table as return -- Inline table functions returns a table.
(
select s.name as ShiftName, count(ed.BusinessEntityID) as NoOfEmployees from [HumanResources].[Shift] s
inner join [HumanResources].[EmployeeDepartmentHistory] ed on s.ShiftID = ed.ShiftID
where s.ShiftID = @ShiftID
group by s.name
)
GO

select * from dbo.EmployeeShiftCount(1)
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/c23f3fc0-50ac-414a-a6f5-d56848916f13)

#### 3) Find a list of employees based on the shiftID value passed.
```
create function EmployeeShiftEmployees(@ShiftID tinyint)
returns table as return -- Inline table function returns a table.
(
select s.name as ShiftName, p.FirstName, p.MiddleName, p.LastName from [HumanResources].[Shift] s
inner join [HumanResources].[EmployeeDepartmentHistory] ed on s.ShiftID = ed.ShiftID
inner join [Person].[Person] p on ed.BusinessEntityID = p.BusinessEntityID
where s.ShiftID = @ShiftID
group by s.name, p.FirstName, p.MiddleName, p.LastName
)
GO

select * from EmployeeShiftEmployees(1)
select * from EmployeeShiftEmployees(2)
select * from EmployeeShiftEmployees(3)
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/b7590f61-bcb7-4d97-ac2f-7aab87d92553)

#### 18) Finding the average time it takes for an order to be shipped after it has been ordered.
```
select avg(DATEDIFF(DD, OrderDate, ShipDate)) as Avg_Days_To_Ship  -- Datediff function used to specify the difference in the number of days between OrderDate and ShipDate.
from [Sales].[SalesOrderHeader]
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/e7fbeb06-2720-4ced-9b0a-58130b18ac89)

#### 19) Total number of orders placed in each month and year.
```
with Year_Month_Orders as -- CTE used to order the table based on Month Number.
	(select DATENAME(YEAR, OrderDate) as OrderYear, DATEPART(MONTH, orderdate) as OrderMonthNumber,  
		   DATENAME(MONTH, OrderDate) as OrderMonth, -- Datename function used to get the year and month from OrderDate.
		   COUNT(SalesOrderID) as NumberOfOrders
	from [Sales].[SalesOrderHeader]
	group by DATENAME(YEAR, OrderDate),  DATENAME(MONTH, OrderDate), DATEPART(MONTH, orderdate))
select OrderYear, OrderMonth, NumberOfOrders 
from Year_Month_Orders
order by OrderYear, OrderMonthNumber
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/a26e9721-811d-4c06-b07e-2b219d13ec07)

#### 20) Total number of orders placed by sales executive Vs. customers online.
```
select case when OnlineOrderFlag = 0 then 'Order placed by sales person'
			when OnlineOrderFlag = 1 then 'Order placed online by customer' end OnlineOrderFlag, 
COUNT(OnlineOrderFlag) as NumberOfOrders
from [Sales].[SalesOrderHeader]
group by OnlineOrderFlag
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/b9b94f0b-08b8-443b-9d51-59d299f75555)

#### 21) Total sales each year for all territories.
```
EXEC sp_rename '[Sales].[SalesTerritory].Group', 'GroupName', 'COLUMN'; -- Changing the column name 'Group' to 'GroupName'.
GO

select t.TerritoryID, t.Name, t.CountryRegionCode, t.GroupName, YEAR(o.OrderDate) as OrderYear, 
format(SUM(o.TotalDue), 'C') as TotalRevenue, -- Formatting the summed value to USD currency.
DENSE_RANK() over(partition by t.TerritoryID order by SUM(o.TotalDue) desc) as RevenueRank
from [Sales].[SalesTerritory] t
inner join [Sales].[SalesOrderHeader] o on t.TerritoryID = o.TerritoryID
group by t.TerritoryID, t.Name, t.CountryRegionCode, t.GroupName, YEAR(OrderDate)
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/93bf0f49-01b5-434e-9441-bca3273e3382)

### 22) Total revenue generated from each store.
```
select tbl1.CustomerID, tbl1.PersonType, tbl1.StoreID, tbl1.StoreName, tbl1.TotalRevenue from
	(select c.CustomerID, p.PersonType ,s.BusinessEntityID as StoreID, s.Name as StoreName, format(SUM(soh.TotalDue), 'C') as TotalRevenue,
	SUM(soh.TotalDue) as TotalRevenue1
	from [Sales].[Customer] c 
	inner join [Sales].[SalesOrderHeader]  soh on c.CustomerID = soh.CustomerID
	inner join [Sales].[Store] s on c.StoreID = s.BusinessEntityID
	inner join [Person].[Person] p on c.PersonID = p.BusinessEntityID
	group by c.CustomerID, p.PersonType, s.BusinessEntityID, s.Name) as tbl1 -- Used derived table to order the TotalRevenue with $ symbol.
order by tbl1.TotalRevenue1 desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/9f624179-76fa-4ce3-b595-8787df42a04a)

#### 23) Total revenue generated from each customer.
```
select tbl1.CustomerID, tbl1.PersonID, tbl1.PersonType, tbl1.Title, tbl1.FirstName, tbl1.MiddleName, tbl1.LastName,
	   tbl1.Suffix, tbl1.TotalRevenue from
(select c.CustomerID, c.PersonID, p.PersonType, p.Title, p.FirstName, P.MiddleName, p.LastName, p.Suffix, 
	   format(SUM(soh.TotalDue), 'C') as TotalRevenue, SUM(soh.TotalDue) as TotalRevenue1
	from [Sales].[Customer] c 
	inner join [Sales].[SalesOrderHeader]  soh on c.CustomerID = soh.CustomerID
	inner join [Person].[Person] p on c.PersonID = p.BusinessEntityID
	group by c.CustomerID, c.PersonID, p.PersonType, p.Title, p.FirstName, P.MiddleName, p.LastName, p.Suffix) as tbl1  -- Used derived table to order the TotalRevenue with $ symbol.
where tbl1.PersonType = 'IN' -- Filtering for only customers
order by tbl1.TotalRevenue1 desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/a3ac2795-be9f-41a8-bb43-f1205673a016)

#### 24) Number of products sold for each sales reason along with the revenue generated for each sales reason.
```
select sr.Name, sr.ReasonType, format(SUM(soh.TotalDue), 'C') as TotalRevenue
from [Sales].[SalesOrderHeader] soh
inner join [Sales].[SalesOrderHeaderSalesReason] sohsr on soh.SalesOrderID = sohsr.SalesOrderID
inner join [Sales].[SalesReason] sr on sohsr.SalesReasonID = sr.SalesReasonID
group by sr.Name, sr.ReasonType
order by sr.Name
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project7-Adventure-Works-Data-Analysis/assets/92262753/c27e6eef-8d05-4c84-ac3e-cad8b4aa96e8)

#### Views
#### 25) Individual customers' details that purchase Adventure Works Cycles products online.
```
create view ViewCustomerDetails as
select p.BusinessEntityID, p.Title, p.FirstName, p.MiddleName, p.LastName, p.Suffix, pp.PhoneNumber, 
	   pnt.Name as PhoneNumberType, ea.EmailAddress, p.EmailPromotion, at.Name as AddressType, a.AddressLine1, 
	   a.AddressLine2, a.City, sp.Name as StateProvinceName, a.PostalCode, cr.Name
from [Person].[Person] p
left join [Sales].[Customer] c on p.BusinessEntityID = c.PersonID
left join [Person].[PersonPhone] pp on p.BusinessEntityID = pp.BusinessEntityID
left join [Person].[PhoneNumberType] pnt on pp.PhoneNumberTypeID = pnt.PhoneNumberTypeID
left join [Person].[EmailAddress] ea on p.BusinessEntityID = ea.BusinessEntityID
left join [Person].[BusinessEntityAddress] bea on p.BusinessEntityID = bea.BusinessEntityID
left join [Person].[Address] a on bea.AddressID = a.AddressID
left join [Person].[AddressType] at on bea.AddressTypeID = at.AddressTypeID
inner join [Person].[StateProvince] sp on a.StateProvinceID = sp.StateProvinceID
inner join [Person].[CountryRegion] cr on sp.CountryRegionCode = cr.CountryRegionCode
where p.PersonType = 'IN'
order by BusinessEntityID offset 0 rows
GO

-- Executing the view.
select * from ViewCustomerDetails
GO
```


#### 26) Sales representatives (names and addresses) and their sales-related information.
```
create view ViewSalesEmployeesDetails as
select p.BusinessEntityID, p.Title, p.FirstName, p.MiddleName, p.LastName, p.Suffix, pp.PhoneNumber, 
	   pnt.Name as PhoneNumberType, ea.EmailAddress, p.EmailPromotion, at.Name as AddressType, a.AddressLine1, 
	   a.AddressLine2, a.City, sp.Name as StateProvinceName, a.PostalCode, cr.Name,
	   st.Name as TerritoryName, st.GroupName as TerritoryGroup, sap.SalesQuota, st.SalesYTD, st.SalesLastYear
from [Person].[Person] p
left join [Sales].[Customer] c on p.BusinessEntityID = c.PersonID
left join [Person].[PersonPhone] pp on p.BusinessEntityID = pp.BusinessEntityID
left join [Person].[PhoneNumberType] pnt on pp.PhoneNumberTypeID = pnt.PhoneNumberTypeID
left join [Person].[EmailAddress] ea on p.BusinessEntityID = ea.BusinessEntityID
left join [Person].[BusinessEntityAddress] bea on p.BusinessEntityID = bea.BusinessEntityID
left join [Person].[Address] a on bea.AddressID = a.AddressID
left join [Person].[AddressType] at on bea.AddressTypeID = at.AddressTypeID
left join [Person].[StateProvince] sp on a.StateProvinceID = sp.StateProvinceID
left join [Person].[CountryRegion] cr on sp.CountryRegionCode = cr.CountryRegionCode
left join [Sales].[SalesTerritory] st on cr.CountryRegionCode = st.CountryRegionCode
left join [Sales].[SalesPerson] sap on sap.BusinessEntityID = p.BusinessEntityID
where p.PersonType = 'SP'
order by BusinessEntityID offset 0 rows
GO

-- Executing the view.
select * from ViewSalesEmployeesDetails
GO
```


#### 27) Using PIVOT to return aggregated sales revenue generated by each sales employee.
```
with Sales_Rep_Data as
	(select sp.BusinessEntityID as SalesPersonID, 
	CONCAT(p.FirstName,' ',p.MiddleName,' ',p.LastName) as FullName, --Concat function to merge first, middle and last name
	e.JobTitle, YEAR(soh.OrderDate) as SaleYear, soh.TotalDue as TotalRevenue
	from [Sales].[SalesPerson] sp 
	inner join [Person].[Person] p on sp.BusinessEntityID = p.BusinessEntityID
	inner join [HumanResources].[Employee] e on p.BusinessEntityID = e.BusinessEntityID
	inner join [Sales].[SalesTerritoryHistory] sth on e.BusinessEntityID = sth.BusinessEntityID
	--inner join [Sales].[SalesTerritory] st on sth.TerritoryID = st.TerritoryID
	inner join [Sales].[SalesOrderHeader] soh on p.BusinessEntityID = soh.SalesPersonID)
select SalesPersonID, FullName, JobTitle, 
		  isnull(FORMAT([2011], 'C'), '$0.00') as '2011', 
		  isnull(FORMAT([2012], 'C'), '$0.00') as '2012',
		  isnull(FORMAT([2013], 'C'), '$0.00') as '2013',
		  isnull(FORMAT([2014], 'C'), '$0.00') as '2014' -- Isnull function to replace Nulls with %0.00 and Format function to get revenue in $
from Sales_Rep_Data
pivot(sum(TotalRevenue) for SaleYear in ([2011], [2012], [2013], [2014])) as pvt
order by SalesPersonID 
GO
```

#### 28) List of all products which are manufactured in-house, along with the sub category name and product category.
```
select p.ProductNumber, pc.name as ProductCategoryName, ps.name as ProductSubCategoryName, p.Name, 
case when p.MakeFlag = 1 then 'Product is manufactured in-house.' end ProductStatus
from [Production].[Product] p
inner join [Production].[ProductSubcategory] ps on p.ProductSubcategoryID = ps.ProductSubcategoryID
inner join [Production].[ProductCategory] pc on ps.ProductCategoryID = pc.ProductCategoryID
where p.MakeFlag = 1
GO
```

#### 29) List of all products which are purchased, along with the sub category name and product category.
```
select p.ProductNumber, pc.name as ProductCategoryName, ps.name as ProductSubCategoryName, p.Name, 
case when p.MakeFlag = 0 then 'Product is purchased' end ProductStatus
from [Production].[Product] p
inner join [Production].[ProductSubcategory] ps on p.ProductSubcategoryID = ps.ProductSubcategoryID
inner join [Production].[ProductCategory] pc on ps.ProductCategoryID = pc.ProductCategoryID
where p.MakeFlag = 0
GO
```

### 30) Total number of products which are purchased Vs those which are manufactured in-house.
```
with cte as 
(select case when MakeFlag = 0 then 'Total Number Of Purchased Products' 
	        when MakeFlag = 1 then 'Total Number Of Products Manufactured In House'
			end ProductStatus,
count(productID) as ProductID
from [Production].[Product]
group by rollup (case when MakeFlag = 0 then 'Total Number Of Purchased Products' 
	        when MakeFlag = 1 then 'Total Number Of Products Manufactured In House'  end))
select case when productStatus is null then 'TOTAL' else ProductStatus end ProductStatus, ProductID
from cte
order by ProductID asc
```


#### 31) List all the product names and product sub categories which come under the product 'Bikes' and mention which of these products are salable and which are not.
```
select p.ProductID, pc.name as ProductCategory, p.Name as ProductName, ps.Name as ProductSubcategoryName, 
	   case when p.FinishedGoodsFlag = 0 then 'Product is not a salable item.'
			when p.FinishedGoodsFlag = 1 then 'Product is salable.' end ProductStatus
from [Production].[Product] p
inner join [Production].[ProductSubcategory] ps on p.ProductSubcategoryID = ps.ProductSubcategoryID
inner join [Production].[ProductCategory] pc on ps.ProductCategoryID = pc.ProductCategoryID
where pc.Name = 'Bikes'
order by p.ProductID
GO
```


#### 32) List all the products names and product subcategories which come under the product 'Components' and mention which of these products are salable and which are not.
```
select p.ProductID, pc.name as ProductCategory, p.Name as ProductName, ps.Name as ProductSubcategoryName, 
	   case when p.FinishedGoodsFlag = 0 then 'Product is not a salable item.'
			when p.FinishedGoodsFlag = 1 then 'Product is salable.' end ProductStatus
from [Production].[Product] p
inner join [Production].[ProductSubcategory] ps on p.ProductSubcategoryID = ps.ProductSubcategoryID
inner join [Production].[ProductCategory] pc on ps.ProductCategoryID = pc.ProductCategoryID
where pc.Name = 'Components'
order by p.ProductID
GO
```

#### 33) List all the products names and product sub categories which come under the product 'Clothing' and mention which of these products are salable and which are not.
```
select p.ProductID, pc.name as ProductCategory, p.Name as ProductName, ps.Name as ProductSubcategoryName, 
	   case when p.FinishedGoodsFlag = 0 then 'Product is not a salable item.'
			when p.FinishedGoodsFlag = 1 then 'Product is salable.' end ProductStatus
from [Production].[Product] p
inner join [Production].[ProductSubcategory] ps on p.ProductSubcategoryID = ps.ProductSubcategoryID
inner join [Production].[ProductCategory] pc on ps.ProductCategoryID = pc.ProductCategoryID
where pc.Name = 'Clothing'
order by p.ProductID
GO
```

#### 34) List all the products names and product sub categories which come under the product 'Accessories' and mention which of these products are salable and which are not.
```
select p.ProductID, pc.name as ProductCategory, p.Name as ProductName, ps.Name as ProductSubcategoryName, 
	   case when p.FinishedGoodsFlag = 0 then 'Product is not a salable item.'
			when p.FinishedGoodsFlag = 1 then 'Product is salable.' end ProductStatus
from [Production].[Product] p
inner join [Production].[ProductSubcategory] ps on p.ProductSubcategoryID = ps.ProductSubcategoryID
inner join [Production].[ProductCategory] pc on ps.ProductCategoryID = pc.ProductCategoryID
where pc.Name = 'Accessories'
order by p.ProductID
GO
```

##### 35) Analysis of product cost and price. Using data only from FinishedGoodsFlag = 1. Considering the cost of the product, find the profit which will be made for each product. Also generate a table with products with the top 10 most profitable products based on rank.
```
with cte as
	(select p.ProductID, pc.Name as CategoryName, ps.Name as ProductSubCategoryName, p.name as ProductName,
		   (p.ListPrice - p.StandardCost) ProfitPerProduct, 
		   dense_rank() over(order by (p.ListPrice - p.StandardCost) desc) as Rank -- ordering based on the profit per product in descending.
	from [Production].[Product] p
	inner join [Production].[ProductSubcategory] ps on p.ProductSubcategoryID = ps.ProductSubcategoryID
	inner join [Production].[ProductCategory] pc on ps.ProductCategoryID = pc.ProductCategoryID
	where p.FinishedGoodsFlag = 1)
select *
from cte
where Rank between 1 and 10
order by Rank
```

#### 36) List of all mountain products for women which are still on sale.
```
select p.ProductID, pc.Name as CategoryName, ps.Name as SubCategoryName, p.Name as ProductName
	from [Production].[Product] p
		inner join [Production].[ProductSubcategory] ps on p.ProductSubcategoryID = ps.ProductSubcategoryID
		inner join [Production].[ProductCategory] pc on ps.ProductCategoryID = pc.ProductCategoryID
	where p.ProductLine = 'M' and p.Style = 'W' and p.SellEndDate is null
order by ProductID
GO
```

#### Using Functions. Multi-Statement Table Functions
#### 1) Using multi-statement table function to find the products which are purchased vs those which are manufactured in-house.
```
create function Purchased0Manufactured1(@MakeFlag bit) -- BIT data type used which stores 0's and 1's.
returns @Purchased0Manufactured1 table -- Multi table function to return table.
(ProductID int, Name nvarchar(50), MakeFlag bit)
as
begin 
		insert into @Purchased0Manufactured1 -- Inserting values into function to be returned.
		select ProductID, Name, MakeFlag from [Production].[Product]
		where MakeFlag = @MakeFlag
		return
end
GO

select * from dbo.Purchased0Manufactured1(1)
GO
```

#### 37) Total quantity of products ordered Vs quantity actually delivered from vendors.
```
select p.ProductID, p.Name, count(pod.OrderQty) as QuantityOrdered, count(pod.ReceivedQty) as QuantityReceived,
count(pod.OrderQty) - count(pod.ReceivedQty) as Difference
from [Production].[Product] p
left join [Purchasing].[PurchaseOrderDetail] pod on p.ProductID = pod.ProductID
group by p.ProductID, p.Name
order by QuantityOrdered desc
GO
```

#### 38) Average amount spent to purchase products from vendors
```
select avg(TotalDue) as AvgCartValue
from [Purchasing].[PurchaseOrderHeader]
GO
```

#### 39) List all vendors with creditrating 1, preferredVendorStatus 1, from USA and Canada
```
select v.BusinessEntityID, v.Name, cr.CountryRegionCode, cr.Name, v.CreditRating, v.PreferredVendorStatus
from [Purchasing].[Vendor] v
inner join [Person].[BusinessEntityAddress] bea on v.BusinessEntityID = bea.BusinessEntityID
inner join [Person].[Address] a on bea.AddressID = a.AddressID
inner join [Person].[StateProvince] sp on a.StateprovinceID = sp.StateProvinceID
inner join [Person].[CountryRegion] cr on sp.CountryRegionCode = cr.CountryRegionCode
where cr.CountryRegionCode in ('CA', 'US') and
      v.CreditRating = 1 and v.PreferredVendorStatus = 1
order by v.BusinessEntityID
GO
```


#### 40) List total number of vendors from each country.
```
select cr.Name, count(v.BusinessEntityID) as totalNumberOfVendors
from [Purchasing].[Vendor] v
inner join [Person].[BusinessEntityAddress] bea on v.BusinessEntityID = bea.BusinessEntityID
inner join [Person].[Address] a on bea.AddressID = a.AddressID
inner join [Person].[StateProvince] sp on a.StateprovinceID = sp.StateProvinceID
inner join [Person].[CountryRegion] cr on sp.CountryRegionCode = cr.CountryRegionCode
group by cr.Name
order by totalNumberOfVendors desc
GO
```


#### 41) Which shipping companies made the most number of shipments from 2012 to 2013.
```
select sm.Name, count(poh.PurchaseOrderID) TotalNumberOfShipments
from [Purchasing].[PurchaseOrderHeader] poh
inner join [Purchasing].[ShipMethod] sm on poh.ShipmethodID = sm.ShipMethodID
where poh.ShipDate between '2012-01-01 00:00:00.000' and '2013-01-01 00:00:00.000'
group by sm.Name
order by TotalNumberOfShipments
GO
```
