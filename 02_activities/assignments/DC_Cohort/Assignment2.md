# Assignment 2: Design a Logical Model and Advanced SQL

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

#### Submission Parameters:
* Submission Due Date: `November 12, 2025`
* Weight: 70% of total grade
* The branch name for your repo should be: `assignment-two`
* What to submit for this assignment:
    * This markdown (Assignment2.md) with written responses in Section 1 and 4
    * Two Entity-Relationship Diagrams (preferably in a pdf, jpeg, png format).
    * One .sql file 
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sql/pulls/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [ ] Create a branch called `assignment-two`.
- [ ] Ensure that the repository is public.
- [ ] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [ ] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via our Slack. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.

***

## Section 1:
You can start this section following *session 1*, but you may want to wait until you feel comfortable wtih basic SQL query writing. 

Steps to complete this part of the assignment:
- Design a logical data model
- Duplicate the logical data model and add another table to it following the instructions
- Write, within this markdown file, an answer to Prompt 3


###  Design a Logical Model

#### Prompt 1
Design a logical model for a small bookstore. 📚

At the minimum it should have employee, order, sales, customer, and book entities (tables). Determine sensible column and table design based on what you know about these concepts. Keep it simple, but work out sensible relationships to keep tables reasonably sized. 

Additionally, include a date table. 

There are several tools online you can use, I'd recommend [Draw.io](https://www.drawio.com/) or [LucidChart](https://www.lucidchart.com/pages/).

**HINT:** You do not need to create any data for this prompt. This is a conceptual model only. 

#### Prompt 2
We want to create employee shifts, splitting up the day into morning and evening. Add this to the ERD.

#### Prompt 3
The store wants to keep customer addresses. Propose two architectures for the CUSTOMER_ADDRESS table, one that will retain changes, and another that will overwrite. Which is type 1, which is type 2? 

**HINT:** search type 1 vs type 2 slowly changing dimensions. 

```
At baseline, both types of tables would need to store information that can connect each customer's address entry to the existing records of customers, so `customer_id` is needed (taking in integer values). Additionally, both tables need to store typical address-related attributes in Canada such as the street address (e.g., 700 University Ave), an optional second line for unit or floor numbers (e.g., Room 01, 7th Floor), city (e.g., Toronto), province (e.g., Ontario), and postal code (e.g., M7A 2S4). Additionally, if the bookstore ships internationally, then a country attribute would also be needed. Apart from `customer_id`, all of these columns would take on string values.

The two tables would diverge from here based on how they handle new information. According to the Oracle Help Centre [1], a table that allows for records to be overwritten when changes occur is called a Type 1 SCD table. I imagine a type 1 table in this context would have an attribute for storing the timestamp for when an address was changed for a customer. In practice, this means when an address changes for a customer (e.g., `customer_id` = 1), the table will update the existing row where `custumer_id` = 1 with the new attributes (`address_line1`, `address_line2`, `city`, `province`, `country`, `postal_code`). When this change occurs, the previous timestamp in an `updated_at` column (representing the last time the address for customer_id=1 was changed or inputted) would be changed to the current timestamp. This allows staff members to know how recent a change was made, although it does not allow them to track historical changes. 

A Type 2 SCD table, on the other hand, allows for historical recordkeeping by forcing each change in address to occupy a new row in the table, rather than overwriting an existing row for the customer [1]. In that case, I imagine that a unique ID or key for each address entry needs to be created (called `customer_address_id`) which takes in integer values. Alongside this key and the baseline attributes mentioned before, there needs to be a space to record the start and end date for when a given customer's address is effective (i.e., it is their current address), as well as a binary flag to indicate whether an address is the current address for each customer. So, in practice this would mean that each row includes the following attributes: `customer_address_id`, `custumer_id`, `address_line1`, `address_line2`, `city`, `province`, `country`, `postal_code`, `effective_start_date`, `effective_end_date`, and `is_current`. For example, if a given customer (customer_id = 1) had changed addresses after previously living at 700 University Ave, Toronto, ON M7A 2S4 for five years (from November 12, 2020 to November 12, 2025), then an update to the address would create a new row that contains a unique `customer_address_id`, with customer_id=1, and values for each of `address_line1`, `address_line2`, `city`, `province`, `country`, and `postal_code` (some of which may stay the same, such as when you move to a different residence in the same city or neighbourhood). The `effective_start_date' for this new address entry would be 2025-11-12 (let's assume the customer informed the bookstore on the same day they moved out of the previous place), and the `is_current` binary would be given a value of 1. The `effective_end_date` may be given a NULL value or a dummy date (e.g., 9999-12-31) since there is no end date for the current address. In parallel to adding this new information, the entry for the previous address would still need to be altered such that the `effective_end_date` is populated with a date (e.g., 2025-11-12) and the `is_current` flag is switched from 1 (yes) to 0 (no). 

Reference:
[1] https://docs.oracle.com/cd/E41507_01/epm91pbr3/eng/epm/phcw/concept_UnderstandingSlowlyChangingDimensions-405719.html

```

***

## Section 2:
You can start this section following *session 4*.

Steps to complete this part of the assignment:
- Open the assignment2.sql file in DB Browser for SQLite:
	- from [Github](./02_activities/assignments/assignment2.sql)
	- or, from your local forked repository  
- Complete each question


### Write SQL

#### COALESCE
1. Our favourite manager wants a detailed long list of products, but is afraid of tables! We tell them, no problem! We can produce a list with all of the appropriate details. 

Using the following syntax you create our super cool and not at all needy manager a list:
```
SELECT 
product_name || ', ' || product_size|| ' (' || product_qty_type || ')'
FROM product
```

But wait! The product table has some bad data (a few NULL values). 
Find the NULLs and then using COALESCE, replace the NULL with a blank for the first column with nulls, and 'unit' for the second column with nulls. 

**HINT**: keep the syntax the same, but edited the correct components with the string. The `||` values concatenate the columns into strings. Edit the appropriate columns -- you're making two edits -- and the NULL rows will be fixed. All the other rows will remain the same.

<div align="center">-</div>

#### Windowed Functions
1. Write a query that selects from the customer_purchases table and numbers each customer’s visits to the farmer’s market (labeling each market date with a different number). Each customer’s first visit is labeled 1, second visit is labeled 2, etc. 

You can either display all rows in the customer_purchases table, with the counter changing on each new market date for each customer, or select only the unique market dates per customer (without purchase details) and number those visits. 

**HINT**: One of these approaches uses ROW_NUMBER() and one uses DENSE_RANK().

2. Reverse the numbering of the query from a part so each customer’s most recent visit is labeled 1, then write another query that uses this one as a subquery (or temp table) and filters the results to only the customer’s most recent visit.

3. Using a COUNT() window function, include a value along with each row of the customer_purchases table that indicates how many different times that customer has purchased that product_id.

<div align="center">-</div>

#### String manipulations
1. Some product names in the product table have descriptions like "Jar" or "Organic". These are separated from the product name with a hyphen. Create a column using SUBSTR (and a couple of other commands) that captures these, but is otherwise NULL. Remove any trailing or leading whitespaces. Don't just use a case statement for each product! 

| product_name               | description |
|----------------------------|-------------|
| Habanero Peppers - Organic | Organic     |

**HINT**: you might need to use INSTR(product_name,'-') to find the hyphens. INSTR will help split the column. 

2. Filter the query to show any product_size value that contain a number with REGEXP. 

<div align="center">-</div>

#### UNION
1. Using a UNION, write a query that displays the market dates with the highest and lowest total sales.

**HINT**: There are a possibly a few ways to do this query, but if you're struggling, try the following: 1) Create a CTE/Temp Table to find sales values grouped dates; 2) Create another CTE/Temp table with a rank windowed function on the previous query to create "best day" and "worst day"; 3) Query the second temp table twice, once for the best day, once for the worst day, with a UNION binding them. 

***

## Section 3:
You can start this section following *session 5*.

Steps to complete this part of the assignment:
- Open the assignment2.sql file in DB Browser for SQLite:
	- from [Github](./02_activities/assignments/assignment2.sql)
	- or, from your local forked repository  
- Complete each question

### Write SQL

#### Cross Join
1. Suppose every vendor in the `vendor_inventory` table had 5 of each of their products to sell to **every** customer on record. How much money would each vendor make per product? Show this by vendor_name and product name, rather than using the IDs.

**HINT**: Be sure you select only relevant columns and rows. Remember, CROSS JOIN will explode your table rows, so CROSS JOIN should likely be a subquery. Think a bit about the row counts: how many distinct vendors, product names are there (x)? How many customers are there (y). Before your final group by you should have the product of those two queries (x\*y). 

<div align="center">-</div>

#### INSERT
1. Create a new table "product_units". This table will contain only products where the `product_qty_type = 'unit'`. It should use all of the columns from the product table, as well as a new column for the `CURRENT_TIMESTAMP`.  Name the timestamp column `snapshot_timestamp`.

2. Using `INSERT`, add a new row to the product_unit table (with an updated timestamp). This can be any product you desire (e.g. add another record for Apple Pie). 

<div align="center">-</div>

#### DELETE 
1. Delete the older record for the whatever product you added.

**HINT**: If you don't specify a WHERE clause, [you are going to have a bad time](https://imgflip.com/i/8iq872).

<div align="center">-</div>

#### UPDATE
1. We want to add the current_quantity to the product_units table. First, add a new column, `current_quantity` to the table using the following syntax.
```
ALTER TABLE product_units
ADD current_quantity INT;
```

Then, using `UPDATE`, change the current_quantity equal to the **last** `quantity` value from the vendor_inventory details. 

**HINT**: This one is pretty hard. First, determine how to get the "last" quantity per product. Second, coalesce null values to 0 (if you don't have null values, figure out how to rearrange your query so you do.) Third, `SET current_quantity = (...your select statement...)`, remembering that WHERE can only accommodate one column. Finally, make sure you have a WHERE statement to update the right row, you'll need to use `product_units.product_id` to refer to the correct row within the product_units table. When you have all of these components, you can run the update statement.
*** 

## Section 4:
You can start this section anytime.

Steps to complete this part of the assignment:
- Read the article
- Write, within this markdown file, <1000 words.

### Ethics

Read: Boykis, V. (2019, October 16). _Neural nets are just people all the way down._ Normcore Tech. <br>
    https://vicki.substack.com/p/neural-nets-are-just-people-all-the

**What are the ethical issues important to this story?**

Consider, for example, concepts of labour, bias, LLM proliferation, moderating content, intersection of technology and society, ect. 


```
This article resonated with me for multiple reasons. Outside of taking this certificate course, my involvement with AI comes through my research into compiling strategies to address bias in each stage of the AI/ML development lifecycle for AI applications in healthcare. As such, I am deeply involved with conversations around how AI/ML may create new forms of bias and inequity, or amplify existing biases/inequities in the healthcare system. Biases can exist all the way at the start of the cycle in the conception phase, where the ways in which we define the research problem and the implicit assumptions we make about the patient population can have cascading effects on the rest of the development process. An example is an application that tries to optimize resource allocation for certain patient populations by using a healthcare utilization index as a predictor. This assumes that individuals who utilize healthcare services often are more in need of resources. This assumption overlooks the fact that certain groups may be disenfranchised in their ability to access healthcare in the first place, either due to limited physical mobility (i.e., disability, lack of transportation) or limited financial resources (i.e., unable to have insurance coverage, inability to get time off from work). So, a model that suggests for resources to be allocated to a high healthcare-use group may inadvertently exacerbate the disparity among communities who are low-income, racialized, immobile, etc. 

Another ethical issue I have is the lack of acknowledgement or financial compensation to groups who provide the data for these models. The story mentions the fact that MTurkers were paid pennies for selecting the right picture of a hotdog out of an array of photos. If this image set and the resulting models that were trained on it, became successful enough to generate millions or billions of dollars for the 'creator' of the code, I can't help but feel that the invisible figures behind the dataset labelling were being under-compensated for their work and not given a fair share. We have seen this with the way that "AI art" is a conglomeration of existing works from many, many human artists. These artists' works were responsible for the training and output of these models, yet these artists cannot be compensated properly for their contributions to the model. This issue could translate to much larger implications in healthcare. For example, with the increased use of AI in healthcare and the desire to implement it in under-served regions like across northern Ontario, there is the valid concern that data from Indigenous peoples and their cultural health teachings would be exploited without proper consent and trust-building. The Ownership, Control, Access, and Possession (OCAP) principles exist to centre Indigenous peoples' right to control their own data. I worry about how their data will be protected and remain sovereign in the current age where private information is continually used to train models in the public and private sectors. 
```
