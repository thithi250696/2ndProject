# :pushpin: TOYCONN 🚗🚓🛴🏍
>동네기반 중고 장난감 대여 서비스 </br>
> 데모사이트 링크: http://localhost:8081/ToyConn_pro/main.jsp
> ppt pdf 파일: [Uploading TOYSCONN_발표.pdf…]()

</br>

## 1. 제작 기간 & 참여 인원❤
- 2023년 11월 14일 ~ 11월 30일
- 팀 프로젝트

</br>

## 2. 사용 기술❤
#### `DB 및 Crawling`
  - MySQL Oracle 11
  - Python
  - Selenium
  - BeautifulSoup
</br>

## 3. ERD - DB 테이블 설계 ❤
<img src = "https://github.com/2023-SMHRD-IS-BigData2/R2L3_team/blob/main/ToyConn_pro/src/main/webapp/images/erd.png">


## 4. 핵심 기능 ❤
  ## 4.1 Web Crawling 데이터 수집
- 중고장난감: 중고사이트에 쿨롤링 -> 데이터 수집 -> excel파알으로 수출 -> 데이터 처리 -> oracle데이터배이스 SQL문 활용하고 저장<br>
  참조사이트: 당근마켓, 중고나라, 스마트네이버스토어
  <img src ="https://github.com/thithi250696/thi/blob/main/carrot.PNG">
- 프리미엄: 스마트네이버스터에 쿨롤링 -> 데이터 수집 -> excel파알으로 수출 -> 데이터 처리 -> oracle데이터배이스 SQL문 활용하고 저장<br>
- 아래에 코드 입니다.<br>
  ### 상품명, 상품값 데이터 수집 🌼
#### library import
from selenium import webdriver as wb <br>
from bs4 import BeautifulSoup as bs<br>
import requests as req<br>
import os<br>
import pandas as pd<br>
from selenium.webdriver.common.keys import Keys<br>
from selenium.webdriver.common.by import By<br>
import time<br>
from urllib.request import urlretrieve <br>
#### get url
url = "https://www.daangn.com/search/%EC%9E%A5%EB%82%9C%EA%B0%90/"<br>
head_option = {<br>
'user-agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36'}<br>
res= req.get(url, headers = head_option)<br>
html = bs(res.text,'html')<br>
#### run driver
driver = wb.Chrome();<br>
driver.get(url);<br>
#### 전체 파이지 화면 보기
last_height = driver.execute_script("return document.body.scrollWidth")<br>
while(True):<br>
    body = driver.find_element(By.TAG_NAME,'body')<br>
    body.send_keys(Keys.END)<br>
    time.sleep(1)<br>
    current_height=driver.execute_script("return document.body.scrollWidth")<br>
    if current_height == last_height:<br>
        break<br>
    last_height = current_height<br>
    print(last_height, current_height)<br>
### 상품명, 상품값 데이터 수집<br>
names = []<br>
prices = []<br>
length = len(driver.find_elements(By.CLASS_NAME,"article-title"))<br>
for i in range (length):<br>
    name = driver.find_elements(By.CLASS_NAME, "article-title")[i].text<br>
    price = driver.find_elements(By.CLASS_NAME, "article-price")[i].text<br>
    names.append(name)<br>
    prices.append(price)<br>
#### 데이터 프레임 넣기<br>
data = pd.DataFrame(data = zip(names,prices),columns = ['Items','Price'])<br>
#### export파일<br>
data.to_excel('당근_장난감23.11.13.03.xlsx',index = False)<br>
### 이미지 수집 ☘
imgs = driver.find_elements(By.CSS_SELECTOR,'div.card-photo>img')
data = []
for i in range(len(imgs)):
    data.append(urlretrieve(imgs[i].get_attribute('src'),f"C:\\Users\\dd\\AppData\\Local\\Temp\\Carrot\\Carrot{i}.jpg"))
### 상품표현 수집🌸🌸
#### link 수집
detail_link = []
for i in range(len(driver.find_elements(By.CSS_SELECTOR,'a.flea-market-article-link'))):
    detail_link.append(driver.find_elements(By.CSS_SELECTOR,'a.flea-market-article-link')[i].get_attribute('href'))
detail_link
#### 상품펴현 수집
details =[]
for i in range(len(detail_link)):<br>
    driver = wb.Chrome()<br>
    driver.get(detail_link[i])<br>
    detail = driver.find_element(By.CSS_SELECTOR,'div#article-detail').text.strip('\n')<br>
    details.append(detail)<br>

#### 데이터 프레임 넣기
data = pd.DataFrame(data = zip(range(1,100),details),columns = [no','detail'])
#### export파일
data.to_excel('당근_장난감_detail23.11.13.03.xlsx',index = False)
    
## 4.2 Oracle Database 관리
--1. table:  회원정보 <br>
CREATE TABLE MEMBER_INFO(<br>
USER_ID VARCHAR2(20) PRIMARY KEY,<br>
NICK VARCHAR2(20),<br>
ADDRESS VARCHAR2(100),<br>
SCORE NUMBER,<br>
JOIN_DATE DATE<br>
);<br>
-- 2. table: 회원별등록 상품정보 <br>
CREATE TABLE PRODUCT_INFO(<br>
P_NUM NUMBER PRIMARY KEY,<br>
USER_ID VARCHAR2(20),<br>
CATEGORY VARCHAR2(100),<br>
P_NAME VARCHAR2(200),<br>
RENT_PRICE NUMBER,<br>
P_QUALITY VARCHAR2(20),<br>
P_CONTENT VARCHAR2(1000),<br>
P_STATUS VARCHAR2(50),<br>
R_STATUS VARCHAR2(50),<br>
GENDER VARCHAR2(50),<br>
IMAGE_FILE VARCHAR2(200)<br>
);<br>
CREATE SEQUENCE NUM_PRODUCT<br>
START WITH 1<br>
INCREMENT BY 1;<br>
--3.  table: 거래내역<br>
CREATE TABLE TRANSACTION_INFO(<br>
TRANS_NUM NUMBER PRIMARY KEY,<br>
USER_ID VARCHAR2(20),<br>
P_NUM NUMBER,<br>
RENT_START_DATE DATE,<br>
RENT_END_DATE DATE,<br>
TRANS_STATUS VARCHAR2(50)<br>
);<br>
CREATE SEQUENCE NUM_TRANSACTION<br>
START WITH 1<br>
INCREMENT BY 1;<br>
-- 4. table: 결재내역<br>
CREATE TABLE PAYMENT_INFO(<br>
TRANS_NUM NUMBER,<br>
PAY_CHOICE VARCHAR2(20),<br>
PAY_AMOUNT NUMBER NOT NULL,<br>
PAY_TIME DATE<br>
);<br>
-- 5. table: 반납내역<br>
CREATE TABLE RETURN_INFO(<br>
RETURN_NUM NUMBER,<br>
TRANS_NUM NUMBER,<br>
RETURN_DATE DATE,<br>
RENT_START_DATE DATE<br>
);<br>
CREATE SEQUENCE NUM_RETURN<br>
START WITH 1<br>
INCREMENT BY 1;<br>


-- 6. table: 대화내역<br>
CREATE TABLE MESSAGE_BOARD(<br>
CHAT_NUM NUMBER PRIMARY KEY,<br>
SENDER VARCHAR2(20), <br>
RECIPIENT  VARCHAR2(20), <br>
TEXT_CONTENT VARCHAR2(1000), <br>
CHAT_TIME DATE<br>
);<br>
CREATE SEQUENCE NUM_CHAT<br>
START WITH 1<br>
INCREMENT BY 1;<br>
--FOREIGN KEY 생성<br>
ALTER TABLE TRANSACTION_INFO <br>
ADD CONSTRAINT FK_TRANSACTION_INFO<br>
FOREIGN KEY(P_NUM) REFERENCES PRODUCT_INFO(P_NUM);<br>

ALTER TABLE MESSAGE_BOARD<br>
ADD CONSTRAINT FK_MESSAGE_BOARD<br>
FOREIGN KEY(SENDER) REFERENCES MEMBER_INFO(USER_ID);<br>

ALTER TABLE MESSAGE_BOARD<br>
ADD CONSTRAINT FK_MESSAGE_BOARD_01<br>
FOREIGN KEY(RECIPIENT) REFERENCES MEMBER_INFO(USER_ID);<br>



ALTER TABLE PAYMENT_INFO <br>
ADD CONSTRAINT FK_PAYMENT_INFO<br>
FOREIGN KEY(TRANS_NUM) REFERENCES TRANSACTION_INFO(TRANS_NUM);<br>

commit;<br>

