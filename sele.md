I'll teach you Selenium in Python with practical examples for all the topics you mentioned. Let's go through each concept step by step.

1. Setup and Installation

First, install the required packages:

```bash
pip install selenium pandas openpyxl webdriver-manager
```

2. Headless Launch and Options Arguments

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

# Configure Chrome options
chrome_options = Options()

# Headless mode
chrome_options.add_argument("--headless")  # Run in background
chrome_options.add_argument("--no-sandbox")
chrome_options.add_argument("--disable-dev-shm-usage")
chrome_options.add_argument("--disable-gpu")  # Disable GPU acceleration
chrome_options.add_argument("--window-size=1920,1080")  # Set window size
chrome_options.add_argument("--disable-blink-features=AutomationControlled")
chrome_options.add_argument("--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36")

# Additional useful options
chrome_options.add_argument("--disable-extensions")
chrome_options.add_argument("--disable-popup-blocking")
chrome_options.add_argument("--disable-notifications")
chrome_options.add_experimental_option("excludeSwitches", ["enable-automation"])
chrome_options.add_experimental_option('useAutomationExtension', False)

# Initialize driver with options
driver = webdriver.Chrome(
    service=Service(ChromeDriverManager().install()),
    options=chrome_options
)

driver.get("https://www.google.com")
print(f"Page title in headless mode: {driver.title}")
```

3. Sending Hotkeys

```python
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.action_chains import ActionChains
import time

driver.get("https://www.google.com")

# Find search box
search_box = driver.find_element("name", "q")

# Method 1: Send keys with hotkeys
search_box.send_keys("Selenium Python")
search_box.send_keys(Keys.RETURN)  # Press Enter

# Method 2: Using ActionChains for complex hotkey combinations
time.sleep(2)
actions = ActionChains(driver)

# Ctrl + A (select all)
actions.key_down(Keys.CONTROL).send_keys('a').key_up(Keys.CONTROL).perform()

# Ctrl + C (copy)
time.sleep(1)
actions.key_down(Keys.CONTROL).send_keys('c').key_up(Keys.CONTROL).perform()

# Ctrl + V (paste)
time.sleep(1)
actions.key_down(Keys.CONTROL).send_keys('v').key_up(Keys.CONTROL).perform()

# Tab key
actions.send_keys(Keys.TAB).perform()

# Common hotkeys:
# Keys.ENTER, Keys.TAB, Keys.ESCAPE
# Keys.CONTROL, Keys.ALT, Keys.SHIFT
# Keys.ARROW_DOWN, Keys.ARROW_UP, etc.
```

4. Asserting Elements

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import unittest

class TestGoogleSearch(unittest.TestCase):
    
    def setUp(self):
        self.driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
        self.driver.get("https://www.google.com")
    
    def test_element_assertions(self):
        driver = self.driver
        
        # Assert page title
        assert "Google" in driver.title, f"Title doesn't contain Google. Actual: {driver.title}"
        
        # Assert element exists
        search_box = driver.find_element(By.NAME, "q")
        assert search_box.is_displayed(), "Search box is not visible"
        assert search_box.is_enabled(), "Search box is not enabled"
        
        # Assert element text
        search_box.send_keys("Selenium")
        search_box.submit()
        
        # Wait for results and assert
        wait = WebDriverWait(driver, 10)
        results = wait.until(EC.presence_of_element_located((By.ID, "search")))
        assert results.is_displayed(), "Search results not displayed"
        
        # Assert number of elements
        result_links = driver.find_elements(By.CSS_SELECTOR, "h3")
        assert len(result_links) > 0, "No search results found"
        
        # Assert specific text in element
        first_result = result_links[0].text
        assert "Selenium" in first_result, f"Expected 'Selenium' in result, got: {first_result}"
        
        # Soft assertions using try-except
        try:
            assert "Python" in driver.page_source
            print("Assertion passed: Python found in page source")
        except AssertionError:
            print("Assertion failed: Python not found")
    
    def tearDown(self):
        self.driver.quit()

# Run test
if __name__ == "__main__":
    unittest.main()
```

5. Running JavaScript

```python
# Execute JavaScript
driver.get("https://www.google.com")

# Simple JavaScript execution
title = driver.execute_script("return document.title;")
print(f"Page title from JS: {title}")

# Scroll to bottom
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")

# Scroll to specific element
element = driver.find_element(By.NAME, "q")
driver.execute_script("arguments[0].scrollIntoView(true);", element)

# Highlight element
driver.execute_script("arguments[0].style.border='3px solid red'", element)

# Get page dimensions
width = driver.execute_script("return window.innerWidth;")
height = driver.execute_script("return window.innerHeight;")
print(f"Viewport size: {width}x{height}")

# Change page attributes
driver.execute_script("document.body.style.backgroundColor = 'lightblue';")

# Complex JavaScript with return value
data = driver.execute_script("""
    var elements = document.querySelectorAll('a');
    var links = [];
    for (var i = 0; i < elements.length; i++) {
        links.push(elements[i].href);
    }
    return links;
""")
print(f"Found {len(data)} links")

# Async JavaScript (rarely needed)
result = driver.execute_async_script("""
    var callback = arguments[arguments.length - 1];
    setTimeout(function() {
        callback('Done after 2 seconds');
    }, 2000);
""")
print(result)
```

6. Open/Close/Move Tabs

```python
# Open new tab
driver.execute_script("window.open('https://www.google.com', '_blank');")

# Get current window handles
original_tab = driver.current_window_handle
all_tabs = driver.window_handles
print(f"Number of tabs: {len(all_tabs)}")

# Switch to new tab
driver.switch_to.window(all_tabs[1])
driver.get("https://www.github.com")
print(f"Second tab title: {driver.title}")

# Open another tab
driver.execute_script("window.open('https://www.python.org', '_blank');")
all_tabs = driver.window_handles

# Switch between tabs
driver.switch_to.window(all_tabs[2])
print(f"Third tab title: {driver.title}")

# Close current tab
driver.close()

# Switch back to original tab
driver.switch_to.window(original_tab)
print(f"Back to original: {driver.title}")

# Move tab to new window (workaround - create new window)
driver.execute_script("window.open('');")
new_window = driver.window_handles[-1]
driver.switch_to.window(new_window)
driver.get("https://www.stackoverflow.com")

# Close all tabs except the original
for handle in driver.window_handles:
    if handle != original_tab:
        driver.switch_to.window(handle)
        driver.close()
driver.switch_to.window(original_tab)
```

7. Upload File

```python
import os

driver.get("https://www.w3schools.com/howto/howto_html_file_upload_button.asp")

# Method 1: Direct send_keys to file input element
file_input = driver.find_element(By.CSS_SELECTOR, "input[type='file']")
file_path = os.path.abspath("test.txt")

# Create test file
with open(file_path, "w") as f:
    f.write("This is a test file for upload")

file_input.send_keys(file_path)
print("File uploaded successfully")

# Method 2: Using pyautogui for non-input file uploads
# import pyautogui
# upload_button = driver.find_element(By.ID, "upload-btn")
# upload_button.click()
# time.sleep(1)
# pyautogui.write(file_path)
# pyautogui.press('enter')

# Method 3: For multiple files
multiple_input = driver.find_element(By.CSS_SELECTOR, "input[multiple]")
file_paths = [
    os.path.abspath("file1.txt"),
    os.path.abspath("file2.txt")
]
multiple_input.send_keys("\n".join(file_paths))
```

8. Download File

```python
import os
import time

# Configure Chrome for automatic downloads
download_dir = os.path.join(os.getcwd(), "downloads")
os.makedirs(download_dir, exist_ok=True)

chrome_options = Options()
prefs = {
    "download.default_directory": download_dir,
    "download.prompt_for_download": False,
    "download.directory_upgrade": True,
    "safebrowsing.enabled": True,
    "profile.default_content_setting_values.automatic_downloads": 1
}
chrome_options.add_experimental_option("prefs", prefs)

driver = webdriver.Chrome(
    service=Service(ChromeDriverManager().install()),
    options=chrome_options
)

# Example: Download a file
driver.get("https://example.com/some-file.pdf")

# Method 1: Click download link
download_link = driver.find_element(By.LINK_TEXT, "Download")
download_link.click()

# Wait for download to complete
time.sleep(5)

# Check if file downloaded
files = os.listdir(download_dir)
if files:
    print(f"Downloaded files: {files}")

# Method 2: Direct download using requests (alternative)
import requests
response = requests.get("https://example.com/file.pdf")
with open(os.path.join(download_dir, "file.pdf"), "wb") as f:
    f.write(response.content)

# Method 3: Monitor download progress
import glob
import time

def wait_for_download(download_dir, timeout=30):
    seconds = 0
    while seconds < timeout:
        files = glob.glob(f"{download_dir}/*.crdownload")
        if not files:
            return True
        time.sleep(1)
        seconds += 1
    return False

# Use the monitor
wait_for_download(download_dir)
```

9. Write to Excel

```python
import pandas as pd
from openpyxl import Workbook, load_workbook
from openpyxl.styles import Font, PatternFill
import datetime

# Method 1: Using pandas (simplest)
def write_with_pandas(data, filename="output.xlsx"):
    df = pd.DataFrame(data)
    df.to_excel(filename, index=False, sheet_name="Web Scraping Data")
    print(f"Data written to {filename}")

# Sample data
scraped_data = {
    "Title": ["Selenium Python", "Web Automation", "Headless Browser"],
    "URL": ["https://selenium.dev", "https://python.org", "https://example.com"],
    "Timestamp": [datetime.datetime.now()] * 3,
    "Status": ["Success", "Success", "Failed"]
}

write_with_pandas(scraped_data)

# Method 2: Using openpyxl (more control)
def write_with_openpyxl(data, filename="detailed_output.xlsx"):
    wb = Workbook()
    ws = wb.active
    ws.title = "Scraped Results"
    
    # Add headers with styling
    headers = list(data.keys())
    header_font = Font(bold=True, color="FFFFFF")
    header_fill = PatternFill(start_color="366092", end_color="366092", fill_type="solid")
    
    for col, header in enumerate(headers, 1):
        cell = ws.cell(row=1, column=col, value=header)
        cell.font = header_font
        cell.fill = header_fill
    
    # Add data
    for row_idx, row_data in enumerate(zip(*data.values()), 2):
        for col_idx, value in enumerate(row_data, 1):
            ws.cell(row=row_idx, column=col_idx, value=value)
    
    # Auto-adjust column widths
    for column in ws.columns:
        max_length = 0
        column_letter = column[0].column_letter
        for cell in column:
            try:
                if len(str(cell.value)) > max_length:
                    max_length = len(str(cell.value))
            except:
                pass
        adjusted_width = min(max_length + 2, 50)
        ws.column_dimensions[column_letter].width = adjusted_width
    
    wb.save(filename)
    print(f"Excel file saved: {filename}")

write_with_openpyxl(scraped_data)

# Method 3: Append to existing Excel file
def append_to_excel(new_data, filename="output.xlsx"):
    try:
        # Try to load existing workbook
        book = load_workbook(filename)
        writer = pd.ExcelWriter(filename, engine='openpyxl')
        writer.book = book
        writer.sheets = {ws.title: ws for ws in book.worksheets}
        
        # Read existing data
        existing_df = pd.read_excel(filename, sheet_name="Web Scraping Data")
        
        # Append new data
        new_df = pd.DataFrame(new_data)
        combined_df = pd.concat([existing_df, new_df], ignore_index=True)
        
        # Write back
        combined_df.to_excel(writer, sheet_name="Web Scraping Data", index=False)
        writer.close()
        
    except FileNotFoundError:
        # Create new file if doesn't exist
        write_with_pandas(new_data, filename)

# Append more data
more_data = {
    "Title": ["New Data 1", "New Data 2"],
    "URL": ["https://new1.com", "https://new2.com"],
    "Timestamp": [datetime.datetime.now()] * 2,
    "Status": ["Success", "Success"]
}
append_to_excel(more_data)
```

10. Complete Example: Web Scraper with All Features

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options
import pandas as pd
import time
import os
from datetime import datetime

class WebScraper:
    def __init__(self, headless=True, download_dir="downloads"):
        self.download_dir = os.path.abspath(download_dir)
        os.makedirs(self.download_dir, exist_ok=True)
        
        options = Options()
        if headless:
            options.add_argument("--headless")
        options.add_argument("--window-size=1920,1080")
        options.add_argument("--disable-blink-features=AutomationControlled")
        
        # Download preferences
        prefs = {
            "download.default_directory": self.download_dir,
            "download.prompt_for_download": False
        }
        options.add_experimental_option("prefs", prefs)
        
        self.driver = webdriver.Chrome(options=options)
        self.wait = WebDriverWait(self.driver, 10)
        
    def search_google(self, query):
        """Perform Google search and return results"""
        self.driver.get("https://www.google.com")
        
        # Find search box and search
        search_box = self.wait.until(EC.presence_of_element_located((By.NAME, "q")))
        search_box.send_keys(query)
        search_box.send_keys(Keys.RETURN)
        
        # Wait for results
        self.wait.until(EC.presence_of_element_located((By.ID, "search")))
        
        # Extract results
        results = []
        result_elements = self.driver.find_elements(By.CSS_SELECTOR, "div.g")
        
        for element in result_elements[:10]:  # Get top 10 results
            try:
                title = element.find_element(By.CSS_SELECTOR, "h3").text
                link = element.find_element(By.CSS_SELECTOR, "a").get_attribute("href")
                results.append({
                    "title": title,
                    "link": link,
                    "timestamp": datetime.now()
                })
            except:
                continue
        
        return results
    
    def save_to_excel(self, data, filename="search_results.xlsx"):
        """Save scraped data to Excel"""
        df = pd.DataFrame(data)
        df.to_excel(filename, index=False)
        print(f"Saved {len(data)} results to {filename}")
        return filename
    
    def take_screenshot(self, filename="screenshot.png"):
        """Take screenshot"""
        self.driver.save_screenshot(filename)
        print(f"Screenshot saved: {filename}")
    
    def execute_custom_js(self, script):
        """Execute custom JavaScript"""
        return self.driver.execute_script(script)
    
    def close(self):
        """Close the driver"""
        self.driver.quit()

# Usage example
if __name__ == "__main__":
    scraper = WebScraper(headless=True)
    
    try:
        # Search for something
        results = scraper.search_google("Selenium Python tutorial")
        
        # Save to Excel
        scraper.save_to_excel(results)
        
        # Take screenshot
        scraper.take_screenshot("google_search.png")
        
        # Execute JavaScript
        page_info = scraper.execute_custom_js("""
            return {
                title: document.title,
                url: window.location.href,
                linksCount: document.getElementsByTagName('a').length
            }
        """)
        print(f"Page info: {page_info}")
        
    finally:
        scraper.close()
```

Key Points to Remember:

1. Wait Strategies: Always use explicit waits (WebDriverWait) instead of time.sleep() for better reliability
2. Element Locators: Use By.ID, By.NAME, By.CSS_SELECTOR, etc. - CSS selectors are most flexible
3. Error Handling: Always include try-catch blocks for robust automation
4. Headless Mode: Great for production but use headed mode during development
5. File Paths: Always use absolute paths (os.path.abspath()) for file operations
6. Clean Up: Always close the driver to free up resources

This comprehensive guide should give you a solid foundation for using Selenium in Python with all the features you requested. Practice with these examples and modify them according to your specific use cases!
