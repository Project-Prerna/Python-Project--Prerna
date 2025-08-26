# Python-Project--Prerna
import os
import time
import pandas as pd
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException, NoSuchElementException
from webdriver_manager.chrome import ChromeDriverManager

genre = input("Enter movie genre (e.g., action, comedy, drama): ").lower()

url = f"https://www.imdb.com/search/title/?genres={genre}&sort=user_rating,desc"

options = Options()
options.add_argument("--start-maximized")
options.add_argument("--disable-gpu")
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")
options.add_argument("--disable-blink-features=AutomationControlled")

service = Service(ChromeDriverManager().install())
driver = webdriver.Chrome(service=service, options=options)

try:
    driver.get(url)

    try:
        WebDriverWait(driver, 30).until(
            EC.presence_of_all_elements_located((By.CSS_SELECTOR, ".ipc-metadata-list-summary-item"))
        )
    except TimeoutException:
        print("❌ Timed out waiting for movie list to load. Try again later.")
        driver.quit()
        exit()

    movies = driver.find_elements(By.CSS_SELECTOR, ".ipc-metadata-list-summary-item")
    movie_data = []

    for movie in movies[:10]:  
      
        try:
            title = movie.find_element(By.CSS_SELECTOR, "h3.ipc-title__text").text
        except NoSuchElementException:
            title = "N/A"

        try:
            imdb_rating = movie.find_element(By.CSS_SELECTOR, ".ipc-rating-star--rating").text
        except NoSuchElementException:
            imdb_rating = "N/A"

        try:
            year = movie.find_element(By.CSS_SELECTOR, "span.dli-title-metadata-item:nth-child(1)").text
        except NoSuchElementException:
            year = "N/A"

        try:
            length = movie.find_element(By.CSS_SELECTOR, "span.dli-title-metadata-item:nth-child(2)").text
        except NoSuchElementException:
            length = "N/A"

        try:
            link = movie.find_element(By.CSS_SELECTOR, "a.ipc-title-link-wrapper").get_attribute("href")
        except NoSuchElementException:
            link = "N/A"

        movie_data.append([title, imdb_rating, year, length, link])

    file_name = f"top_{genre}_movies.csv"
    if os.path.exists(file_name):
        file_name = f"top_{genre}_movies_{int(time.time())}.csv"

    df = pd.DataFrame(movie_data, columns=["Title", "IMDB Rating", "Year", "Length", "URL"])
    df.to_csv(file_name, index=False, encoding="utf-8")
    print(f"\n✅ Data saved successfully in {file_name}\n")

    print("🎬 Top 5 Movie Suggestions:")
    print(df.head(5).to_string(index=False))

except Exception as e:
    print(f"❌ Error occurred: {type(e).__name__} - {e}")

finally:
    driver.quit()
