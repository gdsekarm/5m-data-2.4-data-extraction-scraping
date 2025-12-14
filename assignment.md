# Assignment

## Brief

Write the Python codes for the following questions.

## Instructions

Paste the answer as Python in the answer code section below each question.

### Question 1

Question: The scraping of `https://www.scrapethissite.com/pages/forms/` in the last section assumes a hardcoded (fixed) no of pages. Can you improve the code by removing the hardcoded no of pages and instead use the `»` button to determine if there are more pages to scrape? Hint: Use a `while` loop.

```python
def parse_and_extract_rows(soup: BeautifulSoup):
    """
    Extract table rows from the parsed HTML.

    Args:
        soup: The parsed HTML.

    Returns:
        An iterator of dictionaries with the data from the current page.
    """
    header = soup.find('tr')
    headers = [th.text.strip() for th in header.find_all('th')]
    teams = soup.find_all('tr', 'team')
    for team in teams:
        row_dict = {}
        for header, col in zip(headers, team.find_all('td')):
            row_dict[header] = col.text.strip()
        yield row_dict
```

Answer:

```python
import requests
from bs4 import BeautifulSoup

def parse_and_extract_rows(soup: BeautifulSoup):
    """
    Extract table rows from the parsed HTML.
    Args:
        soup: The parsed HTML.
    Returns:
        An iterator of dictionaries with the data from the current page.
    """
    header = soup.find('tr')
    # Handle cases where the table might be empty or missing
    if not header:
        return []
        
    headers = [th.text.strip() for th in header.find_all('th')]
    teams = soup.find_all('tr', 'team')
    
    for team in teams:
        row_dict = {}
        for header, col in zip(headers, team.find_all('td')):
            row_dict[header] = col.text.strip()
        yield row_dict

def scrape_all_pages():
    # Base domain is needed to construct the full URL from relative hrefs
    base_domain = "https://www.scrapethissite.com"
    current_url = "https://www.scrapethissite.com/pages/forms/"
    
    all_teams = []
    page_count = 1

    while True:
        print(f"Scraping Page {page_count}...")
        
        response = requests.get(current_url)
        soup = BeautifulSoup(response.content, 'html.parser')
        
        # 1. Extract data from the current page
        # extending the list with the generator results
        all_teams.extend(parse_and_extract_rows(soup))
        
        # 2. Look for the "Next" button (»)
        # The site uses aria-label="Next" for the anchor tag containing the » symbol
        next_button = soup.find('a', attrs={'aria-label': 'Next'})
        
        # 3. Check if the button exists and has an href
        if next_button and 'href' in next_button.attrs:
            # The href is relative (e.g., /pages/forms/?page_num=2), so we append it to the domain
            next_link = next_button['href']
            current_url = base_domain + next_link
            page_count += 1
        else:
            # If no "Next" button is found, we have reached the last page
            print("No next page found. Scraping complete.")
            break
            
    return all_teams

# --- Execution ---
if __name__ == "__main__":
    data = scrape_all_pages()
    print(f"\nTotal rows extracted: {len(data)}")
    
    # Print the first 3 rows as a sample
    print("Sample data:")
    for row in data[:3]:
        print(row)
```

## Submission

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.
