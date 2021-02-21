---
layout: post
title: Flake it till you make it
subtitle: Excerpt from Soulshaping by Jeff Brown
cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [books, test]
---

### Introduction

Micrsoft Excel aka Excel, is probably that one tool which is used heavily by everyone in the business world. Of course, there is email, text editors, etc. but I am omitting the basic necessities here. Regardless of the size of a business or the job profile, everyone is using it to some extent.

![art_drawn_in_excel](/assets/img/excel-1/paintin_excel.JPG)

Image credits:[Tatsuo Horiuchi](http://www2.odn.ne.jp/~cbl97790/)

I first met Excel briefly at my structural design internship. As the title implies, I was not doing any heavy number-crunching or business modelling but it still helped by re-using my knowledge of how different parameters affect the final design outputs and designing multiple similar structures without using the calculator everytime. As with every new thing that I do, I sucked at it, and was not even scratching the surface of what Excel had to offer. Fast-forward 1.5 years, I got a job as a *Business Analyst* at a boutique consulting firm. From that day onwards, I have loved Excel, understood its limitations, tried to push its boundaries several times, and used it to the extent that I think in Excel now. Not sure if that is a thing but yes, I do.

> The fundamental problem that Excel solves with its simplicity is **Allowing people to program a logic & re-use it without writing any code**
 
It was this simple need, for people knowing their businesses being able to create their own custom reusable logical templates without hiring any programmers or prior coding knowledge, is what made Excel so popular. This requirement exists even today, and in my opinion, would continue to exist in the foreseeable future.

Coming to the agenda of writing this, I wanted to have a one-stop destination which I can refer back to whenever I need to refresh my memory on any particular topic. Along with that, I hope this could act as a helpful reference to other new/experienced users out there. Lastly, while there are numerous other resources out there which might be much more detailed in nature, I want to cover topics that I have found useful and wished I had been provided as a cheatsheet when I started learning.

This would be a series of posts as I have observed preference for bite-sized content amongst the readers these days (including me). Without further ado, let's get started!

### Part 1: Getting the data in
Excel offers a wide variety of options to fetch the data you need for analysis. Let's go through them one-by-one to make sure we know what all is possible:
1. **Direct Pasting**: As the name suggests, you can just copy any form of tabular or non-tabular (using [Text to Columns](update link here)) and begin your analysis. While this is the most simple & straightforward approach, the main drawback is having to do all the manual labor again when your source data changes and you want a refreshed version. You will need to remember exactly what parameters or filters you had used while fetching the original data.  ![Data tab in ribbon](/assets/img/excel-1/data_tab.JPG)
2. **Using Excel's Get Data Module**: If you navigate to `Data` tab from the top ribbon in Excel (2016 & 365 versions), you will find a section on `Get & Transform Data`. In here, you can basically instruct Excel about the source file path, which tables to pick, any transformations required (more on that later), etc. and Excel will remember it for you as a *Query/Connection*. Therefore, whenever you want a refreshed version of this data you just have to click `Refresh All` from the `Queries & Connections` Section right next to it. As you might have guessed, this options is always preferable over option 1 due to the following reasons:
> Excel uses `Power Query` as the underlying engine when you import data using `Get Data` module. This will allow you to relate that Microsoft PowerBI and Excel are using a common approach to data pull, making your learning curve for PowerBI less steeper

   - **Ability to refresh**: As I just mentioned, it allows single click refresh for your data. Additionally, you can also setup periodic auto-refresh (daily/hourly) or triggered auto-refresh (opening the file/switching to a tab).  ![Query Refresh Menu](/assets/img/excel-1/query_refresh)  To reach the above window, right-click any query & click `Properties`
   - **Not limited by Excel's sheet size**: This one starts getting significant as you deal with more and more data. Excel worksheet object has a limit of 1048576 (2<sup>10</sup>) rows and 16384 (2<sup>14</sup>) columns. Therefore, you cannot fit data larger than that in a sheet, and you might get clipboard memory challenges while copy-pasting that data as well. However, when you use Power Query you have the option to load data as:
       1. Excel Table - This will paste a table to any selected location in your workbook having all the columns but only showing a fraction of rows
       2. Pivot Table - This will insert a pivot table at any selected location in your worbook which you can use to create summary views/tables from your data
       3. Connection - This only creates a connection to your source data. You can find this connection under `Queries & Connections` section and can even use it to populate table or create a pivot table later
   - **Ability to assess data quality**: Everytime we load in a new data source, we start looking for distributions of continuous variables, any missing data in any columns, frequency distributions for categorical variables, etc. Before PowerQuery, all this had to be done manually after loading data in Excel but, now all that can be done by just a few clicks in the PowerQuery Editor window. You can check the distributions, see if any column has missing values, and even fill/remove those rows with missing data.
   - **Ability to infer & control data-types**: Excel has had a bad reputation for inflicting its own understanding of the data type upon the world. *Jeez, that's just so judgemental!* Jokes apart, this is a serious problem and no organization is immune to it. However, when we are importing data through PowerQuery we can check what data type has PowerQuery inferred and even change that at any point of time. Also, as this is a query pipeline which pulls data, transforms it, and loads it the way you choose, it never impacts the original source file. Therefore, you never compromise the source data & associated formats, which you might have in case you had decided to open that manually in Excel and copy-pasting.
   - **Ability to add transformations as part of ingestion pipeline**: Imagine a situation where you want to filter the source data, add few calculated columns, aggregate to some extent but without bulking up your Workbook by loading in the data first. Before PowerQuery, Excel could not do this and was at disadvantage with products like Alteryx, Knime, BI tools, which allow users to create data cleansing & transformation workflow. PowerQuery filled that gap for Excel and quite beautifully. You can use the familiar ribbon based UI to perform these transformation operations and see them getting added as sequential steps to your data ingestion pipeline. Once done, you can just click `Load to` and choose the desired way to load the resulting data
   - **Supports a wide array of data sources**: The source data that we want to analyze might be sitting at different locations (especially in larger organizations). It could be coming from your CRM, ERP, data cubes, data lake, big data warehouse, reporting services, data feeds, or shared storages like cloud/shared directories. Due to this reason, PowerQuery supports many different data source formats. While you might end up using `csv` files as source around 90% of the time, it might help to know what other options are available. Here is a list
       * From files - File formats you can use include `CSV`, `Excel Worbook (XLSX or XLSM or XLSB)`, `XML`, `JSON`, and `PDF`. Regardless of the format, you can still use PowerQuery to transform, load, and refresh your data. At times you might even want your Workbook to refer to a file stored at path relative to your workbook, for that you can check out my article on [how to reference source data using relative paths in Excel](aslalal)
       * From Databases - Options available are SQL Server, SQL Server Analysis Service (Cubes), Microsoft Access, Oracle, IBM, MySQL, PostGreSQL, Sybase, Teradata, SAP HANA, Azure, and HDFS. This should cover you for 99% of your data import scenarios. If the one you are looking for is now available, you would need to setup an export schedule from your source data to a format that Excel can understand.
       * From Online Services - Options available are SharePoint lists, Salesforce reports/objects