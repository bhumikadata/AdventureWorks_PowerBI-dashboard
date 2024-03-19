

# Adventure Works Analytics: Customers and Products Comprehensive Analysis

### Dashboard link : 
https://app.powerbi.com/view?r=eyJrIjoiM2U5ZWU1ZmMtYzg2Ni00YWU3LTgyYjMtNGIwNWU4NzAwZGNiIiwidCI6ImQzNWE5MjEwLTE3MzMtNGQ3My04NzlhLTJmOGQ3YWNkMTNhOCJ9

### Project Breif

This Dashboard is for a hypothectical company Adventure Works, a global manufacturing company that produces cycling equipments and accessories. Let's assume that the management team needs a way to track KPLs(sales, revenue, profir,returns), comapare regional performance, analyze product-level trends and identify high value customers.




## Objective

The Objective is to use Power BI to:
- Connect and tranform the raw data
- Build relational data model
- Create calculated columns and measures with DAX
- Design an interactive dashboard to visualize the data
## Steps Followed

- Loaded data into Power BI, dataset comes from various csv files.
- There are two Fact tables Sales Data and Returns and the rest are Dimension or Lookup tables namely Calender Lookup table, Product Lookup table, Territoty Lookup table, Customer lookup table, Product Category and Product Subcategory table.  
- Open Power Query editor & in view tab under data preview section, check "Column Distribution", "Column Quality" and "Column Profile".
- The default column profiling only analyzes the first 1000 rows but our Customer Lookup table contains more than 18000 rows, so selected "Column Profiling" for based on entire dataset.
- After applying column profiling it was observed that Customer Key column contained data format errors which were removed using power query suggestions and filter the empty rows.
- Used power query editor to perform data transformation using text-specific tools and numerical tools.
- Created a new column in called "Full Name" in the Customer Lookup table by merging columns from Add Column menu.
- I did some exploratory analysis in the Product Lookup table 
  by counting distinct product name, Avg product price, and Max   price of the product.
- I used date and time tools to build  Calender Lookup table in the model and updated all of the dates in long forms into short date from column tools and we did exact same steps with long dates in Sales Data
- Grouped and aggregaed columns based on Product Key and Customer Key in Sales Data to get total quantity and appended sales data of year 2020, 2021, and 2022 into single query.
- Created relationship between all the tables in the Model View of Power BI, all table were connected with the Fact Sales table and Fact Returns table  in Star schema except the Product Lookup table was further connected to Product Category and Product Sub-Category table in Snowflake schema.
- Customized data format from the column tools menu in the Data view and assigned data categories for geospatial fileds in the Territoty Lookup table.
- Created a territory hierarchy in the Territoty Lookup table and a date hierarchy in the Calender Lookup table.
- Created a calculated column in Sales Table called Quantity Type which was based on order Quantity. The following DAX was written:
      
      Quantity Type = 
                   IF
          ('Sales Data'[OrderQuantity] > 1,
              "Multiple Items",
              "Single Items"
          )

- Created a calculated column in Sales Table called Retail Price which was based on order Quantity. The following DAX was written:
      
      Quantity Type = 
                   IF
          ('Sales Data'[OrderQuantity] > 1,
              "Multiple Items",
              "Single Items"
          )

- Created a new measure called Quantity Sold
  Quantity Sold = 
  SUM('Sales Data'[OrderQuantity]
  )
- Created a dedicated table to store measures by using Enter Data into Power Query and moved Quantity Sold measure created in Sales Data table to the Measure table.

- During this project following measures were created using DAX:
          
       Total Orders = 
                DISTINCTCOUNT('Sales Data'[OrderNumber])


        Total Returns = 
         COUNT('Returns Data'[ReturnQuantity])  

        Total Revenue =  
            SUMX(
             'Sales Data',
            'Sales Data'[OrderQuantity]*
             RELATED(
             'Product Lookup'[ProductPrice]
              )
            )        

        Total Cost = 
            SUMX(
             'Sales Data',
             'Sales Data'[OrderQuantity]
              *
            RELATED ('Product Lookup'[ProductCost]) 
            )

        Total Profit = [Total Revenue] - [Total Cost]   

        Total Customers = DISTINCTCOUNT('Sales Data'[CustomerKey])

        Quantity Returned =  sum('Returns Data'[ReturnQuantity])

        Return Rate = 
         DIVIDE(
            [Quantity Returned],
            [Quantity Sold],
            "No Sales"
            )
        
        Previous Month Orders = 
               CALCULATE(
                     [Total Orders],
                     DATEADD(
                         'Calendar Lookup'[Date],
                         -1,
                         MONTH
                         )
                        )
       Previous Month Revenue = 
                CALCULATE(
                    [Total Revenue],
                    DATEADD(
                        'Calendar Lookup'[Date], 
                        -1, 
                        MONTH
                        )
                       )

      Revenue Target = [Previous Month Revenue] *1.1               

      Revenue Target Gap = [Total Revenue] - [Revenue Target]

      Profit Target = [Previous Month Profit]*1.1  

      Profit Target Gap = [Total Profit] - [Profit  Target]  

      YTD Revenue = 
               CALCULATE(
                   [Total Revenue],
                   DATESYTD(
                       'Calendar Lookup'[Date]
                       )
                    )

       All Orders = CALCULATE( 
                            [Total Orders],
                            ALL(
                                'Sales Data'
                                  )
                             )             

        % of All Orders = 
               DIVIDE(
                    [Total Orders],
                    [All Orders]
                    )

        All Returns = 
                CALCULATE(
                     [Total Returns],
                ALL(
                      'Returns Data'
                    )                    
                    )

         % of All Returns = 
                DIVIDE(
                     [Total Returns],
                     [All Returns]
                      )         

- Keeping the goals of this project in mind I decided to break down the dashboard into a multi-page dashboard containg Executive Dashboard, Map, Product Details, and Customer Details.

- The first visual I inserted into the dashboard were cards showing key metrics like Total Revenue, Total Profit, Total Orders, and Return Rate.

![Cards_exec dashboard](https://github.com/Princekrampah/WalmartSalesAnalysis/assets/131578649/e624b1ee-2f41-476d-a218-6be9e19d5d87)

- Cards were created on Customer Detail dashboard showing 
  unique customers and revenue per customer.

![Cards_customerdash](https://github.com/Princekrampah/WalmartSalesAnalysis/assets/131578649/8cf76e49-6176-411e-8647-dfb5d8e1e38a)


- The line chart in this dashboard illustrates the revenue trends over different time intervals. The x-axis is configured to display the start of the year, quarters, and months, providing a comprehensive view of revenue distribution across various time frames. Meanwhile, the y-axis represents the corresponding revenue values.


![LineChart_ExecDash](https://github.com/Princekrampah/WalmartSalesAnalysis/assets/131578649/1fcddef9-19b2-4458-b2b6-078627fd229a)


- I realized I need to define a customer metric selection for creating a table called Customer Metric Selection.This code creates a table named "Customer Metric Selection" with two columns: "Metric Name", "Metric Value", and "Sort Order". 

      Customer Metric Selection = 
      {("Total Customers", NAMEOF('Measure Table'[Total Customers), 0),
       ("Revenue per Customer", NAMEOF('Measure Table'[Average Revenue per Customer]),
        1)}


It is utilized in the below line chart where we want to dynamically switch between total customers and revenue per customer, allowing for flexibility in the visual.

 ![LinChart_CustomerDetail](https://github.com/Princekrampah/WalmartSalesAnalysis/assets/131578649/de8bb68d-926d-47f1-84e7-1a706fb50b17)

- Included a comparative analysis between previous month revenue,current month revenue,previous month orders and current month orders and monthly returns through the integrated KPI cards in Executive Dashboard which serves as a valuable tool for evaluating month-to-month performance trends.

![KPICard_execdash](https://github.com/bhumikadata/Bellabeat/assets/131578649/661d7ae8-828e-440f-9d9b-959d8ffb579c)

- Inserted a bar chart where categories are plotted on the y-axis and total orders are visualized on the x-axis, establishing a clear correlation between the categories and their respective contribution to the overall total orders.

![barchart_execdash](https://github.com/bhumikadata/Bellabeat/assets/131578649/1aa1d4cc-7ed5-477d-a784-15311bd97d42)

- Added a donut chart to the Customer Detail 
  report showing Total Orders by Income Level. I then copied the chart to show Total Orders by Occupation and add a visual level filter to display the three occupation with most orders (used Top N filter).

![DoughnutChart_customerdetail](https://github.com/bhumikadata/Bellabeat/assets/131578649/303406bc-ecf0-4760-b293-0b507758d3b6)

- Implemented a data-rich table showcasing customer key, full name, total orders, and total revenue. The table is optimized to present insights specifically for the top 100 customers, offering a focused view into key customer metrics.
  
 ![Table_Customer_detail](https://github.com/bhumikadata/Bellabeat/assets/131578649/27bae6a3-989e-459d-b5c1-036bc491f1f3)

 - Inserted two cards and added a visual level Top N filter, highlighting the most frequently ordered item and most returned within product subcategory. This enhances visibility into popular products and most returned products within specific subcategory.

 ![Cards_exec](https://github.com/bhumikadata/Bellabeat/assets/131578649/0ed8a0d9-2c26-4db9-8d95-9b486392b9ff)

 - Inserted cards visual in Customer Detail report to show top customer (full name)
in terms of Total Revenue along with Total Orders and total Revenue for top customer.

![Top customer cards_customerdetails](https://github.com/bhumikadata/Bellabeat/assets/131578649/7ef94efb-f949-4a2e-8770-e48fc5caa443)

- Incorporated an interactive map visual, employing country-specific bubbles to represent total orders. This geospatial representation allows users to quickly identify regions with the highest order volumes.

![map](https://github.com/bhumikadata/Bellabeat/assets/131578649/fc3d45a6-cfd5-42f6-a433-2d2b145a5ad7)


- Inserted report slicers to filter map on the basis of Continent.

![slicer_map](https://github.com/bhumikadata/Bellabeat/assets/131578649/b3c0c383-e254-4cf8-9435-633760c829cf)

- Added a silcer to filter the Customer Detail report page by year and also added three new measures using HASONEVALUE function Full Name (Customer Detail), Total Order (Customer Detail), and Total Revenue (Customer Detail)

           Full Name (Customer Detail) = 
                    IF(
                       HASONEVALUE(
                         'Customer Lookup'[CustomerKey]
                        ),
                       max(
                          'Customer Lookup'[Full Name]
                           ),
                          "Multiple Orders"
                       )

           Total Orders (Customer Details) = 
                       If( 
                          HASONEVALUE(
                            'Customer Lookup'[CustomerKey]
                                      ),
                                     [Total Orders],
                                      "-"
                            )


            Total Revenue (Customer Detail) = 
                       If(
                         HASONEVALUE(
                             'Customer Lookup'[CustomerKey]
                                      ),
                                    [Total Revenue],
                                     "-"
                            )                

The use of HASONEVALUE function is to ensure that the information is only displayed when dealing with a single customer. In other cases, where there are multiple customers or none selected, the measures show a "-" to indicate that the information is not applicable or cannot be determined in those situations.

![slicer_customerdetail](https://github.com/bhumikadata/Bellabeat/assets/131578649/f5c5c10e-bb29-4f1a-8365-d579abb5f94d)

- Inserted Guage chart comparing monthly orders vs target orders. In order to view the total orders on the start date of the month I added visual level TOP N filter and added Order Target measure on the Maximum Value so as to make the guage intuitive.

![Guage_chart](https://github.com/bhumikadata/Bellabeat/assets/131578649/a7f03573-bc14-4f81-9b4e-9109cd9842da)

-  Create a Numeric range parameter from Modelling menu called Price Adjustment (%) parameter and the minimum and maximum values for the parameter to create the numeric range with an increment of 0.1 and set the default to 0. In order to tie valu from the parameter to the model, I created three new maesures that take value from the selected parameter and claculate the adjusted price, adjusted revenue and adjusted profit.
  
         Adjusted Price = [Average Retail Price] * (1+'Price Adjustment (%)'[Price Adjustment (%) Value])


       Adjusted Revenue = 
            SUMX(
               'Sales Data',
               'Sales Data'[OrderQuantity]
                *
                [Adjusted Price]
            )

       Adjusted Profit = [Adjusted Revenue] - [Total Cost]      
    

![new Areachart_productdetail](https://github.com/bhumikadata/AdventureWorks_PowerBI-dashboard/assets/131578649/e6631178-0c0e-4c57-8159-a6efe6c5be46)


- Added a new field parameter called Metric selection contain 5 different measures from our model Orders, Revenue, Profit, Returns, and Return %. 

![feild parameter](https://github.com/bhumikadata/AdventureWorks_PowerBI-dashboard/assets/131578649/32290fa4-e1da-4446-9e93-54712af34ac6)



- Inserted a customer silcer panel and also a a button on top right corner to get back to hidden state. 

![slicer panel](https://github.com/bhumikadata/AdventureWorks_PowerBI-dashboard/assets/131578649/cf1e6aff-cb4f-4097-8261-6a70f5ebbedc)

## Snapshot of dashboard (Power BI Service)

![page1](https://github.com/bhumikadata/AdventureWorks_PowerBI-dashboard/assets/131578649/ee31623e-aaec-4402-b564-4e057c727e36)

![page2](https://github.com/bhumikadata/AdventureWorks_PowerBI-dashboard/assets/131578649/f76467ec-4c75-45eb-a5bb-3b17dad18cca)

![page3](https://github.com/bhumikadata/AdventureWorks_PowerBI-dashboard/assets/131578649/6ba8cf34-2594-4631-bc5d-9af16285c4fb)

![page4](https://github.com/bhumikadata/AdventureWorks_PowerBI-dashboard/assets/131578649/076b6796-f523-4f65-bc33-ac6db3777546)

