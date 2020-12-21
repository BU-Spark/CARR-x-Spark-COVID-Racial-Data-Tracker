## Scraper:
Scrapes the commit history using BeautifulSoup for each file and collects the necessary date and commit information. The commits are used to construct the URLs for each raw data file as seen below:
`'https://raw.githubusercontent.com/nychealth/coronavirus-data/[commit_goes_here]/by-race.csv'`

`crawler(URL)`: takes in `URL`, which corresponds to the first page on the commit history of one of the files, and crawls through each "next" page of the history, collecting each link into a list.

`commitChecker(commit, URL)`: ensures each passed in commit corresponds to a working URL for the raw datafile. If it does not (i.e. if it raises an HTTP error), the commit is omitted from the list of commits, as well as the corresponding date in the `masterDays` list.

Each collected link is iterated through and all dates and commit addresses are scraped from each page into two lists: `masterDays` and `masterCommits`. Each list is then parsed for the necessary text attributes. The `masterCommits` list is cleaned using the `commitChecker(commit, URL)` function.

The dates and commits are both iterated through and a pandas data frame is created for each raw datafile. Columns for each date and commit are added and appended to the `masterData` and `confirmedProbableData` data frames, which are then loaded into CSVs.

## Data Quality Checker:
`convert_date(d)`: Helper function for data cleaning. Converts dates scraped from GitHub (in the format "Commits on Nov 8, 2020") into a datetime object in ISO format.

The cumulative datasets for confirmed probable deaths and the master dataset are imported as `nycFatals` and `nycMaster`, respectively.

The columns are renamed to improve readability and the date columns are converted using `convert_date(d)`. Separate data frames are created for the case counts, hospitalization counts, and death counts.

Each row corresponds to one date and each column corresponds to the counts for each race group (i.e for cases, the columns are `Cases API`, `Cases Black`, `Cases Latinx`, and `Cases White`).

Another data frame is created to determine the amount each column increased or decreased in all three data frames. If the column decreased, that cell will be orange.

If it had decreased by more than 100, then the cell would be red. For the `%OfSelf` checks, a new data frame is created.

Each column is calculated by applying the `.pct_change()` function from pandas, and multiplying each value by 100. The data frame is then filtered to today's date, and each cell is highlighted orange if it decreased, and red if it decreased by more than 50%.

For the `%OfTotal` checks, a data frame is created in which each column is calculated by dividing each column in each of the three separate data frames (cases, hospitalizations, and deaths), by the sum of each row, which is the total for that date.

Then, the `.pct_change()` function is applied and each column is multiplied by 100 to get the change in percent of the total for each category. The results are filtered by today's date, and each cell is highlighted yellow if the value increased between 10 and 25%, orange if it increased between 25 and 50%, and red if it increased by more than 50%.

The change in the percent of deaths that NYC reports race data is also checked. 
NYC only reports data pending/unknown race for their confirmed and probable death dataset, so for now, all that can be checked is the change for death data.

The numbers for probable deaths unknown, probable deaths data pending, confirmed deaths unknown, and confirmed deaths data pending are added, and this is divided by the sum for each date's row in the deaths data frame. This is then displayed in the results, alongside the mean of how much NYC usually reports. 
The number of deaths, cases, and hospitalizations as a percentage of the total population of NYC, as well as NYS, are calculated.

To visualize the data, a line graph of the cumulative cases for each of the four race/ethnicity groups that NYC currently reports data on is generated.

Two more line graphs comparing the percent change of self values for Black vs. White cases and comparing the changes of self for API vs. Latinx cases are generated as well.