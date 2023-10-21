# instagram_follower_bot
Logs in to Insta, searches for a page (adidas), gets to the page, clicks on followers of the page, starts following them. If it is a mutual follower, skips that user. Made using selenium.



We will develop a program that automatically follows the followers of a certain page.

Steps
1. Go to Instagram.com
2. Log In
3. If there is 2FA, ask for the code
4. Once logged in, go to adidas's page
5. Click on followers
6. Loop through the followers
7. If the button beside the followers already says "Following", do nothing, otherwise follow them



***1. Go to Instagram.com***

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.by import By

options = webdriver.ChromeOptions()
options.add_experimental_option("detach", True)
driver = webdriver.Chrome(options=options, service=Service(ChromeDriverManager().install()))
driver.get("https://www.instagram.com/")
```

***2. Log In***
***3. If there is 2FA, ask for the code***

The login is a bit tricky, since some of the elements have dynamic class names.

So resorted to finding element by XPATH mostly.

Now what to do if the XPATH is dynamic? In the "Save your login info page", there are buttons called "Save" and "Not Now".

It is also not a link, so cannot use "By.LINK_TEXT"

***XPath contains***

XPath contains is a function within Xpath expression which is used to search for the web elements that contain a particular text.

General format - 
```python
Xpath=//tagname[contains (@Attribute, 'Value')]
```

If I want to find an element that has an ID with a certain string, e.g. 'test' - 
```python
Xpath=//tagname[contains(@id,'test')]
```

If I want to find an element that has an that has a certain string, in my case, "Not Now" - 
```python
Xpath=//tagname[contains(text(),'Not Now')]
```

In code, it will be - 
```python
not_now = driver.find_element(By.XPATH, '//div[contains(text(), "Not Now")]')
```

if you don't know the tagname, i.e. it can be div/span/a/h1, you can use an asterix - 
```python
not_now = driver.find_element(By.XPATH, '//div[contains(text(), "Not Now")]')
```

Now let's log in - 

```python
# Wait till the login fields are available
WebDriverWait(driver, 300).until(EC.visibility_of_element_located((By.XPATH, '//*[@id="loginForm"]/div/div[1]/div/label/input')))
username_input = driver.find_element(By.XPATH, '//*[@id="loginForm"]/div/div[1]/div/label/input')
username_input.send_keys('*********')

pass_input = driver.find_element(By.XPATH, '//*[@id="loginForm"]/div/div[2]/div/label/input')
pass_input.send_keys('***********')

login_btn = driver.find_element(By.XPATH, '//*[@id="loginForm"]/div/div[3]')
login_btn.click()

# Ask for the auth code generated by an authenticator app
auth_code = input('What is the code generated by the authenticator?: ')
# After you clicked the login button, the driver auto selects the auth code input box, so you just gotta switch to active element
auth_input = driver.switch_to.active_element
auth_input.click()
auth_input.send_keys(auth_code)
auth_input.send_keys(Keys.ENTER)

# A different screen appears, where they ask if they should save the login info, wait for that screen to be visible
WebDriverWait(driver, 60).until(EC.visibility_of_element_located((By.CLASS_NAME, '_aa59')))
# Find the element with the text "Not Now"
not_now = driver.find_element(By.XPATH, '//div[contains(text(), "Not Now")]')
not_now.click()

Another window opens, asking to send notifications, click "Not Now"
WebDriverWait(driver, 60).until(EC.visibility_of_element_located((By.CLASS_NAME, '_a9_1')))
dont_enable_notification = driver.find_element(By.CLASS_NAME, '_a9_1')
dont_enable_notification.click()
```

***4. Once logged in, go to adidas's page***
```python
# Maximize window so the search bar with the text "search"  is visible
driver.maximize_window()

# Find the search bar and click on it
search_bar = driver.find_element(By.XPATH, '//span[contains(text(), "Search")]')
search_bar.click()

# A new search bar opens, and it is in focus. So the driver needs to send text in that active element
elem = driver.switch_to.active_element
elem.send_keys('adidas')

# Wait for the search to complete, and adidas page to show
WebDriverWait(driver, 60).until(EC.visibility_of_element_located((By.XPATH, '//span[contains(text(), "adidas")]')))

# Now find the result with the name adidas
adidas_page = driver.find_element(By.XPATH, '//span[contains(text(), "adidas")]')
adidas_page.click()

# We're In
```

***5. Click on followers***
```python
# Wait for the adidas page to fully load
WebDriverWait(driver, 60).until(EC.visibility_of_element_located((By.XPATH, '//h2[contains(text(), "adidas")]')))
time.sleep(3.0)

# Find the link with text 'Followed by'
followers = driver.find_element(By.XPATH, "//span[contains(text(), 'Followed by ')]")
followers.click()

# Now find the element with text "See all followers" and click
time.sleeptime.sleep(3.0)
all_followers = driver.find_element(By.XPATH, "//a[contains(text(), 'See all followers')]")
all_followers.click()
```

Alternatively, we can simply do this - 

```python
# Wait for the adidas page to fully load
WebDriverWait(driver, 60).until(EC.visibility_of_element_located((By.XPATH, '//h2[contains(text(), "adidas")]')))
# Click on followers
followers = driver.find_element(By.XPATH, "//a[contains(text(), 'followers')]")
followers.click()
```


***6. Loop through the followers***

***7. If the button beside the followers already says "Following", do nothing, otherwise follow them***

Now consider these two texts "Followers" and "Follow".
If we use contains functions here it will locate all of these elements, because all of these elements contains "Follow".
So here, we need exact pattern matching/text matching.
We need the text() method.

Basically you just skip the "contains" part in the code.

```python
# Wait till the followers window is visible
# It contains the text 'Only Adidas can see all followers'
WebDriverWait(driver, 60).until(EC.visibility_of_element_located((By.XPATH, '//span[text()=" can see all followers."]')))

# # Find the follow buttons
follow_buttons = driver.find_elements(By.XPATH, '//div[text()="Follow"]')
# Loop through the follow buttons and click on them
for i in follow_buttons:
    print(i.text)
```

This method does not work, it only finds the divs, not the buttons, so a click cannot be managed.

This method below however, works.

```python
# Find the follow buttons
buttons = driver.find_elements(By.CLASS_NAME, '_acan')
# Now this list contains all the follow buttons, including the one to follow the adidas page, at index 0
# We do not want to click that
# We want to click the follow buttons of the followers, that start from index 1
for i in buttons[1:]:
    i.click()
    # What if you clicked a button that says "Following"?
    if driver.find_element(By.XPATH, "//button[contains(text(), 'Cancel')]"):
        cancel_button = driver.find_element(By.XPATH, "//button[contains(text(), 'Cancel')]")
        cancel_button.click()
```

Done
