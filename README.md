# README

## Overview

This project is a specialized web scraping and data processing pipeline designed to extract, clean, and structure information related to upcoming election dates, filing start and end dates, and office positions in the USA government. The pipeline consists of several stages, including web scraping, data filtering, and data processing, ultimately storing the structured data in JSON or CSV format.

## Project Structure

- **web_crawler_and_scraper**: Contains scripts for web scraping and initial data cleaning.
- **url_filter_and_extraction**: Contains scripts for scoring and filtering data, as well as extracting relevant information.
- **data_processor**: Contains scripts for processing extracted data, validating and structuring it, and storing it in structured formats.

## System Flow

1. **Get URLs**:

   - **Input**: Prompt and keywords.
   - **Process**: Use Google programmable search engine's Custom Search JSON API and Tavily's Search API to get the URLs and filter them on the basis of relevance to local government positions using descriptions generated by Tavily's API.
   - **Output**: A list of filtered URLs.

2. **Web Scraping and Data Cleaning**:

   - **Input**: List of URLs.
   - **Process**: Use playwright to scrape the filtered URLs and store the raw HTML content in the `scraped_files` directory. Use BeautifulSoup to clean the data and store the cleaned html data in the `formatted_files` directory.
   - **Output**: Cleaned html data.

3. **Data Extraction**:

   - **Input**: Relevant URLs.
   - **Process**: Extract unstructured data from the cleaned html data using OpenAI's LLM that is relevent to local government positions and store the data in the `shared_data/unstructured_data` directory.
   - **Output**: Unstructured data.

4. **Data Processing and Validation**:

   - **Input**: Unstructured data.
   - **Process**: Extract roles and positions using OpenAI's LLM. Validate and format positions to ensure they follow the format "role: [local government position] of region: [county/municipality/township]". Use LLM to extract structured data for each position, including details like position title, description, election dates, and filing windows. Use pydantic to validate the structured data and output the data in JSON format.
   - **Output**: Structured data in JSON format.

5. **Structuring and Storage**:

   - **Input**: Structured data.
   - **Process**: Store data in JSON or CSV format, organized by role and region.
   - **Output**: Data files ready for merging and analysis.

6. **Merging**:
   - **Input**: Structured data csv files.
   - **Process**: Merge the data from different csv files into a single file by role and region by consolidating the data.
   - **Output**: Merged data in CSV format as `final_merged_data.csv`.

## Usage

### Step 1: Get URLs

- Set the `primary_urls` variable in the `get_urls.py` script to the URLs you want to scrape.
- Set the `prompt` variable in the `get_urls.py` script to the prompt you want to use to get the URLs.
- Set the `keywords` variable in the `get_urls.py` script to the keywords you want to use to get the URLs.

Run the `get_urls.py` script:

```bash
$ python ./web_crawler_and_scraper/scrape_urls.py get_urls
```

- The filtered URLs are stored in `./urls/initial_filtered_urls.txt`.

### Step 2: Web Scraping

To start the web scraping process, run the following command:

```bash
$ python ./web_crawler_and_scraper/scrape_urls.py scrape
```

- This script will scrape the filtered URLs and store the raw HTML content in the `scraped_files` directory.
- The cleaned and formatted files will be stored in the `formatted_files` directory inside `url_filter_and_extraction`.
- You can set the number of pages to scrape by changing the `pages_to_scrape` variable in the `scrape_urls.py` script.

### Step 3: Scoring and Filtering

Run the `score_and_filter.py` script in the `url_filter_and_extraction` directory:

```bash
$ python ./url_filter_and_extraction/score_and_filter.py [--count <number>]
```

- If `--count` is specified, only that many files will be processed.
- This script scores each file based on its content. Files with a score greater than a cutoff score are moved to the `filtered` directory.
- You can set the cutoff score by changing the `CUTOFF_SCORE` variable in the `score_and_filter.py` script.

### Step 4: Data Extraction

Run the `extraction.py` script in the `url_filter_and_extraction` directory:

```bash
$ python ./url_filter_and_extraction/extraction.py [--count <number>]
```

- This script extracts important information from the filtered files and stores them in the `shared_data/unstructured_data` directory, along with their URLs.
- You can adjust the prompt used for extraction by changing the `generate_extraction_prompt` function in the `extraction.py` script to better suit your needs.

### Step 5: Data Processing

Finally, process the extracted data using the `processor.py` script in the `data_processor` directory:

```bash
$ python ./data_processor/processor.py [--count <number>]
```

- The `use_llm_for_extraction` function in the `llmPrompts.py` script uses LLM to extract the roles from the filtered files and uses the `use_llm_for_position_data` function to fetch and save data for each role.
- You can adjust the prompt used for extraction by changing the system prompts used in the `llmPrompts.py` file in the `data_processor` directory to better suit your needs.
- The structured data is validated using a pydantic schema, and stored in a structured format either in JSON or CSV in the `shared_data/structured_data_*` directories.
- If using JSON, files belonging to the same role are stored in the same folder.
- If using CSV, data corresponding to the same role is stored in the same CSV file.

### Step 6: Merging

Run the `merge_csv_files.py` script in the `data_processor` directory:

```bash
$ python ./data_processor/merge_csv_files.py
```

- The merged data is stored in the `final_merged_data.csv` file.

## Requirements

Ensure you have all necessary dependencies installed. You can manage Python packages using Poetry as specified in your environment setup.

## Diagrams included

To better understand the system, consider the following diagrams:

1. **System Architecture Diagram**: Illustrates the components of the system and their interactions, including the flow of data from web scraping to storage.

2. **Data Flow Diagram**: Shows the flow of data through the system, highlighting key processes like validation and extraction.

3. **Sequence Diagram**: Details the sequence of operations for a single URL, from initial scraping to final data storage.

## Notes

- Edit the `.env.sample` file to include your OpenAI API key and save it as `.env`.
- The project is designed to run on Windows using Git Bash as the terminal. Adjust paths and commands if using a different setup.
- The files in the `shared_data` directory are not tracked by git to avoid large file storage issues.
