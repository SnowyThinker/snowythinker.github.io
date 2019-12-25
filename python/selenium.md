# Selenium 表单提交

~~~python
from selenium import webdriver

options = webdriver.ChromeOptions()
# 打开chrome开发工具
options.add_argument('auto-open-devtools-for-tabs')
browser = webdriver.Chrome(chrome_options=options)
# 窗口最大化
browser.maximize_window()

# 打开网址填写用户名与密码登录操作
browser.get('http://www.jd.com')
browser.find_element_by_name('loginName').send_keys('XXX')
browser.find_element_by_name('password').send_keys('XXX')
browser.find_element_by_class_name('login').click()
~~~