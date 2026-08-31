This application is a highly sophisticated Comprehensive Demand Planning (S&OP) and MRP (Material Requirements Planning) System. Its main objective is to transform historical sales data, current inventories, and logistical transit data into automated, accurate, and visual replenishment decisions.

It is specifically designed to govern supply chain flow, prevent stockouts, optimize tied-up capital, and manage import logistics (National/Export suppliers).

Below is a breakdown of the application's architecture and functionality, divided by its main modules:

1. Analytical Engine and ETL (Behind the scenes)
Before displaying any charts, the app performs heavy data processing tasks:

Cleaning and Conversion: Dynamically standardizes units of measurement (Kilograms, Quintals, Bags/Physical Units) depending on the SKU.

Business Rules Intelligence: Instead of using a simple average that can be misleading, it calculates future consumption using a robust formula that defends against spikes: (Max1_6W + Max2_6W + Average_4W) / 3.

Machine Learning (Backtesting): Trains advanced predictive models (Random Forest, XGBoost, Holt-Winters, and Exponential Smoothing), evaluating their accuracy, MAPE, and RMSE against recent real data to see if Artificial Intelligence outperforms the S&OP business rule.

2. User Interface (UI) Modules
The application is divided into 6 strategic tabs, designed for different levels of decision-making:

Global Executive Dashboard
A managerial view to understand the overall health of the inventory.

Calcula the total inventory value in currency (C$) and physical units.

Displays global Days of Coverage and detects critical SKUs that have less than 20 days of inventory against demand spikes.

Identifies atypical variations (strong growth or drops in consumption by comparing the last 2 weeks against the previous month).

OTIF - Transit Tracking
Logistical control module to measure supplier compliance.

Compares estimated times of arrival (ETA) with reality to calculate the OTIF (On-Time In-Full) indicator.

Features an interactive auditing system where the user can manually confirm which containers have already entered the plant and save that record in Excel, or revert erroneous receptions.

S&OP and Container Simulator (What-If)
The sandbox area for the planner.

Allows selecting critical SKUs and "injecting" simulated purchases at different future dates.

Automatically calculates if those simulated purchases fit into standard 20,000 KG containers (load planning/cubing).

Projects whether the suggested quantities will prevent stockouts without generating excess inventory.

MRP Detail per SKU
The deepest and most tactical module for analyzing one code at a time.

Displays historical charts comparing actual consumption vs. predictions from Machine Learning models and the S&OP rule.

Issues Red Alerts if it projects a real stockout in less than 15 days.

Renders the Dual MRP Chart: A visual forward-looking simulation that shows how inventory will drop day by day, crossing the Reorder Point (ROP) and Safety Stock (SS) lines, and rising back up when actual or simulated transits arrive.

Reorder Parameters (ROP/Q)
A purely mathematical matrix ready to trigger purchases.

Calculates the Safety Stock (SS) using the standard deviation of demand and the supplier's Lead Time.

Defines the exact Reorder Point (ROP).

Suggests dynamic purchasing lots by intelligently segmenting whether the supplier is National (PN) or Export (PE), assigning different buffer times.

Model Auditing
A transparency panel to evaluate the AI.

Statistically breaks down which predictive model (XGBoost, Random Forest, etc.) was most accurate for each specific SKU, allowing for accuracy audits.
