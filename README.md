# PRO-CHN
from selenium import webdriver
#from selenium.webdriver.firefox.options import Options
from selenium.webdriver.chrome.options import Options
import pandas as pd
import re
import sys

#set this to something like "C:\\Libs\\chromedriver.exe"
#geckodriver = "C:\\Libs\\geckodriver.exe"
chromedriver = "E:\\UpW\\Excel\\Others\\Jason Debruyn\\chromedriver.exe"

def get_pages():
    pages = []
    link = "https://propertydevelopment.ssc.nsw.gov.au/T1PRPROD/WebApps/eproperty/P1/eTrack/eTrackApplicationSearchResults.aspx?Field=D&Period=LM&Group=DA&SearchFunction=SSC.P1.ETR.SEARCH.DA&r=SSC.P1.WEBGUEST&f=SSC.ETR.SRCH.DLM.DA&ResultsFunction=SSC.P1.ETR.RESULT.DA"
    options = Options()
    options.add_argument("--headless")
    options.add_argument('--disable-gpu')
    driver = webdriver.Chrome(chromedriver, chrome_options=options)
    print("Chrome Browser Invoked")
    driver.get(link)
    try:
        while True:
            pages.append(driver.find_element_by_id("ctl00_Content_cusResultsGrid_repWebGrid_ctl00_grdWebGridTabularView").get_attribute('innerHTML'))
            buttons_el = driver.find_elements_by_xpath('//tr[@class="pagerRow"]//tr//td')
            buttons_cout = len(buttons_el)
            for ind in range(buttons_cout):
                if len(buttons_el[ind].find_elements_by_tag_name("span"))>0:
                    if(ind+1>=buttons_cout):
                        driver.quit()
                        return pages
                    next_page_button = buttons_el[ind+1]
            next_page_button.click()
    except Exception as err: 
        print("An error occured: ", err)
    finally:
        driver.quit()
	
def main(args):
	
    if(len(args)>0): #check if user passed out file name
        result_file = args[0];
    else:
        result_file = "result.csv"
    print("Getting pages...")
    p = get_pages()

    #regex to extract only needed table
    rgx = re.compile("Lodgement Date|Description|Group Description|Category Description|Formatted Address|Applicant Names|Decision|Determined Date")

    result = pd.DataFrame()
    for t in p:
        t = "<table>"+t+"</table>" #we have droped table tag, now we surround table with it so pd can parse it correct 
        h = pd.read_html(t,match=rgx)
        result = result.append(h[0].iloc[1:-3,0:-3]) #we just don't need last 3 rows and cols and first row of table
	
	#set columns names
    result.columns=["Application Link", "Lodgement Date","Description","Group Description","Category Description","Formatted Address","Applicant Names","Decision","Determined Date"]
    result = result.reset_index() #as we appended many DataFraes in one, we got broken indexation, so we reset it
    result['Application Link'] = result['Application Link'].apply(lambda x: x[x.rfind(' '):])#cut application link
    result = result.drop("index",axis=1) #drop old indexes

    try:
        result.to_csv(result_file, index=False)
    except:
        print("Unable to save result file")
    print(f"All done. Your file is in {result_file}")
	
if __name__ == "__main__":
    main(sys.argv[1:])
