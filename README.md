# kqlcheatsheet
I would say the following notes are good for someone who's starting in KQL

# HOW TO USE KQL
> [SQL to Kusto Query Language cheat sheet](https://learn.microsoft.com/en-us/kusto/query/sql-cheat-sheet?view=azure-data-explorer)
1. When using KQL, every query starts with a tabular input, and every operator produces a tabular output

- EXAMPLE BELOW

```SQL
select top (10) * form StormEvents
where state = 'ALASKA'
order by StartTime desc
```
> SAME AS
```KQL
StormEvents
| where State == 'ALASKA'
| top 10 by StartTime
```

- Additionally KQL queries can be crafted from the UI, but it is not the most efficient way [[MustLearnKQL_Book.pdf#page=43|MustLearnKQL_Book, p.43]]
![image](https://github.com/user-attachments/assets/50952fba-7328-491c-b20f-47da838f13da)


# THE WORKFLOW

![image](https://github.com/user-attachments/assets/210d34bb-592f-4ef9-9930-a588bff5c434)


> IN THE FOLLOWING EXAMPLE THE WORKFLOW IS BEING EXPLAINED. VERY IMPORTANT TO LEARN THIS WORKFLOW LINE BY LINE. THE IMPORTANCE IS IN THE RESULTS > [[MustLearnKQL_Book.pdf#page=14|MustLearnKQL_Book, p.14]]
HOWEVER; the workflow is also explained here below, so you have 2 explanations ^^

1. Identify the table we want to query to.
2. Use a pipe `|` in order to separate the commands. Commands can go in a single line; HOWEVER, using them in different lines will allow the query to be more visual.
3. Data filtering:
	1. The were operator is one of the bests to accomplish this function
		1. By time generated
		2. By windows event
4. Data Aggregation:
	1. With the filtered data, in this example it is created a count of accounts (usernames) that produced a successful login
5. Order the results with the `order` operator
6. Use the `project operator` to indicate the exact that that you want to be displayed

# OPERATORS
> AVOID COPYING AND PASTING CONTENT. TYPE THE COMMANDS MANUALLY AND UNDERSTAND WHAT YOU ARE DOING

## Search Operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/search-operator?view=azure-data-explorer&preserve-view=true&pivots=azuredataexplorer)
- When doing a search in order to find if something exists, it is usefull following the checklist below

1. Does it exist? 
2. Where does it exist? 
3. Why does it exist? 
4. BONUS: There’s a final question to this that’s not part of this KQL series, but one that’s important to the total equation and one that should be part of your SOC processes. That question is: How do we respond?
![image](https://github.com/user-attachments/assets/eb561327-5e95-47f6-9473-416a17b8fd25)

> Explained with further details in [[MustLearnKQL_Book.pdf#page=18|MustLearnKQL_Book, p.18]]

## Top operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/top-operator?view=azure-data-explorer&preserve-view=true)

- Used to show only the first entries of the output (head command in Linux)

- Example below

```KQL
SecurityEvent //the table
| top 100 by TimeGenerated desc //Retrieving the top 100 records sorted in descending order by TimeGenerated
```
## Where operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/where-operator?view=azure-data-explorer&preserve-view=true)
> Filtering the data is the key to everything
- This is the most powerful operator for KQL
![image](https://github.com/user-attachments/assets/899107b3-5423-411c-850f-737664a35169)
Explanation in [[MustLearnKQL_Book.pdf#page=21|MustLearnKQL_Book, p.21]]

### Allowable predicates: 

• String predicates: , has, contains, startswith, endswith, matches regex, etc 
• Numeric/Date predicates: , !=, <, >, <=, >=
• Empty predicates: isempty(), notempty(), isnull(), notnull()
> [Best Practices DOC](https://learn.microsoft.com/en-us/kusto/query/best-practices?view=azure-data-explorer&preserve-view=true&WT.mc_id=m365-0000-rotrent)
## getschema operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/getschema-operator?view=azure-data-explorer&preserve-view=true)
- Produces the list of all columns and their types.

## limit and take operators
> The `take` and `limit` operators are equivalent
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/take-operator?view=azure-data-explorer&preserve-view=true)
[[MustLearnKQL_Book.pdf#page=52|MustLearnKQL_Book, p.52]]
### Syntax

```KQL
Tablename
| limit <number> 

-or

Tablename
| take <number>

```

## count operator

### Syntax

```KQL
Tablename
| count

OR

SecurityEvent
| count

```

![image](https://github.com/user-attachments/assets/9bf4daaa-adf8-45a4-9309-096eef0b3da4)

[[MustLearnKQL_Book.pdf#page=56|MustLearnKQL_Book, p.56]]

## Summarize operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/summarize-operator?view=azure-data-explorer&preserve-view=true)
 - Produces a table that aggregates the content of the input table.

## Render operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/render-operator?view=azure-data-explorer&preserve-view=true&pivots=azuredataexplorer)

- Render tells the query engine that you want to take the data you've supplied, and show it in any of the following ways (visualizations)
	- [[MustLearnKQL_Book.pdf#page=62|MustLearnKQL_Book, p.62]]

### Syntax

```KQL
Tablenanme
| render visualization

// JUST THE DATA

SecurityEvent //The table
| where TimeGenerated > ago(7d) //Looking at data in the last 7 days
| summarize count() by bin(TimeGenerated, 1d) //Using Bin to group the data by each day

// THE DATA AS A BARCHART

SecurityEvent //The table
| where TimeGenerated > ago(7d) //Looking at data in the last 7 days
| summarize count() by bin(TimeGenerated, 1d) //Uding Bin to group the data by each day
| render barchart //Looking at the data in a Barchart

```
![image](https://github.com/user-attachments/assets/9b312677-d6ed-4107-8c0d-fa2a29eb6716)

[[MustLearnKQL_Book.pdf#page=64|MustLearnKQL_Book, p.64]]
> See more examples in page 65...

## Extend operator
> DOCS

- Allows us to build custom columns in real-time in the query results. It allows you to create calculated columns and append them to the results. Understand, though, that we're note creating columns of data that are stored back into the data table, but only generating a column of custom data based on our current request

- EXAMPLE
```KQL
SecurityEvent //the table
| extend ComputerNameLenght = strlen(Computer) //creates a new column called ComputerNameLength of the calculation of the number of characters of the computer name in the Computer column
```

### Syntax

```KQL
TableName
| extend[ColumnName | (ColumnName[,...])=] Expression [,...]
```

- See the following example 

![image](https://github.com/user-attachments/assets/0538f565-8192-4647-af88-1c1a7d9ac728)


- See another example to create columns with custom values

```KQL
SecurityEvent
| extend My_Calculation = 8*8
| extend My_Fabricated_Data = "Yay for me!"
```
![image](https://github.com/user-attachments/assets/c96df8b6-5568-443e-beb2-d38e2c64e6c0)
[[MustLearnKQL_Book.pdf#page=69|MustLearnKQL_Book, p.69]]

- A practical usage example of this can be calculating a specific value. In the following example the perf table finds free disk space on the recorded systems and display it in kilobytes, megabytes, and gigabytes

```KQL
Perf //table name
| where CounterName == "Free Megabytes" //filtering data by 'free megabytes'
| extend FreeKB = CounterValue * 1024 //calculating free kilobytes
| extend FreeGB = CounterValue / 1024 //calculating free gigabytes
| extend FreeMB = CounterValue //calculating free megabytes
```
![image](https://github.com/user-attachments/assets/70a1e007-e97a-459b-bbb3-88623f3d59ca)
[[MustLearnKQL_Book.pdf#page=70|MustLearnKQL_Book, p.70]]

## Project operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/project-operator?view=azure-data-explorer&preserve-view=true)

- Indicate the query engine which columns to show 

## Project-away operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/project-away-operator?view=azure-data-explorer&preserve-view=true)

- Select what columns from the input to exclude from the output
	- If from a big amount of columns you want only to exclude a few ones, this operator is useful


## Project-keep operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/project-keep-operator?view=azure-data-explorer&preserve-view=true)

- Select what columns from the input to keep in the output.
	- Hint: Just use the standard Project option.

## Project-rename operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/project-rename-operator?view=azure-data-explorer&preserve-view=true)

- Renames columns in the result output.

- Example below

```KQL
Tablename
| project-rename my_new_column_name = old_column_name
```

## Project-reorder operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/project-reorder-operator?view=azure-data-explorer&preserve-view=true)

- Reorders the columns in the result output.

- Please see the example

```KQL
Tablename
| project-reorder column2, column3, column1
```

## Distinct operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/distinct-operator?view=azure-data-explorer&preserve-view=true)

- Used to show only distinct values of a column (avoid repeated values)

![image](https://github.com/user-attachments/assets/c2acfe2d-7fcc-4593-abd2-62cfa6dd70c2)

- More than one column can be used with this operator. Please see the example below

```KQL
UpdateSummary //the table 
| distinct Computer, WindowsUpdateSetting //We can get our Windows Update Settings for all servers we’re managing with the Update Management solution
```

## The UNION operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/union-operator?view=azure-data-explorer&preserve-view=true&pivots=azuredataexplorer)

- Union allows you to take the data from two or more tables and display the results (all rows from all tables) together
	- Combine data from tables in a different way than JOIN

- Example below

```KQL
SecurityEvent //the table
| union Heartbeat //merging SecurityEvent table with the Heartbeat table
| summarize count() by Computer //showing all computers from both tables and how many times
```

- This next query example is the same as before but merges with an additional table (SecurityAlert) to show the data from three tables instead of two

```KQL
SecurityEvent //the table
| union Heartbeat, SecurityAlert //merging SecurityEvent table with the Heartbeat table and SecurityAlert
| summarize count() by Computer //showing all computers from all tables and how many times they are referenced
```

- The following example introduces a couple changes to the first two queries in that it merges all tables that start with ‘Sec’ (notice the wildcard character) and sorts the computers in alphabetical ascending order (the last line).

```KQL
SecurityEvent //the table
| union Sec* //merging together all tables beginning with 'Sec'
| summarize count () by Computer //showing all computers from all tables and how many times they are referenced
| sort by Computer asc //displaying Computer names in ascending order
```

[[MustLearnKQL_Book.pdf#page=85|MustLearnKQL_Book, p.85]]

## The JOIN operator
> [DOCS](https://learn.microsoft.com/en-us/kusto/query/join-operator?view=azure-data-explorer&preserve-view=true)

- Join, on the other hand, is intended to produce more specific results by joining rows of just two tables through matching the values of columns you specify.
	- Combine data from tables in a different way than UNION

![image](https://github.com/user-attachments/assets/f9492c48-b9bc-49fa-b9e0-3411052f171f)

- See examples below

```KQL
LeftTable
| join [JoinParameters] (RightTable) onAttributes
```

- The following example joins together the SecurityEvent and Heartbeat tables on the common Computer column. It then filters all Computers by the 4688 Event ID (newly spawned process) and shows the Computer name and the installed OS and versioning.

```KQL
SecurityEvent //table name
| join Heartbeat on Computer //joining SecurityEvent with Heartbeat on the common Computer column
| where EventID == "4688" //Looking for Event ID for new process
| project Computer, OSType, OSMajorVersion, Version //displaying data from both tables
```

![image](https://github.com/user-attachments/assets/e8a94835-e49d-4842-b165-213a5178a48a)
[[MustLearnKQL_Book.pdf#page=88|MustLearnKQL_Book, p.88]]

# Azure Activity Command
> [DOCS](https://learn.microsoft.com/es-es/azure/azure-monitor/reference/tables/azureactivity)

- Use this as a table to view possible activities within the azure cloud shell activity

- Example below

```KQL
AzureActivity //the table - this is where Cloud Shell activity is logged
| where ResourceGroup startswith "CLOUD-SHELL" //filtering for Cloud Shell
| where ResourceProviderValue == "MICROSOFT.STORAGE" //to not mistake this for some other cloudf shell operation, also filtering on MICROSOFT.STORAGE. Storage is created anytime Cloud Shell runs
| where ActivityStatusValue == "Start" //making sure that the activity is the spawning of a new Cloud Shell instance
| summarize count() by TimeGenerated, ResourceGroup, Caller, CallerIpAddress, ActivityStatusValue //getting a count of how many times each individual has run Cloud Shell
| extend AccountCustomEntity = Caller //Assigning the Caller column - name of person - to AccountCustomEntity -  this is what is used for the User Entity in Microsoft Sentinel Incidents
| extend IPCustomEntity = CallerIpAddress //assigning the CallerIpAddress column - IP address of user's system - to IPCustomEntity - this is what is used for the IP Entity in Microsoft Sentinel Incidents
```
> [OFFICIAL DOCS](https://learn.microsoft.com/es-es/azure/azure-monitor/reference/tables/azureactivity) to create your analytics rule in Microsoft Sentinel


# Let statement

- It allows you to create variables. This makes sense to a lot of folks who script or program and it’s not dissimilar.
![image](https://github.com/user-attachments/assets/5dcef013-068c-4614-af23-ca75cf82a801)
[[MustLearnKQL_Book.pdf#page=82|MustLearnKQL_Book, p.82]]

## Creating variables from scratch

- Please see the example below. See the refference here
	- The `timeOffset` is used to create a time range of between 7 and 14 days in which to look at data
	- The `discardEventID` is used to show everything BUT 4688 in the results

```KQL
let timeOffset = 7d; //setting the offset variable
let discardEventId = 4688; //assigning new process as the event ID
SecurityEvent //the table
| where TimeGenerated > ago(timeOffset*2) and TimeGenerated < ago(timeOffset) //setting a specific time range of between 7 and 14 days
| where EventID != discardEventId //showing all events but 4688
```

## Creating variables from existing data

- Used to pull data from existing tables

- Example below

```KQL
let login = SecurityEvent //Setting the login variable based on a full query
| where TimeGenerated > ago(1h) //look at records in the last hour
| where EventID == '4624' //setting the event ID to successful login
| project Account, TargetLogonId, loginTime = TimeGenerated; //creating the full output, notice the semicolon to end the let statement
let logout //SecurityEvent //setting the logout variable based on a full query
| where TimeGenerated > ago(1h) //look at records in the last hour
| where EventID == '4634' //setting the event ID to successful logoff
| project Account, TargetLogonId, logoutTime = TimeGenerated; //creating the full output, notice the semicolon to end the let statement
login //accessing the login output
| join kind=leftouter logout on TargetLogonId //joining login output with logout output
| project Account, loginTime, logoutTime //showing login and logout times for each account
```

## Creating variables from Microsoft Sentinel Watchlists
> [DOCS](https://learn.microsoft.com/en-us/azure/sentinel/watchlists)
- The let statement enables you to use the Watchlist feature with their Analytics Rules

- Example below
```KQL
//Watchlist as a variable, where the requested data is in the list
let wathlist = (_GetWatchlist('FeodoTracker')| project DstIP);
Heartbeat
| where ComputerIP in (watchlist)

//Watchlist as a variable, where the requested data is not in the list
let watchlist = (_GetWatchlist('FeodoTracker')| project DstIP);
Heartbeat
| where ComputerIP !in (watchlist)

//Watchlist inline with the quer, where the requested data is in the list
Heartbeat
| where ComputerIP in ((_GetWatchlist('FeodoTracker')| project DstIP)
)

//Watchlist inline with the query, where the requested data isnot in the list Heartbeat
| where ComputerIP !in ((_GetWatchlist('FeodoTracker')| project DstIP)
)


```


# Finding IoCs

KQL queries samples in order to find IoCs based on these notes can be the following:

- Search for Malicious IP Addresses:

```
SecurityEvent
| where IpAddress in ("185.234.217.58", "104.28.10.33")
| project TimeGenerated, Computer, IpAddress, AccountName, EventID
```

- Search for Malicious Domains in CommandLine Logs:

```
SecurityEvent
| where CommandLine has_any ("malicious-domain.com", "evil-site.net")
| project TimeGenerated, Computer, CommandLine, AccountName
```

- Search for Malicious File Hashes:

```
SecurityEvent
| where Hash in ("b0a8fe5b1c739f97cd6abbd18e531b9d3eb84f5f")
| project TimeGenerated, Computer, Hash, AccountName, FileName
```

- Search for Malicious URLs in Network Logs:

```
SecurityEvent
| where CommandLine has_any ("http://bad-url.com/malware")
| project TimeGenerated, Computer, CommandLine, AccountName
```

- Lastly, these queries may be combined

```
SecurityEvent
| where IpAddress in ("185.234.217.58", "104.28.10.33")
or CommandLine has_any ("malicious-domain.com", "evil-site.net", "http://bad-url.com/malware")
or Hash in ("b0a8fe5b1c739f97cd6abbd18e531b9d3eb84f5f")
| project TimeGenerated, Computer, IpAddress, CommandLine, Hash, AccountName
```

