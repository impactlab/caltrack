
# CalTRACK Beta Test Data Preparation Guidelines

Data Cleaning and Quality Checks for the CalTRACK Beta Test will consist of three main tasks

1. Explore raw data sources with non-standard summary statistics
2. Perform data cleaning and integration procedures on raw data
3. Calculate and report prepared data summary statistics

=======


Data Cleaning and Munging
---

A number of cleaning steps are necessary to use the raw data.

### Consumption Data Cleaning

**Elec Traces**

* Remove duplicates
  * No duplicates in monthly data
* Add leading zeroes to `PREM_ID` where missing (length == 8 or 9) in monthly traces.

* 7702 elec monthly traces with unique SA_ID

**Gas Traces**

* Remove duplicates
  * No duplicates in monthly data
* Add leading zeroes to `PREM_ID` where missing (length == 8 or 9) in monthly traces.

* 6919 gas monthly traces with unique SA_ID

**Elec Xrefs**

* Add leading zeroes to `char_prem_id` where missing (length == 8 or 9). Appear to have been lost during PGE's export process 

**For Caltrack data analysis: traces with `is_net_metered=Y` should be excluded from analysis.**

**Gas Xrefs**

* Concatenate two gas xref files

* Add leading zeroes to `char_prem_id` where missing (length == 8 or 9). Appear to have been lost during PGE's export process

**Projects**

* Concatenate three project files together

* Estimate project dates using Caltrack procedure described below

* Assign each project a `char_prem_id` using both elec and gas xref file 

    * Both `Gas Service ID` and `Electric Service ID` always map to the same `char_prem_id`

* Remove any project that did not get a `char_prem_id`

    * 4220 of 4358 projects received a prem id.

* Merge projects with identical `char_prem_id`

    * Earliest Work Start Date in merged project
    * Latest Work Finish Date in merged project
    * 4220 projects before merging premids
    * 4206 projects after merging premids

* Remove `Gas Service ID` and `Electric Service ID` from output. (Misleading since matching should be done with `char_prem_id` after cleaning)


### *Deduplication*

- If a home appears multiple times within a project database, and the project dates are the same the most complete record for that home will be the record used in CalTRACK

- If a home appears multiple times within a project database and the project dates differ because there are multiple measures installed associated with the same incentive program, the start date of the intervention will be the earliest of the project start dates across projects and the end date for the intervention will be the latest of the project end dates

- There are a small number of duplicate traces – consumption traces with unique SAs (though identical SPIDs) and identical consumption data over the time interval.

- If two duplicate records have identical consumption traces and date ranges, drop one at random

- If two duplicate records have identical consumption traces but different date ranges select the more complete record having more dates. If the dates are contiguous, or there are overlapping dates with the same usage values, combine the two traces into a single trace.

- If they have the same date ranges, but different usage values, the project is flagged and the record is excluded from the sample.

### Creating Work Start and Work End dates from raw project data

The dates for CalTRACK Beta Test come from the following Final CalTRACK files provided by Build it Green:

`CalTRACK (AHU) from 1_1_14__6_30_15_v2_FINAL_090816.csv`

`CalTRACK (AHU) from 7_1_15__6_30_16_v2_FINAL_090816.csv`

`CalTRACK (AHUP) from 1_1_14__6_30_15_v2_FINAL_090816.csv`


The following rules are used for determining `work start dates`:

- From the updated ‘AHUP’ file “CalTRACK (AHUP) from 1_1_14__6_30_15_v2_FINAL_090816”
	- Column G – “Notice to Proceed Issued” (Best Proxy for ‘Work Start’)

- From the updated ‘AHU with correlated XML files’ file “CalTRACK (AHU) from 1_1_14__6_30_15_v2_FINAL_090816”
 	- Column F – “Initial Approval Date” (Best Proxy for ‘Work Start’)

- From the updated ‘AHU Control Group (no XML files)’ file “CalTRACK (AHU) from 7_1_15__6_30_16_v2_FINAL_090816”
 	- Column F – “Initial Approval Date” (Best Proxy for ‘Work Start’)

 The following rules are used for determining `work end dates`:

- From the updated ‘AHUP’ file “CalTRACK (AHUP) from 1_1_14__6_30_15_v2_FINAL_090816”
	- Column H – “Full Application Started” (Best Proxy for ‘Work Finished’)
	- Please Note – Column J – “Full Application Submitted” can represent a better proxy for ‘Work Finished’, if the project was not returned for correction(s) during the QA review process (i.e., if there is a date in Column I – “Full Application Returned”, it got returned for correction). If a project gets returned for correction, then Column J’s “Full Application Submitted” date becomes, effectively, a “Full Application ‘Re-submitted’”, and no longer represents a good proxy for ‘Work Finished’ as it is +1-10 Days (or more) removed at that point.

- From the updated ‘AHU with correlated XML files’ file “CalTRACK (AHU) from 1_1_14__6_30_15_v2_FINAL_090816”
	- Column G – “Initial Submission Date” (Best Proxy for ‘Work Finished’)

- From the updated ‘AHU Control Group (no XML files)’ file “CalTRACK (AHU) from 7_1_15__6_30_16_v2_FINAL_090816”
	- Column G – “Initial Submission Date” (Best Proxy for ‘Work Finished’)


### *Missing Values & Imputation*

**Weather**

- Hourly

    - Hourly weather data from GSOD will not be imputed

    - Days with more than 5 contiguous hours of missing data will be thrown out

- Daily

    - GSOD daily averages will not be imputed

    - Months with more than than 3 missing days will be thrown out

**Usage**

- Missing values where the cumulative value is in the following period, the cumulative number of days between the two periods will be used to generate the UPD for that period

- Missing usage values with no cumulative amount in the following period will be counted against data sufficiency requirements

- Homes with Net Metering will be dropped from the analysis

### *Estimated values & deletion*

- Estimated usage data will be used for estimation, but estimation flags will be added to the post-estimation

### *Extreme values*

**Usage**

- Negative values or values with reverse direction of flow will be treated as missing and count against sufficiency criterion. The account will also be flag within CalTRACK for possible net metering if it does not currently contain a net metering flag

- It is assumed at all IOUs comply with DASMMD.

- AMI data will have the DASMMD pass/fail criterion rerun, with failing values coded as missing. ((highest peak - third highest peak)/third highest peak) <= 1.8

**Project Data**

- Extreme project lengths (gap between project start date and project end date longer than 3 months) will be treated as true and impact estimation only through data sufficiency requirements.

- Files without project start dates are thrown out

### *Sum Check*

If both monthly and AMI data are available for a home, CalTRACK will run a sumcheck and use the DASMMD criterion for pass/fail. If it fails, the home is flagged and treated as having missing usage data so no estimation is run on it.

### *Miscoded values*

- Miscoded strings in project data will be deduplicated and matched (fuzzily) to closest value

### *Miscoded dates*

- Implausible day values (>31) will be coded as the beginning of month if project start date and end of month if project end date so that the entire month included in the intervention window

- Implausible month and year values will be flagged and that home not included in estimation.

### *Data sufficiency*

**Usage (Monthly)**

- 12 complete months pre-retrofit for monthly billing data to qualify for estimation or 24 months with up to 2 missing values from different, non-contiguous months

- Post retrofit data sufficiency for estimation will be dealt with in post-estimation model fit criterion

- Total annual savings estimates will require 12 months post-retrofit

**Usage (AMI)**

- 12 months pre-retrofit

- Post retrofit data sufficiency for estimation will be dealt with in post-estimation model fit criterion

- Total annual savings estimates will require 12 months post-retrofit

**Weather**

- There should not be problems with data sufficiency for weather

### *Project or Home Characteristics*

- Exclude homes with PV for whom solar production data is not available

### *Value Adjustments (if values change during performance period)*

### **Usage**

- Use most up-to-date meter read for

- Log prior values and prior estimates

**Project data**

- Use most up to date values for estimation

**Weather data**

- Use most recent daily weather value for estimation

# Data Integration

- Matching project data
	- Matching will be done using the cross-reference files contain the sa_id and sp_id mapping between the project data id fields and the consumption data id fields.

| Column Name | Description |
| --- | --- |
| sa_id | Corresponds to `Electric Service ID` or `Gas Service ID` in projects file |
| sp_id | Corresponds to `SPID` in electricity consumption file and `Service Point` in natural gas file |


- Matching weather stations to projects
	- Weather station mapping will be done using the 86 station standard mapping of zip code to CZ2010 weather files provided in the `/data-source` directory



- Unmatched data
	- Projects that are unmatched to usage data will be listed in CalTRACK for data integrity reporting, but will not be included in any estimation procedures and will not have estimated savings 


## Prepared Data Summary Statistics for Comparison

To ensure that beta testers are working with the same datasets and that we are characterizing the datasets in a way that is helpful for other members of the technical working group as well as the general public, each beta tester will generate a set of summary statistics and perform a set of data quality checks that can be shared with the larger group through csvs saved to this repository. There will be four cleaned data reports generated by each beta tester: a project data summary file, an hourly electric summary file, a daily gas summary file, and data integration summary file. Each file will be a .csv and will have the following general format:

| Summary Stat | Value |
| --- | --- |
| Number of Observations | 4321 |
| Number of Fields | 4 |
| Min Work Start Date | Jan, 21, 2014 |
| Max Work Start Date| June 23, 2015| 

The specifics for each summary file are below:

### Combined Project Data Summary File

**Output Filename: `project_data_summary_NAME_OF_TESTER.csv`**

Apply project filtering rules, then compute the following statistics on the filtered list of projects:

#### Included Summary statistics

- Total number of records
- Number of unique project electric SAIDs
- Top 10 zip codes by count 
	- Sort by descending counts,
	- For identical counts, sort by ascending zip code
- Bottom ten zip codes by count
	- Sort by ascending counts,
	- For identical counts, sort by ascending zip code
- Min Work Start Date
- Max Work Start Date
- Average Work Start Date
- Work Start Date 10th percentile value
- Work Start Date 20th percentile value
- Work Start Date 30th percentile value
- Work Start Date 40th percentile value
- Work Start Date 50th percentile value
- Work Start Date 60th percentile value
- Work Start Date 70th percentile value
- Work Start Date 80th percentile value
- Work Start Date 90th percentile value
- Min Work End Date
- Max Work End Date
- Average Work End Date
- Work End Date 10th percentile value
- Work End Date 20th percentile value
- Work End Date 30th percentile value
- Work End Date 40th percentile value
- Work End Date 50th percentile value
- Work End Date 60th percentile value
- Work End Date 70th percentile value
- Work End Date 80th percentile value
- Work End Date 90th percentile value

_Formatting_

- Truncate dates to the nearest day
- Format dates as YYYY-MM-DD


### Prepared Hourly Electricity Data Summary File

**Output Filename: `hourly_electricity_data_summary_NAME_OF_TESTER.csv`**
 
#### Included Summary statistics

- Total number of usage records
- Number of unique SPIDs
- Total number of missing hours across all usage records
- Total number of estimated hours across all usage records
- Number of net metering accounts dropped from sample
- Top 10 users by total use
- Bottom 10 users by total use
- Min Datetime stamp across all usage records
- Max Datetime stamp across all usage records
- Average Datetime stamp across all usage records
- Min use across all usage records
- Max use across all usage records
- Average use across all usage records
- Use across all usage records 10th percentile value
- Use across all usage records 20th percentile value
- Use across all usage records 30th percentile value
- Use across all usage records 40th percentile value
- Use across all usage records 50th percentile value
- Use across all usage records 60th percentile value
- Use across all usage records 70th percentile value
- Use across all usage records 80th percentile value
- Use across all usage records 90th percentile value

### Prepared Daily Gas Data Summary File

**Output Filename: `daily_gas_data_summary_NAME_OF_TESTER.csv`**
 
#### Included Summary statistics

- Total number of records
- Number of unique SPIDs
- Total number of missing days across all usage records
- Total number of estimated hours across all usage records
- Top 10 users by total use
- Bottom 10 users by total use
- Min Datetime stamp across all usage records
- Max Datetime stamp across all usage records
- Average Datetime stamp across all usage records
- Min use across all usage records
- Max use across all usage records
- Average use across all usage records
- Use across all usage records 10th percentile value
- Use across all usage records 20th percentile value
- Use across all usage records 30th percentile value
- Use across all usage records 40th percentile value
- Use across all usage records 50th percentile value
- Use across all usage records 60th percentile value
- Use across all usage records 70th percentile value
- Use across all usage records 80th percentile value
- Use across all usage records 90th percentile value

### Data Integration Summary File

**Output Filename: `data_integration_electricity_summary_NAME_OF_TESTER.csv`**
 
#### Included Summary statistics

- Number of unique SAIDs in joined data
- Number of unique SPIDs in hourly electricity data
- Number of unique SPIDs in monthly electricity data
- Number of unique SPIDs in the electricity cross reference dataset
- Number of unique SPIDS in the gas cross reference dataset
- Number of unique SAIDs in the electricity cross reference dataset
- Number of unique SAIDs in the gas cross reference dataset
- % Records successfully matched between hourly electricity data to project data
	- Frequency counts of invalid rows from missing sa_id
	- Frequency counts of invalid rows from missing sp_id
- % Records successfully matched between daily gas data to project data
	- Frequency counts of invalid rows from missing sa_id
	- Frequency counts of invalid rows from missing sp_id
- % Records successfully matched between hourly electricity data to project data
	- Frequency counts of invalid rows from missing sa_id
	- Frequency counts of invalid rows from missing sp_id
- % Records successfully matched between monthly electricity data to project data
	- Frequency counts of invalid rows from missing sa_id
	- Frequency counts of invalid rows from missing sp_id





