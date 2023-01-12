# InstagramCrawler
Instagram Insight Crawler
***
## Summary

18개의 게시물의 인사이트(좋아요 수, 댓글 수, 해시태그, 업로드 시간, 프로필 방문 등) 크롤링
그 후 스프레드 시트에 저장

Crawling 18 new posts' insight(e.g. Likes, Comments, Used Tags, Uploaded Time, Profile Visit, etc.)<br>
Saving in Google Spreadsheets <br><br>


인사이트 종류 : 컨텐츠 이름,	크롤링 날짜,	컨텐츠 업로드 날짜,	좋아요 수,	댓글 수,	다이렉트 메세지 수,	스크랩 된 횟수,	광고 누름,	프로필 방문,	웹사이트 방문,	이메일 보내기 버튼 누름,	도달한 계정,	노출,	노출-탐색탭,	노출-홈,	노출-해시태그,	노출-프로필,	노출-기타,	팔로우 획득,	광고 여부,	광고비,	사용된 태그 <br><br>
Insights Type : Post Title, Crawling Date, Uploaded Date, Likes, Comments, DMs, Scrapted, Ad Click, Profile Visit, Website Tap, Email Button Tap, Accounts Reached, Impressions, Impressions from Explore, Impressions from Home, Impressions from Hashtag, Impressions from Profile, Impressions from Other, Follos From this Post, Ad or Not, Ad fee, Used Hashtags <br><br><br>

❗인스타그램은 XPATH를 수시로 바꾸므로 주의 요함
❗Instaram Changes Their XPATH Oftenly, Must Be Aware


~~~
 # Libraries
import time
from datetime import datetime
import pandas as pd
from openpyxl import Workbook
from selenium import webdriver
from selenium.webdriver import ActionChains
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import Select
import gspread
from oauth2client.service_account import ServiceAccountCredentials

# Connecting to Google Spreadsheets
scope = ['https://spreadsheets.google.com/feeds']
json_file_name = '*YOUR JSON FILE*' 
credentials = ServiceAccountCredentials.from_json_keyfile_name(json_file_name, scope)
gc = gspread.authorize(credentials)
spreadsheet_url = '*YOUR SPREAD SHEET URL*'

doc = gc.open_by_url(spreadsheet_url)
worksheet = doc.worksheet('*YOUR WORKSHEET NAME*')
print('Google Spreadsheets Connected')

# Open Instagram
options = Options()
user_agent = "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36"
options.add_argument('user-agent=' + user_agent)
options.add_argument('disable-gpu')
options.add_argument('headless') # Execute Without Chorme Window
options.add_argument('lang=ko')
dr = webdriver.Chrome(options = options)
wait = WebDriverWait(dr, 15)
dr.implicitly_wait(5) # Wait 5 Seconds on Any Error
#dr.maximize_window() # Use When You Edit The Code With Chrome Window
dr.get('https://www.instagram.com')

# Login
act = ActionChains(dr)
id_box = dr.find_element(By.CSS_SELECTOR, '#loginForm > div > div:nth-child(1) > div > label > input')
password_box = dr.find_element(By.CSS_SELECTOR, "#loginForm > div > div:nth-child(2) > div > label > input")
login_button = dr.find_element(By.CSS_SELECTOR, "#loginForm > div > div:nth-child(3) > button")
id = '*YOUR INSTAGRAM ID*'
psw  = '*YOUR PASSWORD*'
act.send_keys_to_element(id_box, id).send_keys_to_element(password_box, psw).click(login_button).perform()
print('Login Successful')

# Select English Option - To Make Variables in English Later
element = wait.until((EC.presence_of_element_located((By.XPATH, '/html/body/div[1]/div/div/div/div[1]/div/div/div/div[1]/div[1]/div[2]/section/div[2]/footer/div/div[2]/div[1]/span/select'))))
select = Select(dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[1]/div/div/div/div[1]/div[1]/div[2]/section/div[2]/footer/div/div[2]/div[1]/span/select'))
select.select_by_value('en')
print('Option Selected English')

 
# Click Profile Button
element = wait.until((EC.element_to_be_clickable((By.XPATH, '/html/body/div[1]/div/div/div/div[1]/div/div/div/div[1]/div[1]/div[1]/div/div/div/div/div[2]/div[7]'))))
profile = dr.find_element(By.XPATH,'/html/body/div[1]/div/div/div/div[1]/div/div/div/div[1]/div[1]/div[1]/div/div/div/div/div[2]/div[7]')
profile.click()

# Click Posts - From 18th Latest Post To The Latest Post
element = wait.until((EC.element_to_be_clickable((By.XPATH, '/html/body/div[1]/div/div/div/div[1]/div/div/div/div[1]/div[2]/div[2]/section/main/div/div[3]/article/div[1]/div/div[6]/div[3]'))))
post = dr.find_element(By.XPATH,'/html/body/div[1]/div/div/div/div[1]/div/div/div/div[1]/div[2]/div[2]/section/main/div/div[3]/article/div[1]/div/div[6]/div[3]')
act.move_to_element(post).perform()
post.click()

count = 18
while True:
    # Initialization
    Adtaps = 0
    ProfileVisits = 0
    Websitetaps = 0
    Emailbuttontaps = 0
    ad = 'X'
    cost = ''
    PeopleReached = 0 
    Accountsreached = 0
    del(PeopleReached)
    del(Accountsreached)
    # PeopleReached/Accountsreached 광고 여부 보고 초기화
    Impressions =0
    FromExplore = 0
    FromHome = 0
    FromHashtags = 0
    FromProfile = 0
    FromOther = 0
    Follows = 0

    # Upload Date
    uptime = dr.find_element(By.XPATH,'/html/body/div[1]/div/div/div/div[2]/div/div/div[1]/div/div[3]/div/div/div/div/div[2]/div/article/div/div[2]/div/div/div[2]/div[2]/div/div/a/div/time').get_attribute('datetime')
    uptime = uptime[:10]
    
    # Post Title
    tt = dr.find_element(By.XPATH,'/html/body/div[1]/div/div/div/div[2]/div/div/div[1]/div/div[3]/div/div/div/div/div[2]/div/article/div/div[2]/div/div/div[2]/div[1]/ul/div/li/div/div/div[2]/div[1]/span')
    title = tt.text
    title = title.splitlines()
    title = title[0]
    print('Upload Date, Post Title Loaded')
    print(uptime, ' ', title)

    # Hashtags in Comments
    """
    대댓글에 해시태그가 있음
    The hashtags are at the last comment's reply
    """
    # Cick All 'View Replies' (Just In Case For Later)
    cnt = 1
    while True :
        used_tags2=''
        try:
            comment_list  = dr.find_element(By.XPATH,'/html/body/div[1]/div/div/div/div[2]/div/div/div[1]/div/div[3]/div/div/div/div/div[2]/div/article/div/div[2]/div/div/div[2]/div[1]/ul/ul[{}]/div/li/div/div/div[2]/div[2]/div'.format(str(cnt)))
            act.move_to_element(comment_list).perform()
            try :
                more_btn = dr.find_element(By.XPATH,'/html/body/div[1]/div/div/div/div[2]/div/div/div[1]/div/div[3]/div/div/div/div/div[2]/div/article/div/div[2]/div/div/div[2]/div[1]/ul/ul[{}]/li/ul/li/div'.format(str(cnt)))
                more_btn.click()
            except Exception as e:
                print(cnt)
                print(e)
                pass
            cnt += 1
        except:
            cnt -= 1
            if cnt == 1:
                used_tags = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[1]/div/div[3]/div/div/div/div/div[2]/div/article/div/div[2]/div/div/div[2]/div[1]/ul/ul/li/ul/div/li/div/div/div[2]/div[1]/span')
                try : 
                    used_tags2 = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[1]/div/div[3]/div/div/div/div/div[2]/div/article/div/div[2]/div/div/div[2]/div[1]/ul/ul/li/ul/div[2]/li/div/div/div[2]/div[1]/span')
                except:
                    pass
            else:
                used_tags = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[1]/div/div[3]/div/div/div/div/div[2]/div/article/div/div[2]/div/div/div[2]/div[1]/ul/ul[{}]/li/ul/div/li/div/div/div[2]/div[1]/span'.format(str(cnt)))
                try : 
                    used_tags2 = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[1]/div/div[3]/div/div/div/div/div[2]/div/article/div/div[2]/div/div/div[2]/div[1]/ul/ul[{}]/li/ul/div[2]/li/div/div/div[2]/div[1]/span'.format(str(cnt)))
                except:
                    pass
            
            if used_tags2 != '':
                used_tags = used_tags.text + ' ' + used_tags2.text
            else:
                used_tags = used_tags.text
            break
    
    # Delete '#' And Put ',' Between The Tags
    used_tags = used_tags.replace('@czechmate.hufs ','')
    used_tags = used_tags.replace('#','')
    used_tags = used_tags.replace(' ',', ')
    print('Hashtags Loaded')
    print(used_tags)
    
    # Click View Insight
    insight_button = '/html/body/div[1]/div/div/div/div[2]/div/div/div[1]/div/div[3]/div/div/div/div/div[2]/div/article/div/div[2]/div/div/div[2]/section[1]/div/section/button'
    dr.find_element(By.XPATH, insight_button).click()
    
    # Ad or Not - 광고 글의 인사이트와 아닌 글의 인사이트 XPATH 다름 / Ad Posts have different XPATH from normal ones
    try: # Normal Post
        dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[10]/div/div/div/div[1]/div/div/span')
        l = 4
        n = 6
        m = 8
        ad = 'O'
        PeopleReached = 0
    except: # Ad
        l = 3
        n = 5
        m = 7
        ad = 'X'
        Accountsreached = 0

    print(ad)

    # Get Insights

    # Liked
    like = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[{}]/div/div/div/div/div[1]/div[2]/span'.format(str(l))).text
    #/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[4]/div/div/div/div/div[1]/div[2]/span

    # Comments
    cmt = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[{}]/div/div/div/div/div[2]/div[2]/span'.format(str(l))).text

    # DMs
    dm = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[{}]/div/div/div/div/div[3]/div[2]/span'.format(str(l))).text

    # Scrapted
    scrp = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[{}]/div/div/div/div/div[4]/div[2]/span'.format(str(l))).text
    
    # 반응 메인 데이터 / Main Interactions
    react = {}
    r_text = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[{}]/div/div/div/div[2]/div/span[2]'.format(str(n))).text
    r_num = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[{}]/div/div/div/div[2]/div/span[1]'.format(str(n))).text
    r_text = r_text.replace(' ','')
    r_num = r_num.replace(',','') #천단위 쉼표 들어있으면 에러남 / Comma Causes Error

    if r_text != 'Actionstakenfromthispost': # 이 게시물에서 발생한 행동의 경우 반응 기타 데이터의 합으로 필요가 없음 / No Need
        react[r_text] = r_num

    # 반응 기타 데이터 / Other Interactions
    cnt = 3
    while True:
        try:
            if ad == 'O': # 광고가 들어간 인사이트의 XPATH에는 div[1]이라는 다른 경로가 추가됨
                xpath = '/div[1]'
            else : 
                xpath = ''
            
            text = '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[{}]/div/div/div/div[{}]{}/span[1]'.format(str(n),str(cnt),xpath)
            num = '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[{}]/div/div/div/div[{}]{}/span[2]'.format(str(n),str(cnt),xpath)
            key = dr.find_element(By.XPATH,text).text
            key = key.replace(' ','')
            val = dr.find_element(By.XPATH, num).text
            val = val.replace(',','')
            react[key] = val
            cnt += 1
        except:
            break

    # Main Discovery
    explr = {}
    e_text = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[{}]/div/div/div/div[2]/div/span[2]'.format(str(m))).text
    e_text = e_text.replace(' ','')
    e_num = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[{}]/div/div/div/div[2]/div/span[1]'.format(str(m))).text
    e_num = e_num.replace(',','')
    explr[e_text] = e_num

    # Other Discovery
    cnt = 3
    while True:
        try:
            if (ad == 'O') & (cnt == 3):
                """
                발견 데이터는 cnt가 높아져 밑으로가면 다시 원래대로 변함
                #/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[8]/div/div/div/div[3]/div[1]/span[2]
                #/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[8]/div/div/div/div[4]/span[2]
                """
                xpath = '/div[1]'
            else : 
                xpath = ''
            text = '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[{}]/div/div/div/div[{}]{}/span[1]'.format(str(m),str(cnt),xpath)
            num = '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[{}]/div/div/div/div[{}]{}/span[2]'.format(str(m),str(cnt),xpath)
            key = dr.find_element(By.XPATH, text).text
            key = key.replace(' ','')
            val = dr.find_element(By.XPATH, num).text
            val = val.replace(',','')
            explr[key] = val
            cnt += 1
        except:
            break
    print('Insights Loaded')

    # Ad Fee
    if ad == 'O' :
        cost = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[2]/div/div/div[2]/div/div/div[10]/div/div/div/div[3]/div[1]/span[2]').text

    # Make Variables With Dictionary 'key = value'
    for k, v in react.items():
        exec('%s = %s' % (k, v))

    for k, v in explr.items():
        exec('%s = %s' % (k, v))

    # Date of Today
    date = datetime.today().strftime("%Y-%m-%d")
    print(date)

    # Save in Google Spreadsheets
    try :
        worksheet.append_row(['',title, date, uptime, like, cmt, dm, scrp, Adtaps, ProfileVisits, Websitetaps, Emailbuttontaps,
        PeopleReached, Impressions, FromExplore, FromHome, FromHashtags, FromProfile, FromOther, Follows, ad, cost, used_tags])
    except :
        worksheet.append_row(['',title, date, uptime, like, cmt, dm, scrp, Adtaps, ProfileVisits, Websitetaps, Emailbuttontaps,
        Accountsreached, Impressions, FromExplore, FromHome, FromHashtags, FromProfile, FromOther, Follows, ad, cost, used_tags])
    print('Saved in Google Spreadsheets - post no.' + count)

    # Close Insights
    dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[2]/div/div/div[1]/div/div[2]/div/div/div/div/div[2]/div/div/div[1]/div/div[3]/div/button').click()
    
    # Move to Next Post
    btn = dr.find_element(By.XPATH, '/html/body/div[1]/div/div/div/div[2]/div/div/div[1]/div/div[3]/div/div/div/div/div[1]/div/div/div[1]/button')
    btn.click()
    print('Next Post')

    count -= 1
    if count == 0:
        print('----------------Crawling Completed----------------')
        dr.quit()
        break
~~~
