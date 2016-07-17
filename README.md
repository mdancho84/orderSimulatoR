# orderSimulatoR

___Fast and easy `R` order simulation for customer and product learning!___

## About

`orderSimulatoR` enables fast and easy creation of order data for simulation, data mining and machine learning. In it's current form, the `orderSimulatoR` is a collection of scripts that can be used to generate sample order data from the following inputs: customer table (e.g. `bikeshops.xlsx`), products table (e.g. `bikes.xlsx`), and customer-products interaction table (e.g. `customer_product_interactions.xlsx`). The output will be order data. Example input files are provided (refer to the data folder). The output generated is similar to that in the file `orders.xlsx`.

## Why this Helps

It's very difficult to create custom order data for data mining, visualization, trending, etc. I've searched for good data sets, and I came to the conclusion that I'm better off creating my own orders data for messing around with on my blog. In the process, I made an algorithm to generate the orders. I made the algorithm publicly available so others can use to show off their analytical abilities.

## Example Usage

* [ORDERSIMULATOR: SIMULATE ORDERS FOR BUSINESS ANALYTICS](http://www.mattdancho.com/business/2016/07/12/orderSimulatoR.html)

## How to Create Orders

There's a few basic steps. I've provided some sample data in the `data` folder to help with the explanation. The scripts used are in the `scripts` folder.

### Prior to starting you will need:

1. __Customers Table__: This is a simple list of customers with features including customer.id, customer.name, and other features related to customers that you plan on using for data mining. An example customer table is provided as `bikeshops.xlsx`.

2. __Products Table__: This is a simple list of products with features including product.id, product.name, and other features related to products that you plan on using for data mining. An example products table is provided as `bikes.xlsx`.

3. __Customer-Product Interactions Table__: This is a matrix that links the customers with their preferences to the products via product features. An excel file is provided that has steps to walk you through the process (refer to `customer_product_interactions.xlsx`). The basic idea is to create a probability distribution for each customer-product in a matrix. The excel file includes some excel functions that helped in setting up the sample files. However, you are free to create the distributions any way you'd like. The key is to use the customer's preferences to create probabilities that will ultimately lead to order trends when simulated. As a result, the more thought you put into the matrix, the better for creating interesting trends that can be data-mined.

### Creating Orders

Once you have the three input tables, you are ready to create orders. A sample script used to create the `orders.xlsx` file is `orderScript.R`, and this can be used as a template. The following scripts are used in sequence, and the parameters are discussed below:

1. `createOrdersAndLines.R`: This function generates the orders and order lines. The function inputs are the number of orders, the maximum number of lines an order can have, and the rate. The number of orders and maximum lines are self explanatory. The rate affects the distribution of the number of lines (line counts) on a given order using the probability density function: _1 / (order line number ^ rate)_. For example, max lines = 3 and rate = 1 yields the following probability distribution:

 * probability of line count of 1 on a given order is _1/(1^1) = 1_ --> scaled to _1 / (1 + 0.5 + 0.33) = 55%_
 * probability of line count of 2 on a given order is  _1/(2^1) = 0.5_ --> scaled to _0.5 / (1 + 0.5 + 0.33) = 27%_
 * probability of line count of 3 on a given order is _1/(3^1) = 0.33_ --> scaled to _0.33 / (1 + 0.5 + 0.33) = 18%_

 Adjusting the rate higher than 1 will place more probability on line 1 and less on subsequent line counts. Increasing the max line count will change the probabilities as well.

2. `createDatesFromOrders.R`: This function adds dates to the orders. The function inputs are the orders data frame from step 1, a monthly order distribution, a yearly order distribution, and a start year. The monthly order distribution is a 12 length numeric vector sequence that determines how orders are distributed Jan through Dec. Seasonality can be input by weighting certain months more than others. The sum must sum to 1.0. The yearly distribution works the same as the monthly distribution, and depending on the weighting will indicate growth or declines in sales over the years. The number of years is determined by the length of the distribution, and the first year will be the start year.

3. `assignCustomersToOrders.R`: This function assigns customer.id's to the orders. The inputs are the orders data frame from step 2, the customers table as a data frame, and the rate. The customers table must have the id's in column 1. The rate alters the customer distributions the same way as in step 1 (`createOrdersAndLines.R`). One twist is that the customers are shuffled so the weighting of orders assigned to customers is selected randomly and then distributed according the the probability density function _1 / (shuffled customer position ^ rate)_. The greater the rate, the more that orders will be unevenly get assigned to top customers.

4. `assignProductsToCustomers.R`: This function assigns products to customers using the probabilities from the customer-probability interaction assignments. The inputs are the orders data frame from step 3 and the customer-product interaction matrix. Make sure to remove all columns except for the product.id and the customer.id probability columns. Make sure all customer.id probability columns sum to 1.0.

5. `createProductQuantities.R`: This function assigns product quantities to each line of the orders. The inputs are the orders data frame from step 4, the max quantity (make sure this is less than the total products), and the rate. This rate controls the distribution of quantities. Higher rates place more probability on lower line quantities.

