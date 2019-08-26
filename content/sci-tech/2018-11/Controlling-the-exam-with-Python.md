---
title: Controlling the exam with Python
date: 2018-11-22 13:19:33
tags: ["Python"]
categories: ["sci-tech"]
draft: false
toc: true
summary: "Using Python to do the exam automatically"
---

## Predicaments

We're asked to learn a library about safety.

However, if the duration is greater than 5 minutes (~300 seconds), we need to click `agree` to let the timing continue. So, when we finish the learning progress, we'll click **6*60/5 = 72** times ........

<!--more-->

![example](/images/sci-tech/2018-11/exam_1.png)![example_2](/images/sci-tech/2018-11/exam_2.png)

## Solution (Script)

```
import time
import selenium
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.common.exceptions import TimeoutException
from selenium.webdriver.support import expected_conditions as EC

# Please edit:
username = 'your_username'
password = 'your_password'

def get_driver(username,password):
    # Using Chrome to access web
    driver = webdriver.Chrome()
    # Open the website
    driver.get('http://examsafety.nuist.edu.cn/')

    # Select the id box
    id_box = driver.find_element_by_name('xuehao')
    # Send id information
    id_box.send_keys(username)
    # Find password box
    pass_box = driver.find_element_by_name('password')
    # Send password
    pass_box.send_keys(password)

    # Find login button
    login_button = driver.find_element_by_name('提交')
    login_button.click()

    # Find and click on list of courses
    title = '安全知识学习'
    button = driver.find_element_by_css_selector("[title^='"+title+"']")
    button.click()

    # Find the exam we need
    title = '大气物理学院实验室安全考试通用题库'
    button = driver.find_element_by_link_text(title)
    button.click()

    return driver

def accept_alert(driver):
    try:
        # Wait for 300 seconds ...
        WebDriverWait(driver, 60*5).until(EC.alert_is_present())
        # Switch to the alert and accept
        alert = driver.switch_to.alert
        alert.accept()
        print("alert accepted")
    except TimeoutException:
        print("no alert, wait for 5 seconds")

        # Just in case the alert is slow ...
        try:
            WebDriverWait(driver, 5).until(EC.alert_is_present())
            alert = driver.switch_to.alert
            alert.accept()
            print("Finally, alert is accepted!")
        except TimeoutException:
            print("no alert")

def main():
    driver = get_driver(username,password)

    # If lefttime is greater than 300, let python click the pop-up for you.
    k = 0
    while True:
        accept_alert(driver)
        k += 1
        if k == 72:
            break

    driver.close()

if __name__ == '__main__':
    main()
```

Now, we can let it go!

## References

1. [Controlling the Web with Python](https://towardsdatascience.com/controlling-the-web-with-python-6fceb22c5f08)
2. [Selenium WebDriver Python, search WebElement](https://stackoverflow.com/questions/34445280/selenium-webdriver-python-search-webelement)
3. [Locating Hyperlinks by Link Text](https://selenium-python.readthedocs.io/locating-elements.html#locating-hyperlinks-by-link-text)
4. [Handling A Confirmation Alert](https://www.techbeamers.com/handle-alert-popup-selenium-python/)
5. [Check if any alert exists using selenium with python](https://stackoverflow.com/questions/19003003/check-if-any-alert-exists-using-selenium-with-python)