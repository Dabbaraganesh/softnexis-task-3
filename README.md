# softnexis-task-3
import requests
from bs4 import BeautifulSoup
import random
import time
from backoff import expo, on_exception
from captcha_solver import solve_captcha # Custom CAPTCHA module
PROXY_POOL = [
"http://user:pass@192.168.1.1:8080",
"socks5://user:pass@10.0.0.1:9050"
]
USER_AGENTS = [
"Mozilla/5.0 (Windows NT 10.0; Win64; x64) ...",
"Mozilla/5.0 (Macintosh; Intel Mac OS X 13_4) ..."
]
@on_exception(expo, requests.exceptions.RequestException, max_tries=5)
def scrape_with_retry(url):
proxy = random.choice(PROXY_POOL)
headers = {"User-Agent": random.choice(USER_AGENTS)}
try:
response = requests.get(
url,
proxies={"http": proxy, "https": proxy},
headers=headers,
timeout=10
)
# CAPTCHA detection
if "captcha" in response.text.lower():
captcha_token = solve_captcha(response.content)
# Resubmit with solved CAPTCHA
return handle_captcha(url, captcha_token, proxy)
response.raise_for_status()
return parse_data(response.text)
except requests.HTTPError as e:
if e.response.status_code == 429:
time.sleep(2 ** random.uniform(1, 5)) # Exponential backoff
raise
def parse_data(html):
soup = BeautifulSoup(html, "lxml")
# Data extraction logic
return {
"title": soup.select_one("h1.product-title").text.strip(),
"price": soup.select_one(".price").attrs["data-value"]
}
def handle_captcha(url, solution, proxy):
# Implementation for CAPTCHA resubmission
pass
if __name__ == "__main__":
data = scrape_with_retry("https://target-ecommerce-site.com/product/123")
print(data)
