# Retail Sales Power BI — Business Concepts and Metric Definitions

This document explains the business logic, data-model structure, and DAX concepts used in the simulated Retail Sales Power BI project. It is intended to serve as both project documentation and a personal learning reference.

> **Scope note:** The dataset is simulated. Definitions in this document describe how the project currently interprets the fields. Any unclear business rules should be validated before treating them as real-world facts.

## Contents

- [1. Model overview](#1-model-overview)
- [2. Core data-model concepts](#2-core-data-model-concepts)
- [3. Table definitions](#3-table-definitions)
- [4. Business and financial concepts](#4-business-and-financial-concepts)
- [5. Metric definitions](#5-metric-definitions)
- [6. Worked product example](#6-worked-product-example)
- [7. DAX concepts](#7-dax-concepts)
- [8. DAX reference](#8-dax-reference)
- [9. Date-table logic](#9-date-table-logic)
- [10. Assumptions and validation questions](#10-assumptions-and-validation-questions)

## 1. Model overview

The model uses a **star schema** with one central fact table and four dimension tables.

| Table | Type | Expected grain |
| --- | --- | --- |
| `Transactions` | Fact table | One row per transaction record or transaction line |
| `Products` | Dimension | One row per product |
| `Customers` | Dimension | One row per customer |
| `Stores` | Dimension | One row per store |
| `DimDate` | Dimension | One row per calendar date |

The relationships are:

| One side | Many side | Relationship |
| --- | --- | --- |
| `Products[ProductID]` | `Transactions[ProductID]` | One product can appear in many transactions |
| `Customers[CustomerID]` | `Transactions[CustomerID]` | One customer can have many transactions |
| `Stores[StoreID]` | `Transactions[StoreID]` | One store can have many transactions |
| `DimDate[Date]` | `Transactions[Date]` | One date can contain many transactions |

Filters should normally flow in a single direction from each dimension table into `Transactions`.

## 2. Core data-model concepts

### Fact table

A fact table records business events. In this project, the event is a retail transaction.

The `Transactions` table contains identifiers that connect each event to its product, customer, store, and date. It also contains numeric values such as quantity and discount that can be aggregated or used in calculations.

### Dimension table

A dimension table describes the **who, what, where, or when** associated with a business event.

- `Customers` describes **who** purchased.
- `Products` describes **what** was purchased.
- `Stores` describes **where** it was purchased.
- `DimDate` describes **when** it was purchased.

Dimensions are used to filter, group, and label facts. For example, `Products[Category]` can filter Total Sales even though the Total Sales calculation is based on rows in `Transactions`.

The prefix `Dim` is a naming convention meaning **dimension**. It does not change Power BI functionality.

### Grain

The grain states exactly what one row represents.

- The grain of `Products` is one product.
- The grain of `Customers` is one customer.
- The grain of `Stores` is one store.
- The grain of `DimDate` is one date.
- The expected grain of `Transactions` is one transaction record or line.

Grain should be established before creating metrics because it determines what can safely be counted or summed.

### Primary key

A primary key uniquely identifies each row in a dimension table.

Examples:

- `Products[ProductID]`
- `Customers[CustomerID]`
- `Stores[StoreID]`
- `DimDate[Date]`

Each of these fields should contain unique, nonblank values on the **one** side of a relationship.

### Foreign key

A foreign key appears in the fact table and points to a corresponding dimension row.

For example, `Transactions[ProductID]` is a foreign key that points to `Products[ProductID]`. The same ProductID can repeat in `Transactions` because the product may be sold many times.

### One-to-many relationship

In a one-to-many relationship:

- `1` represents the dimension table, where the key is unique.
- `*` represents the fact table, where the key can repeat.

For example, one product can appear in many transaction rows.

### Star schema

A star schema places the fact table in the center and connects dimensions around it. This structure:

- keeps descriptive fields separate from business events;
- makes filter behavior easier to understand;
- reduces repeated descriptive data;
- supports reusable measures across products, dates, stores, and customers.

## 3. Table definitions

### Products

**Purpose:** Describes the products available for sale.

**Grain:** One row per product.

| Field | Definition |
| --- | --- |
| `ProductID` | Unique product identifier and primary key |
| `ProductName` | Descriptive product name |
| `Category` | Broad product grouping, such as Electronics |
| `SubCategory` | More specific grouping, such as Camera |
| `UnitPrice` | Customer-facing price for one unit before a transaction discount |
| `CostPrice` | Direct cost to the retailer for one unit, interpreted as an acquisition or production cost |

`Products` is a dimension because it describes **what** was sold. ProductID repeats in `Transactions` whenever the product is purchased again.

### Transactions

**Purpose:** Records retail sales activity.

**Expected grain:** One row per transaction or transaction line.

| Field | Definition |
| --- | --- |
| `TransactionID` | Identifier for the transaction record |
| `Date` | Date of the transaction |
| `CustomerID` | Foreign key connecting the transaction to a customer |
| `ProductID` | Foreign key connecting the transaction to a product |
| `StoreID` | Foreign key connecting the transaction to a store |
| `Quantity` | Number of product units sold in the row |
| `Discount` | Percentage discount stored as a decimal; for example, `0.15` means 15% |
| `PaymentMethod` | Method used to pay, such as Cash or Credit Card |
| `Line Revenue` | Revenue generated by the individual transaction row after discount |
| `Line Profit` | Gross profit generated by the individual transaction row |

`Transactions` is the fact table because it records **what happened**.

### Customers

**Purpose:** Describes customers associated with transactions.

**Grain:** One row per customer.

| Field | Definition |
| --- | --- |
| `CustomerID` | Unique customer identifier and primary key |
| `FirstName` | Customer first name |
| `LastName` | Customer last name |
| `Gender` | Customer gender value supplied by the dataset |
| `BirthDate` | Customer date of birth |
| `City` | Customer city |
| `JoinDate` | Date the customer joined or was acquired |

Potential derived concepts:

- **Age:** Time between BirthDate and an agreed reference date.
- **Tenure:** Time between JoinDate and an agreed reference date.
- **Customer lifetime value:** Revenue or profit attributed to a customer over an explicitly defined period and methodology.

Age and tenure require a clear reference date. Using the current date makes the values change over time; using the transaction date or dataset end date answers a different business question.

### Stores

**Purpose:** Describes retail locations.

**Grain:** One row per store.

| Field | Definition |
| --- | --- |
| `StoreID` | Unique store identifier and primary key |
| `StoreName` | Descriptive store name |
| `City` | Store city |
| `Region` | Broader geographic grouping |

The table supports comparisons of sales, profit, units, and customers by store or region.

### DimDate

**Purpose:** Provides a complete and reusable calendar for time-based analysis.

**Grain:** One row per calendar date.

| Field | Definition |
| --- | --- |
| `Date` | Unique calendar date and primary key |
| `Year` | Calendar year |
| `Quarter` | Calendar quarter, displayed as Q1 through Q4 |
| `Month` | Full month name |
| `MonthNumber` | Numeric month from 1 to 12, used to sort Month |
| `YearMonth` | Month and year label, such as Sep 2023 |
| `YearMonthSort` | Numeric year-month key, such as 202309, used to sort YearMonth |

`DimDate` is preferable to relying only on Power BI's automatic hidden date hierarchy because it provides explicit, reusable fields and controlled sorting.

## 4. Business and financial concepts

### Unit price

Unit Price is the listed selling price charged to the customer for one unit before the transaction discount.

### Cost price

Cost Price is the retailer's direct cost for one unit. Depending on the real business, this might represent wholesale acquisition cost, manufacturing cost, or another direct product cost. The simulated dataset does not establish which interpretation applies, so this project treats it generally as direct product cost.

### Quantity

Quantity is the number of units included in the transaction row.

### Discount

Discount reduces the amount paid by the customer. A discount of `0.15` represents 15%, leaving the retailer with 85% of the pre-discount selling price.

```text
Discount multiplier = 1 - Discount
```

A discount reduces revenue and gross profit, but it does not reduce the retailer's original Cost Price.

### Gross sales

Gross Sales is the customer-facing value before discounts.

```text
Gross Sales = Unit Price × Quantity
```

### Discount amount

Discount Amount is the value removed from Gross Sales.

```text
Discount Amount = Unit Price × Quantity × Discount
```

### Revenue / net sales

In this project, Line Revenue and Total Sales represent sales after the transaction discount.

```text
Line Revenue = Unit Price × Quantity × (1 - Discount)
```

Some organizations use the terms **revenue** and **net sales** differently. The project's definition should remain explicit so that report users know discounts have already been deducted.

### Cost of goods sold

Cost of Goods Sold, often abbreviated COGS, is the direct product cost associated with the units sold.

```text
Line Product Cost = Cost Price × Quantity
```

### Gross profit

Gross Profit is revenue after discount minus direct product cost.

```text
Line Gross Profit = Line Revenue - Line Product Cost
```

The project's fields are named `Line Profit` and `Total Profit`, but they are technically **gross profit** because the dataset does not subtract operating expenses.

### Net profit

Net Profit is the amount remaining after subtracting all applicable costs and expenses, potentially including:

- product cost;
- payroll;
- rent;
- advertising;
- shipping;
- technology;
- interest;
- taxes.

The current dataset cannot calculate true Net Profit because it does not include those expenses.

### Gross margin

Gross Margin expresses Gross Profit as a percentage of revenue.

```text
Gross Margin % = Gross Profit ÷ Revenue
```

A product can generate high revenue but have a weak margin if its cost or discount is high.

## 5. Metric definitions

| Metric | Definition | Basic logic |
| --- | --- | --- |
| Gross Sales | Sales value before discount | Unit Price × Quantity |
| Discount Amount | Sales value removed by discount | Gross Sales × Discount |
| Line Revenue | Revenue for one transaction row after discount | Unit Price × Quantity × (1 - Discount) |
| Total Sales | Line Revenue aggregated over the current filter context | Sum of Line Revenue |
| Line Product Cost | Direct product cost for one row | Cost Price × Quantity |
| Line Profit | Gross profit for one row | Line Revenue - Line Product Cost |
| Total Profit | Line Profit aggregated over the current filter context | Sum of Line Profit |
| Units Sold | Total number of units | Sum of Quantity |
| Transaction Count | Number of distinct transaction records | Distinct count of TransactionID |
| Average Order Value | Average sales value per order or transaction | Total Sales ÷ distinct orders |
| Gross Margin % | Share of revenue remaining after product cost | Total Profit ÷ Total Sales |
| Top Product | Product with the highest Total Sales in the active filters | Rank products by Total Sales |
| Top Category | Category with the highest Total Sales in the active filters | Rank categories by Total Sales |

### Filter context matters

Measures change with report filters.

For example, `[Total Sales]` can represent:

- all sales with no filters;
- sales for Electronics when Category is filtered;
- sales for Store S003 when Store is filtered;
- sales for Sep 2024 when YearMonth is filtered;
- Electronics sales at Store S003 in Sep 2024 when all three filters apply.

Top Product and Top Category should also respond to the active date, store, customer, or region filters unless the DAX intentionally removes those filters.

## 6. Worked product example

For Product P001, Like Camera:

| Field | Value |
| --- | ---: |
| Unit Price | $1,673.69 |
| Cost Price | $1,323.38 |

### One unit with no discount

```text
Revenue = $1,673.69
Product cost = $1,323.38
Gross profit = $1,673.69 - $1,323.38 = $350.31
```

The $350.31 is Gross Profit per unit before other business expenses. It is not Net Profit.

### One unit with a 15% discount

```text
Discounted revenue = $1,673.69 × (1 - 0.15)
                   = $1,422.64

Product cost = $1,323.38

Gross profit = $1,422.64 - $1,323.38
             = $99.26
```

The discount reduces the gross profit from $350.31 to approximately $99.26 per unit.

### Two units with a 15% discount

```text
Line Revenue = $1,673.69 × 2 × 0.85
             = $2,845.27

Line Product Cost = $1,323.38 × 2
                  = $2,646.76

Line Profit = $2,845.27 - $2,646.76
            = $198.51
```

## 7. DAX concepts

### What DAX is

DAX stands for **Data Analysis Expressions**. It is Power BI's formula language for creating calculations in the data model.

DAX resembles Excel formulas, but it also understands tables, relationships, row context, and report filter context.

### DAX versus Power Query

| Tool | Primary purpose |
| --- | --- |
| Power Query | Imports, cleans, reshapes, and combines data before it enters the model |
| DAX | Creates model calculations after the data is loaded |

### Calculated table

A calculated table is created from a DAX expression and stored in the model.

Example: `DimDate` is created with `CALENDAR` using the earliest and latest transaction dates.

### Calculated column

A calculated column produces a stored value for every row in a table.

Examples:

- `Transactions[Line Revenue]`
- `Transactions[Line Profit]`
- `DimDate[Year]`
- `DimDate[Month]`

Calculated columns are useful when the row-level result needs to be displayed, grouped, sorted, or reused. They increase model size because their values are stored.

### Measure

A measure calculates a result dynamically based on the active filter context.

Examples:

- `[Total Sales]`
- `[Total Profit]`
- `[Average Order Value]`

A measure does not appear as a new data column. It appears with a calculator icon and is evaluated when used in a visual or another DAX expression.

### Row context

Row context means that a calculation is currently evaluating one row.

A calculated column naturally has row context. `SUMX` also creates row context while iterating through a table.

### Filter context

Filter context is the set of filters active when a measure is evaluated. Filters can come from:

- slicers;
- visual axes and rows;
- page or report filters;
- relationships from dimension tables;
- DAX functions such as `CALCULATE`.

### SUM versus SUMX

`SUM` adds the existing values in one numeric column.

```DAX
Units Sold = SUM ( Transactions[Quantity] )
```

`SUMX` iterates over a table, evaluates an expression for each row, and then sums the row results.

```DAX
Total Sales =
SUMX (
    Transactions,
    Transactions[Quantity]
        * RELATED ( Products[UnitPrice] )
        * ( 1 - Transactions[Discount] )
)
```

The `X` functions are iterators. Other examples include `AVERAGEX`, `MINX`, and `MAXX`.

### RELATED

`RELATED` retrieves a value from the one side of an existing relationship.

For a transaction row, this expression retrieves the Unit Price belonging to its ProductID:

```DAX
RELATED ( Products[UnitPrice] )
```

Conceptually, this resembles joining Products onto Transactions by ProductID, but it uses the relationship already defined in the Power BI model.

### DIVIDE

`DIVIDE` safely handles division and avoids a divide-by-zero error.

```DAX
Average Order Value = DIVIDE ( [Total Sales], [Transaction Count] )
```

## 8. DAX reference

### Line Revenue — calculated column

```DAX
Line Revenue =
RELATED ( Products[UnitPrice] )
    * Transactions[Quantity]
    * ( 1 - Transactions[Discount] )
```

### Line Profit — calculated column

```DAX
Line Profit =
(
    RELATED ( Products[UnitPrice] )
        * ( 1 - Transactions[Discount] )
        - RELATED ( Products[CostPrice] )
)
    * Transactions[Quantity]
```

This is equivalent to:

```text
(discounted selling price per unit - cost per unit) × quantity
```

### Total Sales — measure using SUMX

```DAX
Total Sales =
SUMX (
    Transactions,
    Transactions[Quantity]
        * RELATED ( Products[UnitPrice] )
        * ( 1 - Transactions[Discount] )
)
```

If Line Revenue already exists as a calculated column, the equivalent measure is:

```DAX
Total Sales = SUM ( Transactions[Line Revenue] )
```

Only one `[Total Sales]` measure is needed.

### Total Profit — measure

```DAX
Total Profit = SUM ( Transactions[Line Profit] )
```

### Units Sold — measure

```DAX
Units Sold = SUM ( Transactions[Quantity] )
```

### Transaction Count — measure

```DAX
Transaction Count = DISTINCTCOUNT ( Transactions[TransactionID] )
```

### Average Order Value — measure

```DAX
Average Order Value =
DIVIDE ( [Total Sales], [Transaction Count] )
```

This interpretation assumes each distinct TransactionID represents one complete order. If TransactionID instead represents an order line, a separate OrderID would be required for true Average Order Value.

### Gross Margin Percentage — measure

```DAX
Gross Margin % = DIVIDE ( [Total Profit], [Total Sales] )
```

## 9. Date-table logic

### Create DimDate — calculated table

```DAX
DimDate =
CALENDAR (
    MIN ( Transactions[Date] ),
    MAX ( Transactions[Date] )
)
```

After creating the table:

1. Set `DimDate[Date]` to the Date data type.
2. Mark `DimDate` as the model's date table using the Date column.
3. Create a one-to-many relationship from `DimDate[Date]` to `Transactions[Date]`.

### Date calculated columns

```DAX
Year = YEAR ( DimDate[Date] )
```

```DAX
Month = FORMAT ( DimDate[Date], "MMMM" )
```

```DAX
MonthNumber = MONTH ( DimDate[Date] )
```

```DAX
Quarter = "Q" & QUARTER ( DimDate[Date] )
```

```DAX
YearMonth = FORMAT ( DimDate[Date], "MMM yyyy" )
```

```DAX
YearMonthSort =
YEAR ( DimDate[Date] ) * 100
    + MONTH ( DimDate[Date] )
```

### Sorting date labels

Text month names sort alphabetically by default. To display them chronologically:

- Select `DimDate[Month]` and sort it by `DimDate[MonthNumber]`.
- Select `DimDate[YearMonth]` and sort it by `DimDate[YearMonthSort]`.

This is similar to using an ordered categorical type in pandas.

## 10. Assumptions and validation questions

The following assumptions should be documented or confirmed before presenting the dashboard as a definitive business analysis.

### Current assumptions

- The data is simulated rather than actual company data.
- `Discount = 0.15` means a 15% discount.
- Unit Price is the pre-discount customer selling price per unit.
- Cost Price is the direct product cost per unit.
- Line Revenue excludes the discount but does not account for taxes, returns, shipping, or refunds.
- Line Profit and Total Profit represent Gross Profit, not Net Profit.
- Product prices and costs are treated as static because they are stored in the Products dimension rather than recorded historically in each transaction.

### Questions to validate

1. Does each TransactionID represent a complete order or a single order line?
2. Can one real order contain multiple products, and if so, is there a separate OrderID?
3. Does Cost Price represent wholesale cost, manufacturing cost, or another direct cost?
4. Does Unit Price include tax or shipping?
5. Are discounts always applied to the entire transaction line?
6. Are returns, cancellations, and refunds present?
7. Are Product prices and costs intended to remain constant across the entire period?
8. What reference date should be used for Age and Customer Tenure?
9. How should Customer Lifetime Value be defined: total revenue, total gross profit, predicted value, or another method?

Keeping these assumptions visible prevents a technically correct calculation from being presented with the wrong business meaning.
