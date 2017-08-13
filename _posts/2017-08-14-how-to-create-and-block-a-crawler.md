Internet includes an enormous amount of data but this data is without a schema and distributed. If one needs to extract some specific information, then a web crawler needs to be written unless this information is provided through APIs.



Writing a web crawler requires the writer to know the way source generates content. This is a dynamic process, the source can change, the writer must adapt its crawler accordingly. If you are the administrator of the source, you generally do not want your source to be crawled without your knowledge. That means you can give keys to others that enable them to pull data by using APIs but  you may want to restrict other raw content crawlers because they swallow extra resource from your valuable server and that makes you provide less privileged access to your own users. 



There are many methods for the administrator to block a crawler. For example they can check the frequency of the connections, group them by ip and block the ip temporarily or permanently. To overcome this, the crawler should be able to hide itself behind proxies dynamically. Another approach the administrator can take is to put authentication layer so anyone wanting to access to the source should authenticate first. Crawler needs to be able to overcome this as well. Crawler can behave like a real browser by using header manipulations and fool the source. 



One way of bypassing the auth layer of the source is using UI testing tools such as Selenium. One can easily open a website by using Selenium filling boxes, clicking buttons, getting redirected and finally being provided cookies, sessions, etc. Later a person can gather these cookies and session info for further crawling the site via bypassing the auth layer. Below you can see the example usage of Selenium in python language.



```python
import requests
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By

def init_phantomjs_driver(self, *args, **kwargs):        
        headers = {
            'Accept-Language':'tr,en-US;q=0.8,en;q=0.6',
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36',
            'Connection': 'keep-alive',
            'Upgrade-Insecure-Requests': '1'
        }

        for key, value in headers.iteritems():
            webdriver.DesiredCapabilities.PHANTOMJS['phantomjs.page.customHeaders.{}'.format(key)] = value

        webdriver.DesiredCapabilities.PHANTOMJS['phantomjs.page.settings.userAgent'] = 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.116 Safari/537.36'

        driver =  webdriver.PhantomJS('phantomjs', *args, **kwargs)
        driver.set_window_size(1400,1000)

        return driver```
 
url = 'XXX'
loginInputXpath = '//input[@type=\'text\']'
passInputXpath = '//input[@type=\'password\']'
loginButtonXpath = '//button[@type=\'submit\']'

proxyArgs = [
    '--proxy=127.0.0.1:PORT',
    '--proxy-type=socks5'
]

driver = self.init_phantomjs_driver(service_args=proxyArgs)
driver.get(url)

WebDriverWait(driver, 60).until(EC.presence_of_element_located(
    (By.XPATH, loginInputXpath)))

UN = driver.find_element_by_xpath(loginInputXpath)
UN.send_keys(username)

PS = driver.find_element_by_xpath(passInputXpath)
PS.send_keys(password)

LI = driver.find_element_by_xpath(loginButtonXpath)

LI.click()

WebDriverWait(driver, 60).until(EC.presence_of_element_located(
    (By.ID, 'some_element')))

headers = {
    "User-Agent":
    "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.157 Safari/537.36"
}

session = requests.Session()
session.headers.update(headers)

for cookie in driver.get_cookies():
    c = {cookie['name']: cookie['value']}
    session.cookies.update(c)

driver.quit()
```



We initiate a phantomjs driver with related headers, window_size, etc. Phantomjs is a headless browser. Selenium uses this driver to connect to the UI of the url and fill username, password and click login button. The way selenium works is all about waiting for a spesific event to happen for example finding the existance of an element in the dom tree. We can use XPATH for identifying elements to make Selenium search for and find it.



We also used proxy while initiating phantomjs driver. The reason for this is not to be identified and blocked by the source. Tor network is a good option as a proxy source but there are many proxies out on the internet one can use.



An administrator can also identify headless browsers connecting its source by checking various properties of the browser such as plugins a browser has. Chrome already has a pdf viewer plugin for example but a headless chrome would have zero plugins enabled and an administrator could check that to search for headless browsers trying to connect to its source.



Nevertheless after the authentication is successfully completed, we transfer our headers and cookies into requests session so that we can continue crawling using requests module.



Quitting the driver after our job with Selenium is completed is important because it consumes a lot of system resources. Efficiency is also an important part of the job other than the functionality so we try to minimize the load while maximizing outcome. By the way if you face any exceptions through your code, you can take a screenshot of the last state of the driver or get html content of the page, this data can be useful for debugging.



After we crawled the source, either we can directly write it into a file in raw format so that we can use it in future or we can create a schema and possibly create a DataFrame and make some transformations on it and save it to a database or write it into a file with a spesific format but this will be the topic of another post.



After the authentication part is complete we can traverse the content of the web page. A web page has a structure of a tree. It is called DOM, Document Object Model. It consists of elements in a hierarchical order.
To extract the content we can traverse through the nodes of this tree. To do that we can use BeautifulSoup library of python. 



```python
    soup = BeautifulSoup(raw_html_content, 'lxml')
    options = soup.find('form', {'id': FORM_ID'}).find_all('input')
```



Above code constructs a beautiful soup object by providing it the raw html data and lxml as the choice of parser. Afterwards we try to find the form with attribute id equaling FORM_ID and we go deeper and find all input elements which are children of the form element we just found in the DOm hierarchy. Logic is really simple, find elements you are looking for through filtering by their type and attributes and then extract the content.



These are the basics of crawling process in general. Hope it helps you in your crawling process.



