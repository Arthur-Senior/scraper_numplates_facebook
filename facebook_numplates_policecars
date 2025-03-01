import os
import platform
import requests
import tarfile
import zipfile
import logging
from selenium import webdriver
from selenium.webdriver.firefox.service import Service
from selenium.webdriver.firefox.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
import time
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException
import re

# Configure logging
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

class GeckoDriverManager:
    def __init__(self, download_path="drivers"):
        self.download_path = download_path
        self.system = platform.system()
        self.geckodriver_exe = "geckodriver.exe" if self.system == "Windows" else "geckodriver"

    def download_geckodriver(self):
        urls = {
            "Windows": "https://github.com/mozilla/geckodriver/releases/download/v0.33.0/geckodriver-v0.33.0-win64.zip",
            "Darwin": "https://github.com/mozilla/geckodriver/releases/download/v0.33.0/geckodriver-v0.33.0-macos.tar.gz",
            "Linux": "https://github.com/mozilla/geckodriver/releases/download/v0.33.0/geckodriver-v0.33.0-linux64.tar.gz"
        }

        url = urls.get(self.system)
        if not url:
            raise Exception("Unsupported OS")

        file_name = "geckodriver.zip" if url.endswith(".zip") else "geckodriver.tar.gz"
        file_path = os.path.join(self.download_path, file_name)

        logger.info(f"Downloading geckodriver from {url}...")
        response = requests.get(url, timeout=30)
        response.raise_for_status()

        with open(file_path, "wb") as file:
            file.write(response.content)

        if file_name.endswith(".zip"):
            with zipfile.ZipFile(file_path, "r") as zip_ref:
                zip_ref.extractall(self.download_path)
        else:
            with tarfile.open(file_path, "r:gz") as tar_ref:
                tar_ref.extractall(self.download_path)

        os.remove(file_path)
        geckodriver_path = os.path.join(self.download_path, self.geckodriver_exe)
        if not os.path.exists(geckodriver_path):
            raise Exception("Geckodriver extraction failed")

        if self.system != "Windows":
            os.chmod(geckodriver_path, 0o755)

        return geckodriver_path

    def setup_geckodriver(self):
        if not os.path.exists(self.download_path):
            os.makedirs(self.download_path)

        return self.download_geckodriver()

class WebDriverHandler:
    def __init__(self):
        self.driver = None
        self.driver_manager = GeckoDriverManager()

    def initialize_driver(self):
        geckodriver_path = self.driver_manager.setup_geckodriver()
        options = Options()
        options.set_capability("acceptInsecureCerts", True)

        service = Service(executable_path=geckodriver_path)
        self.driver = webdriver.Firefox(service=service, options=options)
        return self.driver

    def close_driver(self):
        if self.driver:
            self.driver.quit()

    def solve_accept_cookies_popup(self):
        logger.debug("Handling cookie popup by sending 19x Tab and Space")
        body_element = self.driver.find_element(By.TAG_NAME, "body")
        for _ in range(17):
            body_element.send_keys(Keys.TAB)
            time.sleep(0.5)  # Adjust the delay as needed
        body_element.send_keys(Keys.SPACE)
        logger.debug("Pressed Space after 19x Tab")
        time.sleep(2)  # Wait for the action to take effect



    def login_to_facebook(self, username, password):
            logger.debug("Logging in to Facebook...")
            try:
                # Wait for the username input to be present
                time.sleep(5)
                # Send username, press Tab, then send password
                body_element = self.driver.find_element(By.TAG_NAME, "body")
                #body_element.send_keys(Keys.TAB)
                body_element.send_keys(username)
                body_element.send_keys(Keys.TAB)
                body_element.send_keys(password)
                body_element.send_keys(Keys.RETURN)

                # Wait for the login to complete
                time.sleep(10)
                logger.debug("Logged in to Facebook")
            except TimeoutException:
                logger.error("Login elements not found on the page")

    def search_polish_number_plates(self):
        logger.debug("Searching for Polish number plates...")
        number_plate_pattern = r'[A-Z]{1,3}\s?[0-9]{1,5}[A-Z]{0,2}'

        # Scroll down the page to load more content
        last_height = self.driver.execute_script("return document.body.scrollHeight")
        while True:
            # Extract text from the page
            page_text = self.driver.find_element(By.TAG_NAME, "body").text
            # Search for number plates using regex
            plates = re.findall(number_plate_pattern, page_text)
            if plates:
                logger.info(f"Found Polish number plates: {plates}")

            # Scroll down by pressing the arrow down key 25 times
            for _ in range(25):
                self.driver.find_element(By.TAG_NAME, "body").send_keys(Keys.ARROW_DOWN)
                time.sleep(0.1)  # Small delay between key presses

            # Calculate new scroll height and compare with last scroll height
            new_height = self.driver.execute_script("return document.body.scrollHeight")
            if new_height == last_height:
                break
            last_height = new_height

        logger.debug("Finished searching for Polish number plates.")

# Example usage
if __name__ == "__main__":
    handler = WebDriverHandler()
    driver = handler.initialize_driver()
    try:
        driver.get("https://www.facebook.com/nieoznakowaneradiowozy")
        handler.solve_accept_cookies_popup()
        handler.login_to_facebook("","")
        logger.info("Page title is: %s", driver.title)
        handler.search_polish_number_plates()
    except Exception as e:
        logger.error("An error occurred: %s", str(e))

