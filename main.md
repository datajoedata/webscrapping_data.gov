 #webscrapping #beautifulsoup4 #python #CKAN #opendata #USA
# Scrapping data from data.gov

//////////////////////////////////////// 

![image](https://github.com/datajoedata/data_extraction_nasa_hackaton2023/assets/116616136/d3033658-8818-4825-a329-1c7545e5a894)

//////////////////////////////////////// 

![image](https://github.com/datajoedata/data_extraction_nasa_hackaton2023/assets/116616136/7c602d68-211e-4185-a645-c2f02108651a)

//////////////////////////////////////// /////////////////////////////////////////////////
# 1. Context:


## 1.1 Project goal: Extract all datasets currently available on data.gov, along with their respective metadata.  

## 1.2 But for starters, what is the data.gov?  
"The United States government's open data website. It provides access to datasets published by agencies across the federal government. Data.gov is intended to provide access to government open data to the public, achieve agency missions, drive innovation, fuel economic activity, and uphold the ideals of an open and transparent government."

## 1.3 The [data.gov](https://data.gov) website is based on CKAN. But what is CKAN?
CKAN is an open-source DMS (data management system) for powering data hubs and data portals. "CKAN makes it easy to publish, share and use data."

## 1.4 CKAN provides us an API that have inumerous methods such as: 

   Get JSON-formatted lists of a site’s datasets, groups or other CKAN objects: 

        http://demo.ckan.org/api/3/action/package_list  (action that returns all datasets)

        http://demo.ckan.org/api/3/action/group_list    (returns a list of groups)
        
        http://demo.ckan.org/api/3/action/package_show  (...) 
        
 ((in CKAN documentation they say packages refer to datasets))   
    
//////////////////////////////////////// /////////////////////////////////////////////////

## 2. After spending almost 3 days trying to make "package_list" method work...

Found out that many people were having problems using package_list method: 

I began to suspect that there was an issue with their server. As I kept searching for answers, I discovered that the 'api/3/action/package_list' command is blocked on the DATA.GOV website. Consequently, I had to find an alternative solution.  

![image](https://github.com/datajoedata/data_extraction_nasa_hackaton2023/assets/116616136/4130a549-6f73-4a14-a25d-191f4016998a)  

  
![image](https://github.com/datajoedata/data_extraction_nasa_hackaton2023/assets/116616136/bd7aaa71-9340-4664-a60e-ff098bea3f7b)  
//////////////////////////////////////// /////////////////////////////////////////////////  



## 2.1 If the packages_list method is blocked, our only alternative was to use the "package_show" as described below:

http://demo.ckan.org/api/3/action/package_show?id=adur_district_spending

![image](https://github.com/datajoedata/data_extraction_nasa_hackaton2023/assets/116616136/294275b0-bf3c-4384-a1f0-da7dae5a9f2e)









//////////////////////////////////////// /////////////////////////////////////////////////
# PART II:
## 3. ALTERNATIVE WAY OF GETTING METADATA FOR ALL DATASETS: 
Another approach to this webscrapping was using beautiful soup to scrape all dataset names and then, ride requests through pagination name, as they have a link to  
 
 3.1 Get a list of all dataset names. 
 3.2 Use the get action API in the target API endpoint to retrieve metadata about each dataset.
 3.3 Organize them in a tabular conformation

### Getting the list of dataset names: 

## Python Code

```python
import requests
from bs4 import BeautifulSoup
import csv
import concurrent.futures

# Base URL of the web page containing dataset names
base_url = "https://catalog.data.gov/dataset"

# Function to scrape a single page and write dataset names to a CSV file
def scrape_page(page, csv_writer):
    # Construct the URL for the current page
    url = f"{base_url}?page={page}"

    try:
        # Send an HTTP GET request to the URL with a user-agent header
        response = requests.get(url, headers={'User-Agent': 'Mozilla/5.0'})
        response.raise_for_status()  # Raise an exception for HTTP errors

        # Parse the HTML content of the page
        soup = BeautifulSoup(response.text, 'html.parser')

        # Here I used a web development tools in your web browser to inspect the web page's HTML structure. You can typically do this by right-clicking on the page and selecting "Inspect" or pressing Ctrl + Shift + I or Cmd + Option + I on your keyboard. Dataset titles Mention that you used web development tools in your web browser to inspect the web page's HTML structure. You can typically do this by right-clicking on the page and selecting "Inspect" or pressing Ctrl + Shift + I or Cmd + Option + I on your keyboard.
        dataset_title_elements = soup.select('h3.dataset-heading a[href^="/dataset/"]')

        # Extract and write dataset titles to the CSV file
        dataset_names = [element.text.strip() for element in dataset_title_elements]
        csv_writer.writerows([[name] for name in dataset_names])

        return dataset_names
    except Exception as e:
        print(f"Failed to retrieve data for page {page}. Error: {e}")
        return []

# Set the total number of pages
total_pages = 11826

# Create an empty list to store the dataset names
dataset_names_list = []

# Set the checkpoint page number (the page to start from in case of a crash)
checkpoint_page = 1  # Change this to the last successfully scraped page

# Define the CSV file name
csv_filename = "dataset_names.csv"

# Open the CSV file in append mode to continue writing
with open(csv_filename, "a", newline="", encoding="utf-8") as csvfile:
    csv_writer = csv.writer(csvfile)

    # Only write the header row if starting from the first page
    if checkpoint_page == 1:
        csv_writer.writerow(["Dataset Names"])

    # Number of pages to scrape before pausing
    pages_to_scrape_before_pause = 7000
    # Time to pause (in seconds)
    pause_time = 1

    # Parallelize the scraping process using ThreadPoolExecutor
    with concurrent.futures.ThreadPoolExecutor(max_workers=10) as executor:
        # Use executor.map to apply the scrape_page function to each page concurrently
        results = executor.map(lambda page: scrape_page(page, csv_writer), range(checkpoint_page, total_pages + 1))

        # Iterate through the results and extend the dataset_names_list
        for result in results:
            dataset_names_list.extend(result)

print(f"Scraping completed. Dataset names saved to {csv_filename}")


        # Iterate through the results and extend the dataset_names_list
