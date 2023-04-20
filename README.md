# Summary of the Script

This script is designed to collect, clean, and analyze data related to Dynobil, a fictional car inspection company. The main goal is to gather information about Dynobil locations and their competitors, as well as other relevant data such as notary offices and car dealerships. The script then calculates the distance between these points of interest and combines this information with demographic data from a dataset called endeksa. The final output is a cleaned and organized dataset ready for further analysis.

## Key Functions

- `data_collection_manual(searched_item, how_many_item)`: Collects data for a specific search item manually from Google Places API.
- `data_collection_total(searching_key, how_many_results)`: Collects data for a specific search item for every location mentioned in the "endeksa" CSV file.
- `api_data_adjustment(df)`: Adjusts the raw API data by extracting latitude and longitude, removing unnecessary columns, and formatting the data for further analysis.
- `remove_unrelated(df, searched_word1, searched_word2)`: Removes unrelated results from the dataframe based on the searched words.
- `same_loc_cleaner(changed_df, according_to)`: Cleans data by removing duplicate locations from different dataframes.
- `closest_calculation(origin, df_closest)`: Calculates the closest location between two dataframes.
- `which_are_duplicated(df)`: Identifies which rows are duplicated in a dataframe.

## Workflow

1. The script imports necessary libraries and reads in the base file (endeksa.csv).
2. Data is collected for Dynobil locations, competitors, notary offices, and car dealerships using the `data_collection_total` function.
3. The raw data is adjusted using the `api_data_adjustment` function and cleaned with the `remove_unrelated` and `same_loc_cleaner` functions.
4. The script calculates the closest distances between points of interest using the `closest_calculation` function.
5. Demographic data from endeksa is cleaned and combined with the collected data.
6. The final dataset is prepared for further analysis by removing unnecessary columns and encoding categorical variables using dummy variables.

The script effectively collects, processes, and organizes data related to Dynobil and its competitors, creating a useful dataset for further exploration and analysis.
