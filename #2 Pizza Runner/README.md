# Pizza Runner
<img width="1080" height="1080" alt="image" src="https://github.com/user-attachments/assets/c571cbd9-b8d0-423a-91cb-f03bb4c2a13d" />

## Table of Contents
- [Overview](#overview)
- [Business Context](#business-context)
- [Data Schema](#data-schema)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Data Cleaning & Preparation](#data-cleaning--preparation)
- [Business Questions & Solutions](#business-questions--solutions)
- [Key Insights & Recommendations](#key-insights--recommendations)

## Overview
This project is part of the 8-Week SQL Challenge by Danny Ma. Case Study #2 focuses on Pizza Runner, a mock retro-themed pizza delivery startup. The objective is to clean raw operational data and write end-to-end SQL queries to optimize delivery logistics, analyze runner performance, and evaluate customer ordering behavior.

## Business Context
To scale operations beyond a single location, Pizza Runner collects transactional data on orders, deliveries, and runner logistics. However, the raw database contains formatting inconsistencies, missing values, and unstructured text fields that prevent immediate analysis.

This project addresses two core priorities:
1. Data Cleaning & Transformation: Standardizing data types, stripping text units from numerical metrics, and handling NULL values.
2. Operational Analytics: Querying cleaned datasets to deliver actionable insights on delivery efficiency, customer preferences, and product customization.

## Data Schema
All tables sit within the pizza_runner schema across six relational tables:
- customer_orders: Order-level records, including customer IDs, pizza selections, exclusions, extras, and order timestamps.
- runner_orders: Delivery metrics, including assigned runners, pickup times, travel distances, duration, and cancellation records.
- runners: Registration and onboarding dates for delivery runners.
- pizza_names: Maps pizza IDs to their respective menu names.
- pizza_recipes: Defines the default topping combinations for each pizza type.
- pizza_toppings: Serves as the lookup table for topping names and IDs.

## Entity Relationship Diagram
<img width="520" height="230" alt="image" src="https://github.com/user-attachments/assets/d0096130-fd05-4bbf-a98a-d64493fe1010" />

## Data Cleaning & Preparation
---

## Business Questions & Solutions
---

## Key Insights & Recommendations
---


