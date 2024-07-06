"# oliveyoung crawling" 
import time
import pandas as pd
from selenium import webdriver 
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options

# URL 설정
base_url = 'https://www.oliveyoung.co.kr/store/goods/getGoodsDetail.do?goodsNo=A000000204781&dispCatNo=90000010009&trackingCd=Best_Sellingbest&t_page=%EB%9E%AD%ED%82%B9&t_click=%ED%8C%90%EB%A7%A4%EB%9E%AD%ED%82%B9_%EC%A0%84%EC%B2%B4_%EC%83%81%ED%92%88%EC%83%81%EC%84%B8&t_number=1'

# Chrome 드라이버 옵션 설정
chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-extensions')
chrome_options.add_argument('--disable-dev-shm-usage')
chrome_options.add_experimental_option("excludeSwitches", ["enable-automation"])
chrome_options.add_experimental_option('useAutomationExtension', False)

# Chrome 드라이버 초기화
driver_path = r"C:\\Users\\LG\\.wdm\\drivers\\chromedriver\\win64\\125.0.6422.76\\chromedriver-win32\\chromedriver.exe"
service = Service(driver_path)
driver = webdriver.Chrome(service=service, options=chrome_options)

# 웹페이지 열기
driver.get(base_url)
time.sleep(2)
# 리뷰 버튼 클릭
review_button = driver.find_element(By.XPATH, "/html/body/div[3]/div[8]/div/ul/li[3]/a")
review_button.click()
time.sleep(2)


# 데이터 프레임 생성 
df_review_cos1 = pd.DataFrame(columns=['txt'])

# 리뷰 크롤링 함수
def review_crawling(df, page_num):    
    for current_page in range(1, page_num+1):      
        # 리뷰의 인덱스는 한 페이지 내에서 1~10까지 존재
        for i in range(1, 11):  # 한 페이지 내 10개 리뷰 크롤링
            try:
                txt = driver.find_element(By.CSS_SELECTOR, f'#gdasList > li:nth-child({i}) > div.review_cont > div.txt_inner').text
                df.loc[len(df)] = [txt]
            except:
                pass
        try:
            if current_page % 10 != 0: # 현재 페이지가 10의 배수가 아닐 때
                if current_page // 10 < 1: # 페이지 수가 한 자리수 일 때  
                    # 리뷰 10개 긁으면 next 버튼 클릭
                    page_button = driver.find_element(By.CSS_SELECTOR, f'#gdasContentsArea > div > div.pageing > a:nth-child({current_page%10+1})')
                    page_button.click() 
                    time.sleep(2)
                else: # 페이지 수가 두자리 수 이상일 때 
                    # 리뷰 10개 긁으면 next 버튼 클릭
                    page_button = driver.find_element(By.CSS_SELECTOR, f'#gdasContentsArea > div > div.pageing > a:nth-child({current_page%10+2})')
                    page_button.click() 
                    time.sleep(2)
            else:
                next_button = driver.find_element(By.CSS_SELECTOR, '#gdasContentsArea > div > div.pageing > a.next')
                next_button.click() # 현재 페이지가 10의 배수일 때 페이지 넘김 버튼 클릭
                time.sleep(2)
        except:
            pass
        print(f'{current_page}페이지 크롤링 완료')

    return df

# 크롤링 실행
page_num = 53 # 원하는 페이지 수 설정
df_review_cos1 = review_crawling(df_review_cos1, page_num)

# 크롤링 종료
driver.quit()

# 결과 확인
df_review_cos1

# 엑셀 파일 경로 설정
excel_file_path = "C:\\Users\\LG\\Documents\\product_reviews.xlsx"


# 데이터프레임을 엑셀 파일로 저장
df_review_cos1.to_excel(excel_file_path, index=False)

print("엑셀 파일 저장 완료:", excel_file_path)
