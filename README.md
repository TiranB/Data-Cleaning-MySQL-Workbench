# Data-Cleaning-MySQL-Workbench
This project demonstrates how to clean and prepare raw sales data using SQL queries in MySQL Workbench. 
The goal is to transform inconsistent and incomplete data into a structured, analysis-ready format.

## Data
This data tracks layoff events by companies, and includes contextual info like how much funding the company has raised, what industry they’re in, and what stage they're at
- Source:-
- Content:-
  | **Column Name**         | **Description**                                                                 |
|-------------------------|---------------------------------------------------------------------------------|
| `company`               | The name of the company where layoffs occurred (e.g., Atlassian, SiriusXM)     |
| `location`              | The city where the company or layoffs were primarily located                   |
| `industry`              | The sector or domain the company operates in (e.g., Media, Other)              |
| `total_laid_off`        | The number of employees laid off                                               |
| `percentage_laid_off`   | Proportion of workforce laid off, expressed as a decimal (e.g., 0.05 = 5%)     |
| `date`                  | The date when the layoffs were reported or took place                          |
| `stage`                 | The funding stage of the company (e.g., Post-IPO = after initial public offering) |
| `country`               | The country where the company is based                                         |
| `funds_raised_millions`| Total funds raised by the company in millions of dollars                       |

