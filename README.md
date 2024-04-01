
# Adventure Works-Dashboard

## Problem Statement

This dashboard helps the company understand their revenue and total orders for a specific period better. It helps the company know the who are the potential custommers and their sales and returns with the products they sell. Through different intrests, they get to know their improvement area, & thus they can improve their services by identifying these area. It also lets them know the profit and return rate, thus since by using this dashboard they have identified this problem, they can further work on factors responsible for these unwanted return rates.


Also they get to know about the Monthly stats for their products.  


### Steps followed 

- Step 1 : Load data into Power BI Desktop, dataset is a csv files.
- Step 2 : Open power query editor & in view tab under Data preview section, check "column distribution", "column quality" & "column profile" options.
- Step 3 : Also since by default, profile will be opened only for 1000 rows so you need to select "column profiling based on entire dataset".
- Step 4 : It was observed that in none of the columns errors & empty values were present except column named "Prefix" in Customer LookUp table.

     ![1](https://github.com/manjunath528/mlprojet2/assets/109943347/12c23475-acca-4831-9ae1-ce34d38f46a2)

- Step 5 : In the data transformations , we created the running calendar by writing the M code in Advanced editor.

     ![2](https://github.com/manjunath528/mlprojet2/assets/109943347/003ac97b-278d-4fe7-b9e2-1d83b4513a48)
- Step 6 :As the sales data is in two seperate files, appended the files in to one folder and loaded the data and deleted the source column in the table.

- Step 7 : By default, all queries will refresh when you use the Refresh command from the Home tab. So to avoid that ,excluded the queries from refresh that don't change often(like lookups or static data tables).

- Step 8 : In the Data Modeling section, created the all the relationships between the tables.Here the static tables are Customer Lookup , Calender Lookup, Product Lookup , Territory Lookup and the fact tables are Sales Data and Return Data.

     ![5](https://github.com/manjunath528/adventure_dashborad/assets/109943347/328d277b-0b74-4106-badb-97e037131fdd)


- Step 9 : Hidded the all the foriegn keys in the fact tables for the report view so that we can filter only by using the primary keys.
      ![4](https://github.com/manjunath528/adventure_dashborad/assets/109943347/2bc6c764-7ce2-4d95-809d-5ec276bb1637)

- Step 10 : Created the all the required calucated columns and the measures for the report view.
     
    Calculated Columns are :
     
     Parent Column in the Customer Lookup table:
         
        Is Parent? = IF(
        'Customer Lookup'[TotalChildren] > 0, 
        "Yes",
        "No"
        )
     
     Customer Priority in the Customer Lookup table:
        
         Customer Priority = 
        IF(
        'Customer Lookup'[AnnualIncome] > 100000 &&
        'Customer Lookup'[Is Parent?] = "Yes",
        "Priority",
        "Standard"
        )
    
     Income Level in the Customer Lookup table:

         Income Level = 
        IF('Customer Lookup'[AnnualIncome] >= 150000, "Very High",
        IF('Customer Lookup'[AnnualIncome] >= 100000, "High",
        IF('Customer Lookup'[AnnualIncome] >= 50000,  "Average",   "Low")
        )
        )

     Education Category in the Customer Lookup table:

         Education Category = 
        SWITCH(
        'Customer Lookup'[EducationLevel],
        "High School", "High School",
        "Partial High School", "High School",
        "Bachelors", "Undergrad",
        "Partial College", "Undergrad",
        "Graduate Degree", "Graduate"
        )

     Month Short in the Calendar Looup table:
      
         Month Short = 
        UPPER(
          LEFT(
          'Calendar Lookup'[Month Name],
          3
             )
            )
    
     Weekend in the Calendar Lookup table:

          Weekend = 
            IF(
            'Calendar Lookup'[Day of Week] IN {6,7},
            "Weekend",
            "Weekday"
            )

     SKU-Category in the Product Lookup table:
          
           SKU Category = 
            LEFT(
              'Product Lookup'[ProductSKU],
                  SEARCH(
                        "-",
                        'Product Lookup'[ProductSKU]
                        )
                        -1
                    )

         
     Birth year in the Customer Lookup table:
         
         Birth Year = 
         YEAR(
         'Customer Lookup'[BirthDate]
         )
    

    Measures are:

     Quantity Returned:
        
         Quantity Returned = 
         SUM(
         'Returns Data'[ReturnQuantity]
          )

     Quantity Sold:

        Quantity Sold = 
         SUM(
         'Sales Data'[OrderQuantity]
        )

     Return Rate:
    
        Return Rate = 
         DIVIDE(
          [Quantity Returned],
          [Quantity Sold],
          "No Sales"
         )

     Bike Sales:
      
      Bike Sales = 
      CALCULATE(
         [Quantity Sold],
         'Product Categories Lookup'[CategoryName] = "Bikes"
        )

     Bike Returns:

        Bike Returns = 
        CALCULATE(
         [Quantity Returned],
         'Product Categories Lookup'[CategoryName] = "Bikes"
        ) 
    
     Bike Return Rate:

         Bike Return Rate = 
         CALCULATE(
         [Return Rate],
         'Product Categories Lookup'[CategoryName] = "Bikes"
        )

     Total Returns:

         Total Returns = 
         COUNT(
         'Returns Data'[ReturnQuantity]
         )
        
     Total Revenue:
      
        Total Revenue = 
         SUMX(
         'Sales Data',
         'Sales Data'[OrderQuantity]
         *
         RELATED(
           'Product Lookup'[ProductPrice]
         )
        )
    
     Total Cost:

       Total Cost = 
        SUMX(
         'Sales Data',
         'Sales Data'[OrderQuantity]
          *
          RELATED(
          'Product Lookup'[ProductCost]
           )
        )

     Total Customers:
       
        Total Customers = 
          DISTINCTCOUNT(
          'Sales Data'[CustomerKey]
          )
    
     Total Orders:

        Total Orders = 
         DISTINCTCOUNT(
         'Sales Data'[OrderNumber]
         )

     

     All Returns:

          All Returns = 
           CALCULATE(
           [Total Returns],
            ALL(
             'Returns Data'
           )
        )

        

     % of All Returns:

        All Returns = 
            CALCULATE(
             [Total Returns],
              ALL(
               'Returns Data'
             )
            )

     Previous Month Returns:

        Previous Month Returns = 
         CALCULATE(
         [Total Returns],
         DATEADD(
        'Calendar Lookup'[Date],
        -1,
        MONTH
       )
       )
    

     Previous Month Orders:
        
        Previous Month Orders = 
        CALCULATE(
         [Total Orders],
          DATEADD(
         'Calendar Lookup'[Date],
         -1,
         MONTH
        )
        )
    

     Previous Month Profit:
      
          Previous Month Profit = 
          CALCULATE(
          [Total Profit],
           DATEADD(
          'Calendar Lookup'[Date],
            -1,
           MONTH
        )
        )
    
     Order Target:

        Order Target = 
           [Previous Month Orders] * 1.1

     Profit Target:
      
        Profit Target = 
            [Previous Month Profit] * 1.1


     90-days Rolling Profit:
         
        90-day Rolling Profit = 
          CALCULATE(
             [Total Profit],
             DATESINPERIOD(
             'Calendar Lookup'[Date],
             LASTDATE(
             'Calendar Lookup'[Date]
          ),
         -90,
         DAY
       ) 

      )

- Step 11 : In the Report View ,created the 4 report pages for our requirement.

    - 1.Exec Dashboard
       
       Created all the required cards and charts in this report page. It includes:

        - 1.1 Inserted the Logo of the company

             ![pic_1](https://github.com/manjunath528/adventure_dashborad/assets/109943347/0b7e51ac-2bc4-4d92-b9fa-e15abe2744e9)

        - 1.2 Revenue card

             ![pic_2](https://github.com/manjunath528/adventure_dashborad/assets/109943347/a2f30e91-b137-4f8d-9cad-85b6f25c474a)
          

        - 1.3 Profit card

             ![pic_3](https://github.com/manjunath528/adventure_dashborad/assets/109943347/323e12b2-10f2-4313-971f-878e4786f726)
          
        
        - 1.4 Orders card

             ![pic_4](https://github.com/manjunath528/adventure_dashborad/assets/109943347/33373257-0185-41a7-92ec-9132d5eab771)
         

        - 1.5 Return rate  

             ![pic_5](https://github.com/manjunath528/adventure_dashborad/assets/109943347/2dc55ad6-8f7f-4529-bbd6-09cfe1135935)
          

        - 1.6 Line Chart for the Month vs Revenue
           
            ![pic_6](https://github.com/manjunath528/adventure_dashborad/assets/109943347/91a81088-47f7-4041-ba11-c45ec1136779)
          
        
        - 1.7 Bar Chart for the Category and Orders
        
             ![pic_7](https://github.com/manjunath528/adventure_dashborad/assets/109943347/bee47e0b-8d70-4fa1-b09e-89cb053dda64)
          
        
        - 1.8 Matrix chart of top 10 products
        
             ![pic_8](https://github.com/manjunath528/adventure_dashborad/assets/109943347/de106104-01cb-4e90-b1ee-c021073fd0b5)
          
        
        - 1.9 Monthly Cards for the Revenue, Orders, Returns
       
            ![pic_9](https://github.com/manjunath528/adventure_dashborad/assets/109943347/395e68e2-9129-40e8-8f51-0c0ad7de2f5d)
          
        
        - 1.10 Text cards for the Most ordered and returned products
       
             ![pic_10](https://github.com/manjunath528/adventure_dashborad/assets/109943347/52bc3813-3daa-473c-888d-06a3705dedd1)
          

        
    

    - 2.Map
      
      Created the basic map report page for the orders based on the location and added a slicer for the each region.
     
       ![pic_11](https://github.com/manjunath528/adventure_dashborad/assets/109943347/625ee943-ec33-4c2c-88f0-804a263b2214)

      

    - 3.Product Detail

       Created a drill through and configured drill through from product name in the Exec Dashboard.
       
       - 3.1 Inserted the gauge charts for monthly order, revenue and profit with their targets.
      
           ![pic_12](https://github.com/manjunath528/mlprojet2/assets/109943347/29fb59f5-8149-40c2-898d-6c18246209bc)

    
       - 3.2 Inserted the line charts for the Total profit and adjusted profit to the month wise. And added a slicer for the price adjustment.
      
           ![pic_13](https://github.com/manjunath528/adventure_dashborad/assets/109943347/c29b8a8c-06a2-4f65-bf77-46949633c3e7)

       
    
       - 3.3 Inserted the area chart for the Monthly Returns , Profit, Orders ,Revenue and Return rate using the slicer which is single selection. 
       
           ![pic_14](https://github.com/manjunath528/adventure_dashborad/assets/109943347/b8a32565-328b-4f71-b4b2-3426b3ea9db0)
       

       - 3.4 Added a Text box to get the Report Summary of the selected product.
      
           ![pic_15](https://github.com/manjunath528/adventure_dashborad/assets/109943347/1d818b84-e67e-4513-93f9-5ba49a872f98)
       

    - 4.Customer Detail
        
       - 4.1 Inserted a cards for the total unique customers, revenue per customer .
      
           ![pic_16](https://github.com/manjunath528/adventure_dashborad/assets/109943347/45225bcc-3346-4f0e-a5a7-0d66fe4713bd)
       

       - 4.2 Inserted a donut chart for the orders by income level and orders by the occupation .
      
           ![pic_17](https://github.com/manjunath528/adventure_dashborad/assets/109943347/3ddc2a93-f6a8-45dd-a9c2-4da79e58f8b5)
       

       - 4.3 Inserted the matrix to show the top 100 customers with the orders and Revenue.
       
            ![pic_18](https://github.com/manjunath528/adventure_dashborad/assets/109943347/33f88c7e-87ed-48cc-b362-24e3728d0492)
       

       - 4.4 Inserted the line chart for the total number of customers and revenue per customer using slicer for a months.
       
      
           ![pic_19](https://github.com/manjunath528/adventure_dashborad/assets/109943347/3ea60c5b-196f-4349-bbd8-b3644e67849d)

       - 4.5 Inserted the cards for the top customer by revenue , orders by the customer and Revenue of the customer.
       
            ![pic_20](https://github.com/manjunath528/adventure_dashborad/assets/109943347/3c342a95-a9e4-4625-b062-a0f096d5075e)
       

       - 4.6 Inserted the bookmark for the information.
      
            ![pic_21](https://github.com/manjunath528/adventure_dashborad/assets/109943347/de31ce7c-df75-4bbb-aa8b-43c6df4cefdd)


After completing all this our dashboards look as shown below :

- Exec Dashboard
  
     ![Screenshot 2024-03-31 205757](https://github.com/manjunath528/adventure_dashborad/assets/109943347/31d6cdad-534e-4511-8d49-ecd26e9e4842)



- Map
     
     ![Screenshot 2024-03-31 205835](https://github.com/manjunath528/adventure_dashborad/assets/109943347/d04a5806-9e53-4d14-b84d-bf7ea2151a53)


- Product Details
 
     ![Screenshot 2024-03-31 205900](https://github.com/manjunath528/adventure_dashborad/assets/109943347/9dcdca4d-057d-41f9-a7dd-2c6119e54379)

- Customer Details

     ![Screenshot 2024-03-31 205920](https://github.com/manjunath528/adventure_dashborad/assets/109943347/eb29179d-6523-460f-a04d-67be1464c47a)
     
