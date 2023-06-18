# Data Science jobs in Canada in 2023. Analysis of Job Postings from LinkedIn

This is the project containing 3 parts:
1. Scraping linkedin job list using LinkedIn job search engine which provide Job Title, Company Name, Location and URL of each job.
2. Scraping job posting including job description, skill set, number of applied applicants, type of workplace (Remote, On-site, Hybrid), job level, job type (Contract, Full-time, Part-time, Internship etc.), industry of the hiring company, how long it was posted.
3. Analysis of job market, including geo analysis, job market structure, skills and competences analysis and analysis of popular programming languages and tools.

The main article is provided here: https://orlovtsu.github.io/job_postings_analysis.html.

# Job List from LinkedIn
This is the first step of job market analysis provided in the article: https://orlovtsu.github.io/job_postings_analysis.html.

If you use this code for scraping data from LinkedIn be aware about the LinkedIn Term of Use and be sure that you do not violate it.

## Import Libraries
1. Selenium is a tool for automating web browsers, and these modules allow you to interact with web elements, locate elements by various criteria, and simulate keyboard actions.
2. BeautifulSoup module, which is used for parsing HTML and XML documents, provides convenient methods for extracting data from web pages.
3. Pandas library is a powerful data manipulation and analysis tool. It provides data structures and functions for efficiently handling structured data.
4. Time module provides functions for working with time-related operations, such as delays and timestamps.


```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from bs4 import BeautifulSoup
import pandas as pd
import time
from random import randint
```

## Initialization 
This next chunk assigns the path to the ChromeDriver executable to the variable chromedriver_path. The ChromeDriver is a separate executable that is required when using Selenium with Google Chrome. It acts as a bridge between the Selenium WebDriver and the Chrome browser, allowing automated interactions with the browser.

In this case, the chromedriver_path is set to './chromedriver', indicating that the ChromeDriver executable is located in the current directory (denoted by '.') and its filename is chromedriver. The specific path may vary depending on the actual location of the ChromeDriver executable on your system.

Make sure to provide the correct path to the ChromeDriver executable file in order for Selenium to work properly with Google Chrome.

For more details: https://chromedriver.chromium.org/downloads


```python
chromedriver_path = './chromedriver'
```

The next chunk create an instance of ChromeOptions class from the Selenium webdriver module. ChromeOptions allows you to customize the behavior of the Chrome browser when it is launched. In this case, the --start-maximized argument is added to the options, which instructs Chrome to start in maximized window mode.

This code also creates an instance of the webdriver.Chrome class, passing the options object and chromedriver_path as arguments. It initializes the Chrome webdriver, using the ChromeDriver executable located at chromedriver_path and applying the specified options for Chrome's behavior. Code sets an implicit wait time of 10 seconds for the driver object. The implicit wait instructs Selenium to wait for a certain amount of time when trying to locate elements on the web page. It allows the driver to wait for a specified duration before throwing a NoSuchElementException if the element is not immediately available. In this case, the implicit wait is set to 10 seconds.


```python

options = webdriver.ChromeOptions()
options.add_argument("--start-maximized")

driver = webdriver.Chrome(options=options, executable_path=chromedriver_path)

driver.implicitly_wait(10)
```

Following chunk opens the page where you should authorize and changes the scale of viewing page.


```python
# Open the web page with the login and password fields
# Enter your username and password to authenticate as a LinkedIn user
driver.get('https://www.linkedin.com/login')


# To speed up the downloading process for scraping the page content, it is recommended to reduce the page scale to 25%.
# This will result in faster download of the majority of the content you require.
driver.execute_script("document.body.style.zoom = '25%'")
```

## Job List Scraping script:


```python
# Set the search query and location
query = '"BI"'
location = 'Canada'

# Create an empty list to store the job data
data = []

# Loop through multiple pages
for page_num in range(1, 40):
    # Construct the URL for each page based on the query, location, and page number
    url = f'https://www.linkedin.com/jobs/search/?keywords={query}&location={location}&start={25 * (page_num - 1)}'

    # Decrease the page scale to 25% for faster content downloading    
    driver.execute_script("document.body.style.zoom = '25%'")

    # Open the URL in the web driver
    driver.get(url)
    
    # Pause for a random time between 10 and 20 seconds
    time.sleep(randint(10,20))  
    
    # Parse the page source using BeautifulSoup
    soup = BeautifulSoup(driver.page_source, 'html.parser')
    
    # Find all job postings on the page
    job_postings = soup.find_all('li', {'class': 'jobs-search-results__list-item'})
    
    # Print job titles for each job posting
    for k, job in enumerate(job_postings):
        try:                
            print(k, ':', job.find('a', class_='job-card-list__title').get_text().strip())
        except:
            print(None)
            
    print('-----')
    
    # Extract relevant information from each job posting and store it in a list of dictionaries
    for i in range(len(job_postings)):
        j = 0
        while ((job_postings[i].find('a', class_='job-card-list__title')== None) & (j < 10)):
            print(i,': Attempt again -', j)
            job_postings = soup.find_all('li', {'class': 'jobs-search-results__list-item'})
            for k, job in enumerate(job_postings):
                try:
                    print(k, ':', job.find('a', class_='job-card-list__title').get_text().strip())
                except:
                    print(None)            
            print('-----')
            j += 1
            if j == 4:
                i += 1
                j = 0
        
        job_posting = job_postings[i]

        # Extract job title, company name, location, and URL from the job posting
        try:
            job_title = job_posting.find('a', class_='job-card-list__title').get_text().strip()
        except AttributeError:
            job_title = None
        
        # Extract company name. Depending of personal account settings, name of html tags and html structure may vary. 
        # If this line does not work, uncomment any of other lines to try again
        try:
            #company_name = job_posting.find('span', class_='job-card-container__primary-description ').get_text().strip()
            company_name = job_posting.find('div', class_='job-card-container__company-name').text.strip()
            #company_name = job_posting.find('span', class_='job-card-container__primary-description').get_text(strip=True)
        except AttributeError:
            company_name = None
        print(i, company_name)
        try:
            job_location = job_posting.find('li', class_='job-card-container__metadata-item').get_text().strip()
        except AttributeError:
            job_location = None

        try:
            URL = job_posting.find('a', class_='job-card-container__link', href=True).get('href')
        except AttributeError:
            URL = None

        try:
            job_next = job_postings[i+1].find('a', class_='job-card-list__title').get_text().strip()
        except:
            job_next = None
        
        if not job_next:
            URL1 = '/'.join(URL.split('/')[0:4])
            button = driver.find_element(by=By.XPATH, value = f"//a[contains(@href, '{URL1}')]")
            button.click()
      
            time.sleep(2)
            current_url = driver.current_url   
            
            driver.get(current_url)
            
            if ((i < len(job_postings)-1)):
                time.sleep(randint(10,20))  # Wait for 20 seconds
                soup = BeautifulSoup(driver.page_source, 'html.parser')        
                j = 0
                while ((job_postings[i+1].find('a', class_='job-card-list__title')== None) & (j < 10)):
                    print(i,': Attempt again')
                    job_postings = soup.find_all('li', {'class': 'jobs-search-results__list-item'})
                    for k, job in enumerate(job_postings):
                        try:
                            print(k, ':', job.find('a', class_='job-card-list__title').get_text().strip())
                        except:
                            print(None)                
                    print('-----')
                    j += 1
                    if j == 4:
                        i += 1
                        j = 0
        # Append the extracted job information to the data list   
        data.append({
            'Job Title': job_title,
            'Company Name': company_name,
            'Location': job_location,
            'URL': URL,
         })
        
        # Pause for 5 seconds
        time.sleep(5) 
```

Depending of an account and some personal settings, the strucure of HTML page may vary and this is why sometimes the previous chink may finish by any type of error. This is why, it should be tune sometimes and DataFrame should be saved to save data and not to repeat the scraping you already did.

## Results saving
Do not forget to save you results:


```python
# Create a DataFrame from the collected job data        
df = pd.DataFrame(data)
df.to_csv('job_list_all.csv')
```

# 2. Job Posting from LinkedIn

This is the code provides the script for scraping job posting including job description, skill set, number of applied applicants, type of workplace (Remote, On-site, Hybrid), job level, job type (Contract, Full-time, Part-time, Internship etc.), industry of the hiring company, how long it was posted. 

This is the second step of job market analysis provided in the article: https://orlovtsu.github.io/job_postings_analysis.html.

If you use this code for scraping data from LinkedIn be aware about the LinkedIn Term of Use and be sure that you do not violate it.

## Import Libraries
1. Selenium is a tool for automating web browsers, and these modules allow you to interact with web elements, locate elements by various criteria, and simulate keyboard actions.
2. BeautifulSoup module, which is used for parsing HTML and XML documents, provides convenient methods for extracting data from web pages.
3. Pandas library is a powerful data manipulation and analysis tool. It provides data structures and functions for efficiently handling structured data.
4. Time module provides functions for working with time-related operations, such as delays and timestamps.


```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from bs4 import BeautifulSoup
import pandas as pd
import time
from random import randint
```

## Initialization 
This next chunk assigns the path to the ChromeDriver executable to the variable chromedriver_path. The ChromeDriver is a separate executable that is required when using Selenium with Google Chrome. It acts as a bridge between the Selenium WebDriver and the Chrome browser, allowing automated interactions with the browser.

In this case, the chromedriver_path is set to './chromedriver', indicating that the ChromeDriver executable is located in the current directory (denoted by '.') and its filename is chromedriver. The specific path may vary depending on the actual location of the ChromeDriver executable on your system.

Make sure to provide the correct path to the ChromeDriver executable file in order for Selenium to work properly with Google Chrome.

For more details: https://chromedriver.chromium.org/downloads


```python
chromedriver_path = './chromedriver.exe'
```

The next chunk create an instance of ChromeOptions class from the Selenium webdriver module. ChromeOptions allows you to customize the behavior of the Chrome browser when it is launched. In this case, the --start-maximized argument is added to the options, which instructs Chrome to start in maximized window mode.

This code also creates an instance of the webdriver.Chrome class, passing the options object and chromedriver_path as arguments. It initializes the Chrome webdriver, using the ChromeDriver executable located at chromedriver_path and applying the specified options for Chrome's behavior. Code sets an implicit wait time of 10 seconds for the driver object. The implicit wait instructs Selenium to wait for a certain amount of time when trying to locate elements on the web page. It allows the driver to wait for a specified duration before throwing a NoSuchElementException if the element is not immediately available. In this case, the implicit wait is set to 10 seconds.


```python
options = webdriver.ChromeOptions()
options.add_argument("--start-maximized")

driver = webdriver.Chrome(options=options)

driver.implicitly_wait(10)
```

Following chunk opens the page where you should authorize and changes the scale of viewing page.


```python
# Open the web page with the login and password fields
# Enter your username and password to authenticate as a LinkedIn user
driver.get('https://www.linkedin.com/login')


# To speed up the downloading process for scraping the page content, it is recommended to reduce the page scale to 25%.
# This will result in faster download of the majority of the content you require.
driver.execute_script("document.body.style.zoom = '25%'")
```

## Job Posting Scraping script:


```python
# Read the job list from a CSV file
job_list = pd.read_csv('job_list_all.csv')
data = []

# Iterate through each job in the job list starting from index 526
for index, job in job_list[526:].iterrows():
    URL = 'https://www.linkedin.com' + '/'.join(job['URL'].split('/')[0:4]) # Create the URL for each job
    driver.get(URL)  # Open the URL in the web driver
    time.sleep(randint(4,10))  # Pause for a random time between 4 and 10 seconds
    
    soup = BeautifulSoup(driver.page_source, 'html.parser')# Parse the page source using BeautifulSoup
    
    job_location = job['Location']  # Extract job location from the job list
    job_title = job['Job Title']  # Extract job title from the job list
    company_name = job['Company Name']  # Extract company name from the job list

    try:
        no_longer_message =  soup.find('span', class_ = 'artdeco-inline-feedback__message').get_text(strip = True)
        print(index, no_longer_message)
        no_longer = 'No longer' in no_longer_message  # Check if the job posting is no longer available
    except:
        no_longer = False
        
    if not no_longer:
        job_insights = soup.find_all('li', {'class': 'jobs-unified-top-card__job-insight'})# Find job insights
 
        # Extract job type and job level from job insights
        try:
            job_type = job_insights[0].find('span').get_text(strip = True).split('·')[0]
        except:
            job_type = 'Not defined'
        try:
            job_level = job_insights[0].find('span').get_text(strip = True).split('·')[1]
        except:
            job_level = 'Not defined'

        # Extract company size and job industry from job insights
        try:
            company_size = job_insights[1].find('span').get_text(strip = True).split('·')[0]
        except:
            company_size = 'Not defined'
        try:
            job_industry = job_insights[1].find('span').get_text(strip = True).split('·')[1]
        except:
            job_industry = 'Not defined'

        # Extract number of job applicants
        try:
            for insight in job_insights[2:]:  
                text = insight.find('span').get_text(strip = True)
                if 'applicants' in text:
                    job_applicants = text.split(' ')[5]
                    break
        except:
            try:
                job_applicants = soup.find('span', {'class': 'jobs-unified-top-card__applicant-count'}).get_text(strip = True).split(' ')[0]
            except:
                try:
                    span_element = soup.find('span', class_='jobs-unified-top-card__subtitle-secondary-grouping')
                    job_applicants = span_element.find('span', class_='jobs-unified-top-card__bullet').get_text(strip=True)
                except:
                    job_applicants = 'Not defined'
        
        try:
            posted = soup.find('span', {'class': 'jobs-unified-top-card__posted-date'}).get_text(strip = True)
        except:
            posted = 'Not defined'
        try:
            workplacetype = soup.find('span', {'class': 'jobs-unified-top-card__workplace-type'}).get_text(strip = True)
        except:
            workplacetype = 'Not defined'
            
        # Click the "Skills" button to view job skills
        try:
            button_skills = driver.find_element(by=By.CLASS_NAME, value = 'jobs-unified-top-card__job-insight-text-button')
            button_skills.click()
            time.sleep(randint(4,10))
            soup_skills = BeautifulSoup(driver.page_source, 'html.parser')   
            skills = soup_skills.find_all('li', {'class': 'job-details-skill-match-status-list__unmatched-skill'})
            skill_set = ''
            for skill in skills:
                div_element = skill.find('div', class_='display-flex')  # Example: Locating the div based on the 'display-flex' class
                skill_text = div_element.get_text(strip=True)
                skill_set = skill_set + skill_text + '; '

            button_exit = driver.find_element(By.XPATH, "//span[text()='Done']")
            button_exit.click()
        except:
            skill_set = ''
        
        # Click the "More" button to view full job description
        try:
            button_more = driver.find_element(by=By.CLASS_NAME, value = 'jobs-description__footer-button')
            button_more.click()
            time.sleep(randint(4,10))
            soup_more = BeautifulSoup(driver.page_source, 'html.parser')  
        except:
            button_more = None
                
        # Extract job description 
        try:
            description = soup.find('div', {'class': 'jobs-box__html-content'}).get_text(strip = True)
        except:
            description = 'Not defined'
        
        # Output for checking the progress
        print(index, job_title, company_name)
        
        # Append the extracted job information to the data list
        data.append({
            'Job Title': job_title, #+
            'Company Name': company_name, 
            'Company Size': company_size, 
            'Location': job_location, 
            'URL': URL, 
            'Workplace_Type': workplacetype, 
            'Posted': posted,
            'Applicants': job_applicants,
            'Industry': job_industry, 
            'Job level': job_level,
            'Job type': job_type, 
            'Skillset': skill_set,
            'Description': description
        })

```

## Results saving
Do not forget to save you results:


```python
# Create a DataFrame from the collected job data
df = pd.DataFrame(data)
df.to_csv('job_postings.csv')
```



# 3. LinkedIn Job Analysis 

This is the third step of job market analysis provided in the article: https://orlovtsu.github.io/job_postings_analysis.html.

If you use this code for scraping data from LinkedIn be aware about the LinkedIn Term of Use and be sure that you do not violate it.

## Import Libraries


```python
import pandas as pd
import numpy as np
import plotly.express as px
import matplotlib.pyplot as plt
import plotly.graph_objects as go
import json
import plotly.colors as colors
import operator
```

## Data Reading 
Read and concatinate the results from previous part to entire dataset


```python
df1 = pd.read_csv('job_list_1.csv')
df2 = pd.read_csv('job_list_2.csv')
df3 = pd.read_csv('job_list_3.csv')
```


```python
df = pd.concat([df1, df2, df3], ignore_index = True)
df
```






```python
df = df.dropna()
```

# Data Preparation


Define the Data Science job types to classify all jobs through the dataset


```python
ds_job_types = [
    'Data Scientist',
    'Data Analyst',
    'Data Engineer',
    'Machine Learning Engineer',
    'Product Manager',
    'Project Manager', 
    'Program Manager',
    'Technical Manager',
    'Business Intelligence',
    'Manager',
    'Software Engineer',
    'Product Analyst',
    'HR Manager',
    'DS/AI Expert',
    'Data Rater',
    'Not Data or AI job'
]
```

The main chunk is responsible for labeling and classifying job classes. This step can be challenging as it requires significant manual effort and a deep understanding of job classes by analyzing the job descriptions and titles. This chunk needs to be iterated multiple times as necessary to accurately label and classify the jobs.


```python
df.loc[((df['Job Title'].str.lower().str.contains('base')) & 
        (df['Job Title'].str.lower().str.contains('data')) & 
        (~df['Job Class'].isin(ds_job_types))) , 'Job Class'] = ds_job_types[2]#'Data Engineer'
```

Checking not classified rows:


```python
df.loc[pd.isnull(df['Job Class']), 'Job Class'] 
```

Checking classified jobs statistic:


```python
df['Job Class'].value_counts()
```

### Translate titles from French to English using OpenAI API

I encountered some job titles that were written in French. However, since I don't know French, I utilized the OpenAI library and OpenAI API to translate the job titles from French to English. This allowed me to understand the nature of each job and gather relevant information. Special thanks for DeepLearning.AI (http://deeplearning.ai), which provided the free course about prompt engineering and usage of OpenAI API. 


```python
import openai
import os

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv()) # read local .env file

os.environ["OPENAI_API_KEY"] = "YOUR_OPENAI_API_KEY"   # From OpenAI account
openai.organization = 'YOUR_ORGANIZATION_KEY' # From OpenAI account
openai.api_key = os.getenv("OPENAI_API_KEY")
openai.api_key
```


```python
def get_completion(prompt, model="gpt-3.5-turbo",temperature=0): 
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=temperature, 
    )
    return response.choices[0].message["content"]
```


```python
french_titles = df.loc[pd.isnull(df['Job Class']), 'Job Title']
titles = '; '.join(french_titles)
```


```python

prompt = f"""
You are french-english translator. 
Translate job titles from French ```{titles}``` to English and return absolutely all values without passes as string and separated by ; 
Do not pass any values 
"""

response = get_completion(prompt)
answer = response.split(';')
```


```python
df_titles = pd.DataFrame(french_titles)
df_titles['English title'] = answer
df.loc[pd.isnull(df['Job Class']), 'Job Title2'] = df_test['English title']
```

Save your results sometimes because you did so many work for classification.


```python
df.to_csv('df_all_jobs_labeled.csv')
```

The same job should be done for Job Levels. It's easier because usually it's defined in one or more filds of each row.


```python
df.loc[((df['Job level'].str.lower().str.contains('full-time')) &
        (df['Job Title'].str.lower().str.contains('manager')) &
        (pd.isnull(df['JobLevel']))), 'JobLevel'] = 'Manager'
```


```python
df[(df['Job level'] != 'Not defined') & (pd.isnull(df['JobLevel']))]
```


```python
df['Job level'].value_counts()
```




    Mid-Senior level    1643
    Entry level         1097
    Not defined          265
    Associate            137
    Director             122
    Internship           116
    Manager               94
    Executive             39
    Lead                  31
    Principal             16
    Staff                 15
    Name: Job level, dtype: int64



Save your results sometimes because you did so many work for classification.


```python
df.to_csv('df_all_jobs_labeled.csv')
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    /var/folders/9z/scwdbhm50c1848gsllv1nqk80000gn/T/ipykernel_22875/797945982.py in <module>
    ----> 1 df.to_csv('df_all_jobs_marked.csv')
    

    NameError: name 'df' is not defined


## Analysis 
### 1. What Data jobs are in demand in Canada in 2023?


```python
df = pd.read_csv('df_all_jobs_labeled.csv')
```


```python
df1 = df[df['Job Class'] != 'Not Data or AI job']
```

First of all, the most interesting question is what type of Data Science job is most hired by companies and how the proportion of each job is distributed in Canada in June 2023.


```python
df11 = df1['Job Class'].value_counts()
df11 = pd.DataFrame(df11)
df11['Job Name'] = df11.index
df11 = df11[:8] # We are interesting in Top 8 jobs, but you can be interesting in other specific number of jobs
df11
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Job Class</th>
      <th>Job Name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Data Engineer</th>
      <td>821</td>
      <td>Data Engineer</td>
    </tr>
    <tr>
      <th>Data Analyst</th>
      <td>663</td>
      <td>Data Analyst</td>
    </tr>
    <tr>
      <th>Data Scientist</th>
      <td>448</td>
      <td>Data Scientist</td>
    </tr>
    <tr>
      <th>Software Engineer</th>
      <td>255</td>
      <td>Software Engineer</td>
    </tr>
    <tr>
      <th>Machine Learning Engineer</th>
      <td>180</td>
      <td>Machine Learning Engineer</td>
    </tr>
    <tr>
      <th>Product Manager</th>
      <td>76</td>
      <td>Product Manager</td>
    </tr>
    <tr>
      <th>Data Rater</th>
      <td>75</td>
      <td>Data Rater</td>
    </tr>
    <tr>
      <th>Business Intelligence</th>
      <td>59</td>
      <td>Business Intelligence</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Defining color palette for the sunflowers
sunflowers_colors = ['rgb(177, 127, 38)', 'rgb(205, 152, 36)', 'rgb(99, 79, 37)',
                     'rgb(129, 180, 179)', 'rgb(124, 103, 37)','rgb(33, 75, 99)', 'rgb(79, 129, 102)', 'rgb(151, 179, 100)',
                 'rgb(175, 49, 35)', 'rgb(36, 73, 147)', 'rgb(146, 123, 21)', 'rgb(177, 180, 34)', 'rgb(206, 206, 40)',
                'rgb(175, 51, 21)', 'rgb(35, 36, 21)']

# Creating a pie chart figure
fig = go.Figure(data=[go.Pie(labels=df11['Job Name'], values=df11['Job Class'], hole=.3, 
                             marker_colors=sunflowers_colors)])

# Updating the trace properties
fig.update_traces(textposition='inside', textinfo='percent+label')

# Updating the layout
fig.update_layout(
    title={
        'text': "Top 8 Data Jobs in-demand in Canada",
        'y': 0.98,
        'x': 0.4,
        'xanchor': 'center',
        'yanchor': 'top'
    }
)
fig.update_layout(margin=dict(t=30, b=0, l=0, r=0))

# Displaying the figure
fig.show()

# Saving the figure as an HTML file
fig.write_html("plot1.html")
```
[plot1](plot1.png)

The first key insight is the Data Engineer, Data Scientist and Data Analyst jobs take more than 70% of all Data Science job market. These jobs are mostly in demand now and the most demanded job is Data Engineer with 31% of the job market. So, if you are a Data Engineer or planning to become a Data Engineer, this is the hottest job now in our field. Data Analyst and Data Scientist are still on top of Data Science jobs. And the insightful thing that Software Engineers are hired by companies mostly for developing AI tools together with Machine Learning Engineers. Product Managers is also a popular job based on job posting data, such as Data Entry position. Actually, Data Entry job is hardly to classify by Data Science role, but since this work is data related and quite popular, I decided to include it in this study. And BI has not a huge proportion in our dataset but maybe the reason is in the title of jobs and mostly BI tasks are included in the Data Analyst positions.

Along with the question of who is in demand, a good question is what level companies need. And the answer to this question is very clear. 


```python
df12 = df1['Job level'].value_counts()
df12 = pd.DataFrame(df12)
df12['Job Level'] = df12.index
df12.rename(columns = {'Job level': 'Count'})
df12
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Job level</th>
      <th>Job Level</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Mid-Senior level</th>
      <td>1446</td>
      <td>Mid-Senior level</td>
    </tr>
    <tr>
      <th>Entry level</th>
      <td>623</td>
      <td>Entry level</td>
    </tr>
    <tr>
      <th>Not defined</th>
      <td>204</td>
      <td>Not defined</td>
    </tr>
    <tr>
      <th>Director</th>
      <td>114</td>
      <td>Director</td>
    </tr>
    <tr>
      <th>Associate</th>
      <td>105</td>
      <td>Associate</td>
    </tr>
    <tr>
      <th>Manager</th>
      <td>85</td>
      <td>Manager</td>
    </tr>
    <tr>
      <th>Internship</th>
      <td>78</td>
      <td>Internship</td>
    </tr>
    <tr>
      <th>Executive</th>
      <td>34</td>
      <td>Executive</td>
    </tr>
    <tr>
      <th>Lead</th>
      <td>30</td>
      <td>Lead</td>
    </tr>
    <tr>
      <th>Staff</th>
      <td>15</td>
      <td>Staff</td>
    </tr>
    <tr>
      <th>Principal</th>
      <td>14</td>
      <td>Principal</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Defining color palette for the sunflowers
sunflowers_colors = ['rgb(79, 129, 102)', 'rgb(151, 179, 100)',
                     'rgb(175, 49, 35)', 'rgb(36, 73, 147)', 'rgb(146, 123, 21)', 'rgb(177, 180, 34)', 'rgb(206, 206, 40)',
                     'rgb(175, 51, 21)', 'rgb(35, 36, 21)', 'rgb(177, 127, 38)', 'rgb(205, 152, 36)', 'rgb(99, 79, 37)',
                     'rgb(129, 180, 179)', 'rgb(124, 103, 37)', 'rgb(33, 75, 99)']

# Creating a pie chart figure
fig = go.Figure(data=[go.Pie(labels=df12['Job Level'], values=df12['Job level'], hole=.3, marker_colors=sunflowers_colors)])

# Updating the trace properties
fig.update_traces(textposition='inside', textinfo='percent+label')

# Updating the layout
fig.update_layout(
    margin=dict(t=30, b=0, l=0, r=0),
    title={
        'text': "Data Job Levels in Demand in Canada",
        'y': 0.98,
        'x': 0.43,
        'xanchor': 'center',
        'yanchor': 'top'
    }
)

# Displaying the figure
fig.show()

# Saving the figure as an HTML file
fig.write_html("plot2.html")
```
[plot2](plot2.png)
More than half (52,6%) of the positions are open to middle and senior level specialists. However, in 2023, Canada also has a significant number of positions for entry-level data specialists (22.7%) and associates (3.82%), as well as internships (2.84), most of which are co-op formats that are unique to Canada and are excellent opportunities for starting a career for young specialists. Data Science Directors are most demanded from leading positions with 4.15%.

And sometimes it's interesting to look in what industries all these specialists work.


```python
df13 = df1['Industry'].value_counts()[:20]
df13 = pd.DataFrame(df13)
df13['Job Industry'] = df13.index
df13
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Industry</th>
      <th>Job Industry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Software Development</th>
      <td>390</td>
      <td>Software Development</td>
    </tr>
    <tr>
      <th>IT Services and IT Consulting</th>
      <td>390</td>
      <td>IT Services and IT Consulting</td>
    </tr>
    <tr>
      <th>Financial Services</th>
      <td>263</td>
      <td>Financial Services</td>
    </tr>
    <tr>
      <th>Staffing and Recruiting</th>
      <td>181</td>
      <td>Staffing and Recruiting</td>
    </tr>
    <tr>
      <th>Technology, Information and Internet</th>
      <td>135</td>
      <td>Technology, Information and Internet</td>
    </tr>
    <tr>
      <th>Hospitals and Health Care</th>
      <td>125</td>
      <td>Hospitals and Health Care</td>
    </tr>
    <tr>
      <th>Banking</th>
      <td>92</td>
      <td>Banking</td>
    </tr>
    <tr>
      <th>Telecommunications</th>
      <td>88</td>
      <td>Telecommunications</td>
    </tr>
    <tr>
      <th>Biotechnology Research</th>
      <td>85</td>
      <td>Biotechnology Research</td>
    </tr>
    <tr>
      <th>Insurance</th>
      <td>70</td>
      <td>Insurance</td>
    </tr>
    <tr>
      <th>Government Administration</th>
      <td>66</td>
      <td>Government Administration</td>
    </tr>
    <tr>
      <th>Retail</th>
      <td>59</td>
      <td>Retail</td>
    </tr>
    <tr>
      <th>Not defined</th>
      <td>59</td>
      <td>Not defined</td>
    </tr>
    <tr>
      <th>Translation and Localization</th>
      <td>56</td>
      <td>Translation and Localization</td>
    </tr>
    <tr>
      <th>Information Technology &amp; Services</th>
      <td>48</td>
      <td>Information Technology &amp; Services</td>
    </tr>
    <tr>
      <th>Human Resources Services</th>
      <td>41</td>
      <td>Human Resources Services</td>
    </tr>
    <tr>
      <th>Business Consulting and Services</th>
      <td>36</td>
      <td>Business Consulting and Services</td>
    </tr>
    <tr>
      <th>Computer and Network Security</th>
      <td>35</td>
      <td>Computer and Network Security</td>
    </tr>
    <tr>
      <th>Pharmaceutical Manufacturing</th>
      <td>34</td>
      <td>Pharmaceutical Manufacturing</td>
    </tr>
    <tr>
      <th>Higher Education</th>
      <td>29</td>
      <td>Higher Education</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Defining color palette for the sunflowers
sunflowers_colors = ['rgb(79, 129, 102)', 'rgb(151, 179, 100)',
                     'rgb(175, 49, 35)', 'rgb(36, 73, 147)', 'rgb(146, 123, 21)', 'rgb(177, 180, 34)', 'rgb(206, 206, 40)',
                     'rgb(175, 51, 21)', 'rgb(35, 36, 21)', 'rgb(177, 127, 38)', 'rgb(205, 152, 36)', 'rgb(99, 79, 37)',
                     'rgb(129, 180, 179)', 'rgb(124, 103, 37)', 'rgb(33, 75, 99)']

# Creating a pie chart figure
fig = go.Figure(data=[go.Pie(labels=df13['Job Industry'], values=df13['Industry'], hole=.3, marker_colors=sunflowers_colors)])

# Updating the trace properties
fig.update_traces(textposition='inside', textinfo='percent+label')

# Updating the layout
fig.update_layout(
    margin=dict(t=30, b=0, l=0, r=0),
    title={
        'text': "Top 20 Industries Hiring Data Specialists in Canada",
        'y': 0.98,
        'x': 0.35,
        'xanchor': 'center',
        'yanchor': 'top'
    }
)

# Displaying the figure
fig.show()

# Saving the figure as an HTML file
fig.write_html("plot3.html")

```
[plot3](plot3.png)

And the answer is that 4 main industries takes more than 50% of Data Science jobs market: Software Development (17.1%), IT Service and Consulting (17.1%), Finance Service (11.5%) and Staffing and Recruiting (7.93%). The last industry is highly probable represented by intermediary companies that hire data scientists for their clients. The one additional insight that HealthCare industry has almost 5.5% of Data Science job market which is very high and characterizes that in Canada, the field of HealthCare and medicine is at a high-tech level.

# 2. How are Data jobs distributed through the Canada, industries and different companies?

But what if you are a young Data Science specialist who decides where to live, what is the city where you can find the most matching to you job. To answer this question let's prepare data for map plot.


```python
df2 = df1.copy()
df2 = df2[df2['Job Class'].isin(list(df11['Job Name'].values))]
```


```python
df2['Location'] = df1['Location'].str.split(',').str.get(-1).str.split(' ').str.get(1)
df2.loc[(df2['Location'] == 'Montreal'), 'Location'] = 'QC'
df2.loc[(df2['Location'] == 'Edmonton'), 'Location'] = 'AB'
df2.loc[(df2['Location'] == 'Calgary'), 'Location'] = 'AB'
df2.loc[(df2['Location'] == 'Vancouver'), 'Location'] = 'BC'
df2.loc[(df2['Location'] == 'Ottawa'), 'Location'] = 'ON'
df2.loc[(df2['Location'] == 'Quebec'), 'Location'] = 'QC'
df2.loc[(df2['Location'] == 'Winnipeg'), 'Location'] = 'MB'
df2.loc[(df2['Location'] == 'Regina'), 'Location'] = 'SK'
df2.loc[(df2['Location'] == 'Halifax'), 'Location'] = 'NS'
df2.loc[(df2['Location'] == 'Canada'), 'Location'] = 'Remote'
df2.loc[(df2['Location'] == '(Remote)'), 'Location'] = 'Remote'
```


```python
df2['City'] = df1['Location'].str.split(',').str.get(0)#.str.split(' ').str.get(1)
df2['City'].value_counts()[:50]
df2.loc[df2['City'].str.lower().str.contains('montreal'), 'City'] = 'Montreal'
df2.loc[df2['City'].str.lower().str.contains('toronto'), 'City'] = 'Toronto'
df2.loc[df2['City'].str.lower().str.contains('vancouver'), 'City'] = 'Vancouver'
df2.loc[df2['City'].str.lower().str.contains('namer'), 'City'] = 'Remote'
df2.loc[df2['City'].str.lower().str.contains('remote'), 'City'] = 'Remote'
cities = df2['City'].value_counts()[:20].index
df3 = df2[df2['City'].isin(cities)]
df21 = df3[['City', 'Job Class']].value_counts().reset_index()
df21 = df21.rename(columns = {0: 'Count'})
df21
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Job Class</th>
      <th>Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Toronto</td>
      <td>Data Engineer</td>
      <td>210</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Remote</td>
      <td>Data Engineer</td>
      <td>171</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Toronto</td>
      <td>Data Analyst</td>
      <td>126</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Remote</td>
      <td>Data Scientist</td>
      <td>110</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Montreal</td>
      <td>Data Analyst</td>
      <td>99</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>119</th>
      <td>Quebec</td>
      <td>Machine Learning Engineer</td>
      <td>1</td>
    </tr>
    <tr>
      <th>120</th>
      <td>Regina</td>
      <td>Data Rater</td>
      <td>1</td>
    </tr>
    <tr>
      <th>121</th>
      <td>Regina</td>
      <td>Product Manager</td>
      <td>1</td>
    </tr>
    <tr>
      <th>122</th>
      <td>Remote</td>
      <td>Business Intelligence</td>
      <td>1</td>
    </tr>
    <tr>
      <th>123</th>
      <td>Alberta</td>
      <td>Data Analyst</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>124 rows × 3 columns</p>
</div>




```python
df2['City'] = df1['Location'].str.split(',').str.get(0)#.str.split(' ').str.get(1)
df2['City'].value_counts()[:50]
df2.loc[df2['City'].str.lower().str.contains('montreal'), 'City'] = 'Montreal'
df2.loc[df2['City'].str.lower().str.contains('toronto'), 'City'] = 'Toronto'
df2.loc[df2['City'].str.lower().str.contains('vancouver'), 'City'] = 'Vancouver'
df2.loc[df2['City'].str.lower().str.contains('namer'), 'City'] = 'Remote'
df2.loc[df2['City'].str.lower().str.contains('remote'), 'City'] = 'Remote'
#df2.loc[df2['City'].str.lower().str.contains('new york'), 'City'] = 'Remote'
cities = df2['City'].value_counts()[:40].index
df3_ = df2[df2['City'].isin(cities)]
df25 = df3_['City'].value_counts().reset_index()
df25 = df25.rename(columns = {0: 'Count'})
#df25
#cities
```

Addition of Latitude and Longitude dataframe:


```python

cities_coords = {
    'City': ['Toronto', 'Remote', 'Montreal', 'Vancouver', 'Canada', 'Calgary', 'Mississauga', 'Ottawa', 'Québec',
             'Markham', 'Edmonton', 'Ontario', 'Winnipeg', 'British Columbia', 'Waterloo', 'North York', 'Regina',
             'Saskatoon', 'Alberta', 'Quebec', 'London', 'Dorval', 'Sherbrooke', 'Burnaby', 'Kitchener', 'Victoria',
             'Brampton', 'Montréal-Ouest', 'Manitoba', 'Richmond Hill', 'Kelowna', 'Vaughan', 'Laval', 'Surrey',
             'Burlington', 'Trois-Rivières', 'Midland', 'Gatineau', 'Halifax', 'Hamilton'],
    'Latitude': [43.651070, None, 45.501690, 49.282730, 56.130366, 51.044730, 43.589045, 45.421530, 46.813878,
                 43.854671, 53.546120, 51.253775, 49.895140, 53.726668, 43.464257, 43.761539, 50.454660, 52.134370,
                 53.933270, 46.829853, 42.981733, 45.457717, 45.403220, 49.282729, 43.450000, 48.428421, 43.683333,
                 45.470310, 45.963589, 43.840424, 49.282729, 45.594564, 49.895136, 49.104430, 48.428421, 48.438950,
                 45.347598, 45.411171, 44.648764, 44.648763],
    'Longitude': [-79.347015, None, -73.567253, -123.120735, -106.346771, -114.071890, -79.644120, -75.697193,
                  -71.207980, -79.383469, -113.490910, -85.323213, -97.138400, -127.647620, -80.520409, -79.411079,
                  -104.618896, -106.670890, -116.576508, -71.208908, -81.246229, -73.741778, -71.288780, -123.120735,
                  -80.488998, -123.365644, -79.650917, -73.636016, -66.469253, -73.480979, -123.120735, -73.739213,
                  -96.656105, -79.421157, -79.676920, -73.583529, -75.718209, -79.700996, -63.573467, -79.688692]
}

df26 = pd.DataFrame(cities_coords)
#df26

```

Merge data with coordinates and number of jobs in each city.


```python
df25 = df25.rename(columns = {'index': 'City', 'City':'Count'})
df27 = pd.merge(df25, df26, on = 'City')
df27
```

Prepare data for choropleth map with all provincies and territories.


```python
import plotly.graph_objects as go
import pandas as pd

# Sample data with latitude and longitude coordinates for cities in Canada

provinces = {
    'AB': 'Alberta',
    'BC': 'British Columbia',
    'MB': 'Manitoba',
    'NB': 'New Brunswick',
    'NL': 'Newfoundland and Labrador',
    'NS': 'Nova Scotia',
    'NT': 'Northwest Territories',
    'NU': 'Nunavut',
    'ON': 'Ontario',
    'PE': 'Prince Edward Island',
    'QC': 'Quebec',
    'SK': 'Saskatchewan',
    'YT': 'Yukon Territory'
}

df_provinces = pd.DataFrame(provinces.items(), columns=['Abbreviation', 'Province'])

```

Upload GEOJSON file with provincies boarders:


```python
with open('canada.geojson') as f:
    geojson_data = json.load(f)

```


```python
df27 = df27.drop(labels=4) # Remove Canada from cities list
```

Choropleth Map of Canadian provincies and territories and bubble charts with size of bubble depending on number of Data Jobs in each city or province. Sometimes city of job is not defined, this is why some provincies have number of jobs separated from the main cities.


```python
custom_color_scale = colors.make_colorscale(['blue', 'blue'])
fig = px.choropleth_mapbox(data_frame=df_provinces,
                           geojson=geojson_data,
                           locations='Province',  # Column identifying the province for each city
                           featureidkey='properties.name',  # Key in the GeoJSON file that identifies the province name
                           color_continuous_scale=custom_color_scale,  # Set a single color for all provinces
                           mapbox_style='carto-positron',
                           zoom=6,  # Adjust the zoom level accordingly
                           opacity=0.5,
                           labels={'SomeValueColumn': 'Value Label'},
                           hover_data={'Province': True, 'Abbreviation': True},  # Include additional hover data
                           title='Top Canadian cities with Data Jobs',
                           center={'lat': 65.9241, 'lon': -95.4613}  # Set the center coordinates for Canada

                           )

fig.update_layout(
    mapbox={'style': 'carto-positron', 'zoom': 1.8},
    margin={'r': 0, 't': 40, 'l': 0, 'b': 0}
)

# Add bubble markers for each city
fig.add_trace(
    go.Scattermapbox(
        lat=df27['Latitude'],  # Latitude values for cities
        lon=df27['Longitude'],  # Longitude values for cities
        mode='markers+text',
        marker=go.scattermapbox.Marker(
            size=df27['Count']/5,  # Adjust the size of the bubbles
            color=df27['Count'],  # Choose an appropriate color for the bubbles
            sizeref = 1,
            opacity=0.7,
            colorbar=dict(title='Count'),  # Add a colorbar with a suitable title
            colorscale='Bluered'  # Specify the desired colorscale

        ),
        text=df27['City'] + '<br>Count: ' + df27['Count'].astype(str),  # City name and count for hover info
        hoverinfo='text',
        textposition='bottom center'  # Position the city names at the bottom of the markers
    )
)


fig.show()
fig.write_html("plot4.html")
```
[plot4](plot4.png)
The answer to guiding question we can find in Geographical analysts and find that the most Data Science Jobs city in Canada is Toronto, the second one - Montreal, the third one is Vancouver and Calgary is placed in fourth place. So, as a Calgary resident I can say that Calgary is the first Data Science city in Alberta and one of the top Data Science cities in Canada.

Analysing the structure of these jobs using the following bar chart we can find some more insights, such as the second popular location for Data Science jobs is Remote. 


```python
# Creating a bar chart using Plotly Express
fig = px.bar(df21, x="Count", y="City", color='Job Class', orientation='h', height=600, 
             title='Data Jobs in Canadian Cities in June 2023')

# Updating the layout and customization
fig.update_layout(
    barmode='stack',
    yaxis={'categoryorder':'total ascending'},
    xaxis_title='Number of Jobs'
)

# Displaying the figure
fig.show()

# Saving the figure as an HTML file
fig.write_html("plot5.html")
```
[plot5](plot5.png)
Hiring remote specialists in Canada is highly popular, especially for the U.S. companies. So, if you are looking for a remote job from home, Data Science is the field where you can find a lot of opportunities for this. As Calgarian Data Scientist I'm very interesting about Calgary and I found then unlike other locations Calgarian companies more likely hire Data Scientists but not other related specialists such as Data Engineers or Data Analysts. This fact gives us some insights about the structure of the Calgarian job market which is filled with a lot of young and fast growing startups.

The structure of the job market would be incomplete without understanding the number of companies hiring Data Science specialists and without understanding how many job postings are posted in each part of the job market structure. 


```python

df2.loc[(df2['Company Size'].str.contains('1-10 employees')), 'Company Size'] = '1-10 employees'
df2.loc[(df2['Company Size'].str.contains('11-50 employees')), 'Company Size'] = '11-50 employees'
df2.loc[(df2['Company Size'].str.contains('51-200 employees')), 'Company Size'] = '51-200 employees'
df2.loc[(df2['Company Size'].str.contains('201-500 employees')), 'Company Size'] = '201-500 employees'
df2.loc[(df2['Company Size'].str.contains('501-1,000 employees')), 'Company Size'] = '501-1,000 employees'
df2.loc[(df2['Company Size'].str.contains('1,001-5,000 employees')), 'Company Size'] = '1,001-5,000 employees'
df2.loc[(df2['Company Size'].str.contains('5,001-10,000 employees')), 'Company Size'] = '5,001-10,000 employees'
df2.loc[(df2['Company Size'].str.contains('10,001+ employees')), 'Company Size'] = '10,001+ employees'
company_sizes = df2['Company Size'].value_counts()[:8].index
df4 = df2[df2['Company Size'].isin(company_sizes)]
df22 = df4[['Company Size', 'Job Class']].value_counts().reset_index()
df22 = df22.rename(columns = {0: 'Count'})
#df22
```


```python
# Creating a bar chart using Plotly Express
fig = px.bar(df22, y="Count", x="Company Size", color='Job Class', orientation='v',
             height=600, 
             title='Data Jobs in Canadian Companies')

# Updating the layout and customization
fig.update_layout(barmode='stack', yaxis_title = 'Number of jobs', xaxis={'categoryorder':'array', 'categoryarray':['1-10 employees',
                                                                                    '11-50 employees',
                                                                                    '51-200 employees',
                                                                                    '201-500 employees',
                                                                                    '501-1,000 employees',
                                                                                    '1,001-5,000 employees',
                                                                                    '5,001-10,000 employees', 
                                                                                     '10,001+ employees ']})
# Displaying the figure
fig.show()

# Saving the figure as an HTML file
fig.write_html("plot6.html")
```
[plot6](plot6.png)
So, analysing these two questions I found that most of the jobs came from middle-large and extra-large companies which may be as long as large Canadian companies as international companies hiring through Canada. And the other part is the local part of companies of different sizes and mostly hired locally in cities they are located.


```python
levels = df2['Job level'].value_counts()
df23 = df4[['Company Size', 'Job level']].value_counts().reset_index()
df23 = df23.rename(columns = {0: 'Count'})
#df23
#levels
```


```python

fig = px.bar(df23, y="Count", x="Company Size", color='Job level', orientation='v',
             #hover_data=["tip", "size"],
             height=600, 
             title='Data Jobs in Canadian Companies')
fig.update_layout(barmode='stack', yaxis_title = 'Number of jobs', xaxis={'categoryorder':'array', 'categoryarray':['1-10 employees',
                                                                                    '11-50 employees',
                                                                                    '51-200 employees',
                                                                                    '201-500 employees',
                                                                                    '501-1,000 employees',
                                                                                    '1,001-5,000 employees',
                                                                                    '5,001-10,000 employees', 
                                                                                     '10,001+ employees ']})
fig.show()
fig.write_html("plot6_.html")
```
[plot6_](plot6_.png)

```python
df5 = df4[['Company Name', 'Company Size']]
len(df5['Company Name'].unique())
companies = [len(df5.loc[df5['Company Size'] == size, 'Company Name'].unique()) for size in df5['Company Size'].unique()]
```


```python
df_companies = pd.DataFrame({'Size': df5['Company Size'].unique(), 'Number of Companies': companies})
```


```python
fig = px.bar(df_companies, y='Number of Companies', x='Size', orientation='v',
             #hover_data=["tip", "size"],
             height=600, 
             title='Number of Canadian Companies hiring for Data jobs')
fig.update_layout(barmode='stack', yaxis_title = 'Number of companies', xaxis={'categoryorder':'array', 'categoryarray':['1-10 employees',
                                                                                    '11-50 employees',
                                                                                    '51-200 employees',
                                                                                    '201-500 employees',
                                                                                    '501-1,000 employees',
                                                                                    '1,001-5,000 employees',
                                                                                    '5,001-10,000 employees', 
                                                                                     '10,001+ employees ']})
fig.show()
fig.write_html("plot6_1.html")
```
[plot6_1](plot6_1.png)

## 3. Which skills set is important for each job? How to make your LinkedIn profile matching with job postings? What should you know from related fields?


Understanding the jobs required by companies we should also understand what skill set an ideal candidate should have for each job position. Of course each position is different from each other, but knowing what is most required, improving their skills in each of this skill and sharing their experience in their LinkedIn page candidates could improve their chances to match with job posting in the field they focused on. This is why I am providing 8 simple bar charts containing the top 20 skills for each Data Science job based on job postings skill sets. 


```python
# Data Scientists
# Creating a filtered DataFrame for Data Scientists
df_ds = df2[df2['Job Class'] == ds_job_types[0]]

# Initializing empty lists and dictionary
skilllist = []
skilldict = {}

# Iterating through each skillset in the 'Skillset' column of df_ds DataFrame
for sklst in df_ds['Skillset']:
    # Splitting the skillset into a list of individual skills
    sklst_list = sklst.split(';')
    
    # Iterating through each skill in the skillset list
    for skill in sklst_list:
        # Removing leading whitespace if present
        if skill[0] == ' ':
            skill = skill[1:]
        
        # Checking if the skill is not empty and not already in the skilllist
        if skill != '' and skill not in skilllist:
            # Adding the skill to the skilllist and initializing its count in the skilldict
            skilllist.append(skill)
            skilldict[skill] = 1
        # If the skill is not empty and already in the skilllist, increment its count in the skilldict
        elif skill != '':
            skilldict[skill] += 1

# Sorting the skilldict based on the count of each skill in descending order
skilldict = dict(sorted(skilldict.items(), key=operator.itemgetter(1), reverse=True))

# Printing or further processing the skilldict
# print(skilldict)
```


```python
df_skills_ds = pd.DataFrame({'Skill': skilldict.keys(), 'Count': skilldict.values()})
#df_skills_ds[:20]
```


```python
# Creating a filtered DataFrame for Data Analysts
df_da = df2[df2['Job Class'] == ds_job_types[1]]

# Initializing empty lists and dictionary
skilllist = []
skilldict = {}

# Iterating through each skillset in the 'Skillset' column of df_da DataFrame
for sklst in df_da['Skillset']:
    # Splitting the skillset into a list of individual skills
    sklst_list = sklst.split(';')
    
    # Iterating through each skill in the skillset list
    for skill in sklst_list:
        # Removing leading whitespace if present
        if skill[0] == ' ':
            skill = skill[1:]
        
        # Checking if the skill is not empty and not already in the skilllist
        if skill != '' and skill not in skilllist:
            # Adding the skill to the skilllist and initializing its count in the skilldict
            skilllist.append(skill)
            skilldict[skill] = 1
        # If the skill is not empty and already in the skilllist, increment its count in the skilldict
        elif skill != '':
            skilldict[skill] += 1

# Sorting the skilldict based on the count of each skill in descending order
skilldict = dict(sorted(skilldict.items(), key=operator.itemgetter(1), reverse=True))

# Printing or further processing the skilldict
# print(skilldict)

```


```python
df_skills_da = pd.DataFrame({'Skill': skilldict.keys(), 'Count': skilldict.values()})
#df_skills_da[:20]
```


```python
# Creating a filtered DataFrame for Data Engineers
df_de = df2[df2['Job Class'] == ds_job_types[2]]

# Initializing empty lists and dictionary
skilllist = []
skilldict = {}

# Iterating through each skillset in the 'Skillset' column of df_de DataFrame
for sklst in df_de['Skillset']:
    # Splitting the skillset into a list of individual skills
    sklst_list = sklst.split(';')
    
    # Iterating through each skill in the skillset list
    for skill in sklst_list:
        # Removing leading whitespace if present
        if skill[0] == ' ':
            skill = skill[1:]
        
        # Checking if the skill is not empty and not already in the skilllist
        if skill != '' and skill not in skilllist:
            # Adding the skill to the skilllist and initializing its count in the skilldict
            skilllist.append(skill)
            skilldict[skill] = 1
        # If the skill is not empty and already in the skilllist, increment its count in the skilldict
        elif skill != '':
            skilldict[skill] += 1

# Sorting the skilldict based on the count of each skill in descending order
skilldict = dict(sorted(skilldict.items(), key=operator.itemgetter(1), reverse=True))

# Printing or further processing the skilldict
# print(skilldict)
```


```python
df_skills_de = pd.DataFrame({'Skill': skilldict.keys(), 'Count': skilldict.values()})
#df_skills_de[:20]
```


```python
# Creating a filtered DataFrame for Machine Learning Engineers
df_ml = df2[df2['Job Class'] == ds_job_types[3]]

# Initializing empty lists and dictionary
skilllist = []
skilldict = {}

# Iterating through each skillset in the 'Skillset' column of df_ml DataFrame
for sklst in df_ml['Skillset']:
    # Splitting the skillset into a list of individual skills
    sklst_list = sklst.split(';')
    
    # Iterating through each skill in the skillset list
    for skill in sklst_list:
        # Removing leading whitespace if present
        if skill[0] == ' ':
            skill = skill[1:]
        
        # Checking if the skill is not empty and not already in the skilllist
        if skill != '' and skill not in skilllist:
            # Adding the skill to the skilllist and initializing its count in the skilldict
            skilllist.append(skill)
            skilldict[skill] = 1
        # If the skill is not empty and already in the skilllist, increment its count in the skilldict
        elif skill != '':
            skilldict[skill] += 1

# Sorting the skilldict based on the count of each skill in descending order
skilldict = dict(sorted(skilldict.items(), key=operator.itemgetter(1), reverse=True))

# Printing or further processing the skilldict
# print(skilldict)

```


```python
df_skills_ml = pd.DataFrame({'Skill': skilldict.keys(), 'Count': skilldict.values()})
#df_skills_ml[:20]
```


```python
# Creating a filtered DataFrame for Product Managers
df_pm = df2[df2['Job Class'] == ds_job_types[4]]

# Initializing empty lists and dictionary
skilllist = []
skilldict = {}

# Iterating through each skillset in the 'Skillset' column of df_pm DataFrame
for sklst in df_pm['Skillset']:
    # Splitting the skillset into a list of individual skills
    sklst_list = sklst.split(';')
    
    # Iterating through each skill in the skillset list
    for skill in sklst_list:
        # Removing leading whitespace if present
        if skill[0] == ' ':
            skill = skill[1:]
        
        # Checking if the skill is not empty and not already in the skilllist
        if skill != '' and skill not in skilllist:
            # Adding the skill to the skilllist and initializing its count in the skilldict
            skilllist.append(skill)
            skilldict[skill] = 1
        # If the skill is not empty and already in the skilllist, increment its count in the skilldict
        elif skill != '':
            skilldict[skill] += 1

# Sorting the skilldict based on the count of each skill in descending order
skilldict = dict(sorted(skilldict.items(), key=operator.itemgetter(1), reverse=True))

# Printing or further processing the skilldict
# print(skilldict)
```


```python
df_skills_pm = pd.DataFrame({'Skill': skilldict.keys(), 'Count': skilldict.values()})
#df_skills_pm[:20]
```


```python
# Creating a filtered DataFrame for Software Engineers
df_se = df2[df2['Job Class'] == ds_job_types[10]]

# Initializing empty lists and dictionary
skilllist = []
skilldict = {}

# Iterating through each skillset in the 'Skillset' column of df_se DataFrame
for sklst in df_se['Skillset']:
    # Splitting the skillset into a list of individual skills
    sklst_list = sklst.split(';')
    
    # Iterating through each skill in the skillset list
    for skill in sklst_list:
        # Removing leading whitespace if present
        if skill[0] == ' ':
            skill = skill[1:]
        
        # Checking if the skill is not empty and not already in the skilllist
        if skill != '' and skill not in skilllist:
            # Adding the skill to the skilllist and initializing its count in the skilldict
            skilllist.append(skill)
            skilldict[skill] = 1
        # If the skill is not empty and already in the skilllist, increment its count in the skilldict
        elif skill != '':
            skilldict[skill] += 1

# Sorting the skilldict based on the count of each skill in descending order
skilldict = dict(sorted(skilldict.items(), key=operator.itemgetter(1), reverse=True))

# Printing or further processing the skilldict
# print(skilldict)

```


```python
df_skills_se = pd.DataFrame({'Skill': skilldict.keys(), 'Count': skilldict.values()})
#df_skills_se[:20]
```


```python
# Creating a filtered DataFrame for Business Intelligence professionals
df_bi = df2[df2['Job Class'] == ds_job_types[8]]

# Initializing empty lists and dictionary
skilllist = []
skilldict = {}

# Iterating through each skillset in the 'Skillset' column of df_bi DataFrame
for sklst in df_bi['Skillset']:
    # Splitting the skillset into a list of individual skills
    sklst_list = sklst.split(';')
    
    # Iterating through each skill in the skillset list
    for skill in sklst_list:
        # Removing leading whitespace if present
        if skill[0] == ' ':
            skill = skill[1:]
        
        # Checking if the skill is not empty and not already in the skilllist
        if skill != '' and skill not in skilllist:
            # Adding the skill to the skilllist and initializing its count in the skilldict
            skilllist.append(skill)
            skilldict[skill] = 1
        # If the skill is not empty and already in the skilllist, increment its count in the skilldict
        elif skill != '':
            skilldict[skill] += 1

# Sorting the skilldict based on the count of each skill in descending order
skilldict = dict(sorted(skilldict.items(), key=operator.itemgetter(1), reverse=True))

# Printing or further processing the skilldict
# print(skilldict)
```


```python
df_skills_bi = pd.DataFrame({'Skill': skilldict.keys(), 'Count': skilldict.values()})
#df_skills_bi[:20]
```


```python
# Creating a filtered DataFrame for Data Entry professionals
df_entry = df2[df2['Job Class'] == 'Data Entry']

# Initializing empty lists and dictionary
skilllist = []
skilldict = {}

# Iterating through each skillset in the 'Skillset' column of df_entry DataFrame
for sklst in df_entry['Skillset']:
    # Splitting the skillset into a list of individual skills
    sklst_list = sklst.split(';')
    
    # Iterating through each skill in the skillset list
    for skill in sklst_list:
        # Removing leading whitespace if present
        if skill[0] == ' ':
            skill = skill[1:]
        
        # Checking if the skill is not empty and not already in the skilllist
        if skill != '' and skill not in skilllist:
            # Adding the skill to the skilllist and initializing its count in the skilldict
            skilllist.append(skill)
            skilldict[skill] = 1
        # If the skill is not empty and already in the skilllist, increment its count in the skilldict
        elif skill != '':
            skilldict[skill] += 1

# Sorting the skilldict based on the count of each skill in descending order
skilldict = dict(sorted(skilldict.items(), key=operator.itemgetter(1), reverse=True))

# Printing or further processing the skilldict
# print(skilldict)

```


```python
df_skills_entry = pd.DataFrame({'Skill': skilldict.keys(), 'Count': skilldict.values()})
#df_skills_entry[:20]
```

And analysing this we can find that most important skills for Data Scientists in 2023 are Data Science (which is obvious), Data Visualization, Data Analytics, Natural Language Processing, Predictive Analytics, Statistics, Communications and so on. The entire skill set is placed on the bar chart below.


```python
fig = px.bar(df_skills_ds[:20], x="Count", y="Skill", orientation='h',
             #hover_data=["tip", "size"],
             height=600, 
             title='Top 20 skills for Data Scientist in 2023 based on LinkedIn job postings')
fig.update_layout(yaxis={'categoryorder':'total ascending'}, xaxis_title = 'Number of mentions')
fig.show()
fig.write_html("plot7.html")
```
[plot7](plot7.png)
For Data Analysts the demanded skill set starts from Data Analytics, Communications, Analytical Skills, Data Analysis, Analytics, Visualization and Problem Solving. The same as the previous, the entire skill set is placed on the chart below.


```python
fig = px.bar(df_skills_da[:20], x="Count", y="Skill", orientation='h',
             #hover_data=["tip", "size"],
             height=600, 
             title='Top 20 skills for Data Analyst in 2023 based on LinkedIn job postings')
fig.update_layout(yaxis={'categoryorder':'total ascending'}, xaxis_title = 'Number of mentions')
fig.show()
fig.write_html("plot8.html")
```
[plot8](plot8.png)
Machine Learning Engineers to be matching with the most MLE job postings should know Computer Science, Data Science, Artificial Intelligence, Machine Learning, Pattern Recognition, Natural Language Processing, Software Engineering, Programming, Python, Data Mining, Deep Learning etc.


```python
fig = px.bar(df_skills_ml[:20], x="Count", y="Skill", orientation='h',
             #hover_data=["tip", "size"],
             height=600, 
             title='Top 20 skills for Machine Learning Engineer in 2023 based on LinkedIn job postings')
fig.update_layout(yaxis={'categoryorder':'total ascending'}, xaxis_title = 'Number of mentions')
fig.show()
fig.write_html("plot9.html")
```
[plot9](plot9.png)

The top 5 Data Engineer Skills: ETL (Extract, Transform and Load), Data Engineering, Databases, Communication and Data Modeling. Following skills are placed on the bar chart below.


```python
fig = px.bar(df_skills_de[:20], x="Count", y="Skill", orientation='h',
             #hover_data=["tip", "size"],
             height=600, 
             title='Top 20 skills for Data Engineer in 2023 based on LinkedIn job postings')
fig.update_layout(yaxis={'categoryorder':'total ascending'}, xaxis_title = 'Number of mentions')
fig.show()
fig.write_html("plot10.html")
```
[plot10](plot10.png)
Software Engineers looking for a career in Data Science field should know Software Engineering (obviously), Communication, Computer Science, Databases, Back-End Web Development, Programming, SQL and other skills below.


```python
fig = px.bar(df_skills_se[:20], x="Count", y="Skill", orientation='h',
             #hover_data=["tip", "size"],
             height=600, 
             title='Top 20 skills for Software Engineer working developing Data applications in 2023 based on LinkedIn')
fig.update_layout(yaxis={'categoryorder':'total ascending'}, xaxis_title = 'Number of mentions')
fig.show()
fig.write_html("plot11.html")
```
[plot11](plot11.png)
Business Intelligence specialists should be experienced in Communication, BI, Data Analytics, Databases, Analytical Skills, creating Dashboards, operating with Data Warehouses, ETL, Problem Solving and Data Modeling and other skills below.


```python
fig = px.bar(df_skills_bi[:20], x="Count", y="Skill", orientation='h',
             #hover_data=["tip", "size"],
             height=600, 
             title='Top 20 skills for Business Intelligence in 2023 based on LinkedIn job postings')
fig.update_layout(yaxis={'categoryorder':'total ascending'}, xaxis_title = 'Number of mentions')
fig.show()
fig.write_html("plot12.html")
```
[plot12](plot12.png)
Product Managers also should be able to Communicate efficiently, to be familiar with Data Analytics and Data Analysis, Problem Solving, Query Writing, Project Management (obviously), MS Power Query and some more skills below.


```python
fig = px.bar(df_skills_pm[:20], x="Count", y="Skill", orientation='h',
             #hover_data=["tip", "size"],
             height=600, 
             title='Top 20 skills for Product Manager in 2023 based on LinkedIn job postings')
fig.update_layout(yaxis={'categoryorder':'total ascending'}, xaxis_title = 'Number of mentions')
fig.show()
fig.write_html("plot13.html")
```
[plot13](plot13.png)
Data Entry specialists should be qualified in English, Online Search, Global Business, Data Science, Data Mining and other skill sets below.


```python
fig = px.bar(df_skills_entry[:20], x="Count", y="Skill", orientation='h',
             #hover_data=["tip", "size"],
             height=600, 
             title='Top 20 skills for Data Entry in 2023 based on LinkedIn job postings')
fig.update_layout(yaxis={'categoryorder':'total ascending'}, xaxis_title = 'Number of mentions')
fig.show()
fig.write_html("plot14.html")
```
[plot14](plot14.png)
As a careful reader could find, many skills are placed in different Data Science jobs and may sign that any Data specialist should be familiar with skills from related job. This insight leads us to construct the model of competencies of each job position. To construct the model I calculated the number of includings of each skill in each job. Then, the number of intersections with each other category of job gives us the value of each ray. But to be more accurate, each value on each ray is divided to the number of jobs in each class. This gives us the relative value of the intersections of each position with each other and allows you to build an approximate wheel of competencies for each profession.


```python
# 4. Job profiles
l = len(df2.loc[df2['Job Class']=='Data Scientist', 'Job Class'])
```




    448




```python
l = len(df2.loc[df2['Job Class']=='Data Scientist', 'Job Class'])
ds = df_skills_ds['Count'].sum()/l
da = df_skills_ds.loc[df_skills_ds['Skill'].isin(df_skills_da['Skill']), 'Count'].sum()/l
de = df_skills_ds.loc[df_skills_ds['Skill'].isin(df_skills_de['Skill']), 'Count'].sum()/l
se = df_skills_ds.loc[df_skills_ds['Skill'].isin(df_skills_se['Skill']), 'Count'].sum()/l
bi = df_skills_ds.loc[df_skills_ds['Skill'].isin(df_skills_bi['Skill']), 'Count'].sum()/l
ml = df_skills_ds.loc[df_skills_ds['Skill'].isin(df_skills_ml['Skill']), 'Count'].sum()/l
pm = df_skills_ds.loc[df_skills_ds['Skill'].isin(df_skills_pm['Skill']), 'Count'].sum()/l
entry = df_skills_ds.loc[df_skills_ds['Skill'].isin(df_skills_entry['Skill']), 'Count'].sum()/l
values = [ds, da, de, se, bi, ml, pm, entry]
keys = ['Data Scientist', 'Data Analyst', 'Data Engineer', 'Software Engineer', 
        'Business Intelligence', 'Machine Learning Engineer', 'Product Manager', 'Data Entry']
values
df_profile_ds = pd.DataFrame({'keys': keys, 'Data Scientist': values})
df_profiles = df_profile_ds
```


```python
l = len(df2.loc[df2['Job Class']=='Data Engineer', 'Job Class'])
ds = df_skills_de.loc[df_skills_de['Skill'].isin(df_skills_ds['Skill']), 'Count'].sum()/l
da = df_skills_de.loc[df_skills_de['Skill'].isin(df_skills_da['Skill']), 'Count'].sum()/l
de = df_skills_de.loc[df_skills_de['Skill'].isin(df_skills_de['Skill']), 'Count'].sum()/l
se = df_skills_de.loc[df_skills_de['Skill'].isin(df_skills_se['Skill']), 'Count'].sum()/l
bi = df_skills_de.loc[df_skills_de['Skill'].isin(df_skills_bi['Skill']), 'Count'].sum()/l
ml = df_skills_de.loc[df_skills_de['Skill'].isin(df_skills_ml['Skill']), 'Count'].sum()/l
pm = df_skills_de.loc[df_skills_de['Skill'].isin(df_skills_pm['Skill']), 'Count'].sum()/l
entry = df_skills_de.loc[df_skills_de['Skill'].isin(df_skills_entry['Skill']), 'Count'].sum()/l
values = [ds, da, de, se, bi, ml, pm, entry]
df_profile_de = pd.DataFrame({'keys': keys, 'Data Engineer': values})
df_profiles = pd.merge(df_profiles, df_profile_de, on='keys')
#df_profiles
```


```python
l = len(df2.loc[df2['Job Class']=='Data Analyst', 'Job Class'])
ds = df_skills_da.loc[df_skills_da['Skill'].isin(df_skills_ds['Skill']), 'Count'].sum()/l
da = df_skills_da.loc[df_skills_da['Skill'].isin(df_skills_da['Skill']), 'Count'].sum()/l
de = df_skills_da.loc[df_skills_da['Skill'].isin(df_skills_de['Skill']), 'Count'].sum()/l
se = df_skills_da.loc[df_skills_da['Skill'].isin(df_skills_se['Skill']), 'Count'].sum()/l
bi = df_skills_da.loc[df_skills_da['Skill'].isin(df_skills_bi['Skill']), 'Count'].sum()/l
ml = df_skills_da.loc[df_skills_da['Skill'].isin(df_skills_ml['Skill']), 'Count'].sum()/l
pm = df_skills_da.loc[df_skills_da['Skill'].isin(df_skills_pm['Skill']), 'Count'].sum()/l
entry = df_skills_da.loc[df_skills_da['Skill'].isin(df_skills_entry['Skill']), 'Count'].sum()/l
values = [ds, da, de, se, bi, ml, pm, entry]
df_profile_de = pd.DataFrame({'keys': keys, 'Data Analyst': values})
df_profiles = pd.merge(df_profiles, df_profile_de, on='keys')
#df_profiles
```


```python
l = len(df2.loc[df2['Job Class']=='Machine Learning Engineer', 'Job Class'])
ds = df_skills_ml.loc[df_skills_ml['Skill'].isin(df_skills_ds['Skill']), 'Count'].sum()/l
da = df_skills_ml.loc[df_skills_ml['Skill'].isin(df_skills_da['Skill']), 'Count'].sum()/l
de = df_skills_ml.loc[df_skills_ml['Skill'].isin(df_skills_de['Skill']), 'Count'].sum()/l
se = df_skills_ml.loc[df_skills_ml['Skill'].isin(df_skills_se['Skill']), 'Count'].sum()/l
bi = df_skills_ml.loc[df_skills_ml['Skill'].isin(df_skills_bi['Skill']), 'Count'].sum()/l
ml = df_skills_ml.loc[df_skills_ml['Skill'].isin(df_skills_ml['Skill']), 'Count'].sum()/l
pm = df_skills_ml.loc[df_skills_ml['Skill'].isin(df_skills_pm['Skill']), 'Count'].sum()/l
entry = df_skills_ml.loc[df_skills_ml['Skill'].isin(df_skills_entry['Skill']), 'Count'].sum()/l
values = [ds, da, de, se, bi, ml, pm, entry]
df_profile_de = pd.DataFrame({'keys': keys, 'Machine Learning Engineer': values})
df_profiles = pd.merge(df_profiles, df_profile_de, on='keys')
#df_profiles
```


```python
l = len(df2.loc[df2['Job Class']=='Software Engineer', 'Job Class'])
ds = df_skills_se.loc[df_skills_se['Skill'].isin(df_skills_ds['Skill']), 'Count'].sum()/l
da = df_skills_se.loc[df_skills_se['Skill'].isin(df_skills_da['Skill']), 'Count'].sum()/l
de = df_skills_se.loc[df_skills_se['Skill'].isin(df_skills_de['Skill']), 'Count'].sum()/l
se = df_skills_se.loc[df_skills_se['Skill'].isin(df_skills_se['Skill']), 'Count'].sum()/l
bi = df_skills_se.loc[df_skills_se['Skill'].isin(df_skills_bi['Skill']), 'Count'].sum()/l
ml = df_skills_se.loc[df_skills_se['Skill'].isin(df_skills_ml['Skill']), 'Count'].sum()/l
pm = df_skills_se.loc[df_skills_se['Skill'].isin(df_skills_pm['Skill']), 'Count'].sum()/l
entry = df_skills_se.loc[df_skills_se['Skill'].isin(df_skills_entry['Skill']), 'Count'].sum()/l
values = [ds, da, de, se, bi, ml, pm, entry]
df_profile_de = pd.DataFrame({'keys': keys, 'Software Engineer': values})
df_profiles = pd.merge(df_profiles, df_profile_de, on='keys')
#df_profiles
```


```python
l = len(df2.loc[df2['Job Class']=='Business Intelligence', 'Job Class'])
ds = df_skills_bi.loc[df_skills_bi['Skill'].isin(df_skills_ds['Skill']), 'Count'].sum()/l
da = df_skills_bi.loc[df_skills_bi['Skill'].isin(df_skills_da['Skill']), 'Count'].sum()/l
de = df_skills_bi.loc[df_skills_bi['Skill'].isin(df_skills_de['Skill']), 'Count'].sum()/l
se = df_skills_bi.loc[df_skills_bi['Skill'].isin(df_skills_se['Skill']), 'Count'].sum()/l
bi = df_skills_bi.loc[df_skills_bi['Skill'].isin(df_skills_bi['Skill']), 'Count'].sum()/l
ml = df_skills_bi.loc[df_skills_bi['Skill'].isin(df_skills_ml['Skill']), 'Count'].sum()/l
pm = df_skills_bi.loc[df_skills_bi['Skill'].isin(df_skills_pm['Skill']), 'Count'].sum()/l
entry = df_skills_bi.loc[df_skills_bi['Skill'].isin(df_skills_entry['Skill']), 'Count'].sum()/l
values = [ds, da, de, se, bi, ml, pm, entry]
df_profile_de = pd.DataFrame({'keys': keys, 'Business Intelligence': values})
df_profiles = pd.merge(df_profiles, df_profile_de, on='keys')
#df_profiles
```


```python
l = len(df2.loc[df2['Job Class']=='Product Manager', 'Job Class'])
ds = df_skills_pm.loc[df_skills_pm['Skill'].isin(df_skills_ds['Skill']), 'Count'].sum()/l
da = df_skills_pm.loc[df_skills_pm['Skill'].isin(df_skills_da['Skill']), 'Count'].sum()/l
de = df_skills_pm.loc[df_skills_pm['Skill'].isin(df_skills_de['Skill']), 'Count'].sum()/l
se = df_skills_pm.loc[df_skills_pm['Skill'].isin(df_skills_se['Skill']), 'Count'].sum()/l
bi = df_skills_pm.loc[df_skills_pm['Skill'].isin(df_skills_bi['Skill']), 'Count'].sum()/l
ml = df_skills_pm.loc[df_skills_pm['Skill'].isin(df_skills_ml['Skill']), 'Count'].sum()/l
pm = df_skills_pm.loc[df_skills_pm['Skill'].isin(df_skills_pm['Skill']), 'Count'].sum()/l
entry = df_skills_pm.loc[df_skills_pm['Skill'].isin(df_skills_entry['Skill']), 'Count'].sum()/l
values = [ds, da, de, se, bi, ml, pm, entry]
df_profile_de = pd.DataFrame({'keys': keys, 'Product Manager': values})
df_profiles = pd.merge(df_profiles, df_profile_de, on='keys')
#df_profiles
```


```python
l = len(df2.loc[df2['Job Class']=='Data Entry', 'Job Class'])
ds = df_skills_entry.loc[df_skills_entry['Skill'].isin(df_skills_ds['Skill']), 'Count'].sum()/l
da = df_skills_entry.loc[df_skills_entry['Skill'].isin(df_skills_da['Skill']), 'Count'].sum()/l
de = df_skills_entry.loc[df_skills_entry['Skill'].isin(df_skills_de['Skill']), 'Count'].sum()/l
se = df_skills_entry.loc[df_skills_entry['Skill'].isin(df_skills_se['Skill']), 'Count'].sum()/l
bi = df_skills_entry.loc[df_skills_entry['Skill'].isin(df_skills_bi['Skill']), 'Count'].sum()/l
ml = df_skills_entry.loc[df_skills_entry['Skill'].isin(df_skills_ml['Skill']), 'Count'].sum()/l
pm = df_skills_entry.loc[df_skills_entry['Skill'].isin(df_skills_pm['Skill']), 'Count'].sum()/l
entry = df_skills_entry.loc[df_skills_entry['Skill'].isin(df_skills_entry['Skill']), 'Count'].sum()/l
values = [ds, da, de, se, bi, ml, pm, entry]
df_profile_de = pd.DataFrame({'keys': keys, 'Data Entry': values})
df_profiles = pd.merge(df_profiles, df_profile_de, on='keys')
#df_profiles
```


```python
df_profiles
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>keys</th>
      <th>Data Scientist</th>
      <th>Data Engineer</th>
      <th>Data Analyst</th>
      <th>Machine Learning Engineer</th>
      <th>Software Engineer</th>
      <th>Business Intelligence</th>
      <th>Product Manager</th>
      <th>Data Entry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Data Scientist</td>
      <td>9.794643</td>
      <td>6.105968</td>
      <td>6.206637</td>
      <td>7.083333</td>
      <td>5.658824</td>
      <td>5.254237</td>
      <td>5.171053</td>
      <td>6.546667</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Data Analyst</td>
      <td>6.973214</td>
      <td>7.263094</td>
      <td>9.775264</td>
      <td>5.822222</td>
      <td>6.325490</td>
      <td>7.254237</td>
      <td>6.802632</td>
      <td>7.680000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Data Engineer</td>
      <td>6.446429</td>
      <td>9.708892</td>
      <td>7.337858</td>
      <td>6.394444</td>
      <td>7.874510</td>
      <td>6.898305</td>
      <td>6.078947</td>
      <td>6.533333</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Software Engineer</td>
      <td>4.772321</td>
      <td>6.482339</td>
      <td>5.564103</td>
      <td>6.200000</td>
      <td>9.752941</td>
      <td>5.288136</td>
      <td>4.473684</td>
      <td>5.146667</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Business Intelligence</td>
      <td>3.796875</td>
      <td>4.729598</td>
      <td>5.422323</td>
      <td>2.861111</td>
      <td>3.800000</td>
      <td>9.711864</td>
      <td>4.171053</td>
      <td>3.933333</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Machine Learning Engineer</td>
      <td>5.863839</td>
      <td>5.019488</td>
      <td>4.932127</td>
      <td>9.683333</td>
      <td>5.462745</td>
      <td>4.135593</td>
      <td>3.828947</td>
      <td>5.226667</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Product Manager</td>
      <td>3.104911</td>
      <td>3.209501</td>
      <td>4.342383</td>
      <td>2.255556</td>
      <td>2.282353</td>
      <td>3.050847</td>
      <td>9.973684</td>
      <td>2.853333</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Data Entry</td>
      <td>2.361607</td>
      <td>2.115713</td>
      <td>3.150830</td>
      <td>2.511111</td>
      <td>1.517647</td>
      <td>2.084746</td>
      <td>2.000000</td>
      <td>9.906667</td>
    </tr>
  </tbody>
</table>
</div>



Resize dataframe to the format which we can use to plot line polar chart.


```python
df_line_polar = df_profiles[['keys','Data Scientist']]
df_line_polar = df_line_polar.rename(columns = {'Data Scientist': 'value'})
df_line_polar['Job Class'] = 'Data Scientist'
for column in df_profiles.columns[2:]:
    new_df = pd.DataFrame(df_profiles[['keys',column]]).rename(columns = {column: 'value'})
    new_df['Job Class'] = column
    df_line_polar = pd.concat([df_line_polar, 
                               new_df])

#df_line_polar #= df_line_polar.rename(columns = {'keys': 'Job Class', })
```


```python
fig = px.line_polar(df_line_polar[:32], r="value", theta="keys", color = 'Job Class', line_close=True)
fig.show()
fig.write_html("plot15.html")
```
[plot15](plot15.png)
[plot16](plot16.png)
As we can find on the chart below Data Scientists should also have skills of Data Analyst, Data Engineer and Machine Learning Engineer. As a Data Scientist other specialists on the charts below should be familiar with skills from different jobs. To take more information please play with charts below and fill free to give the feedback about this model.

# 4. What are the most popular programming languages and tools in Data Science based on Job Postings?

Most of the competencies are understandable for us now, but practically the question "What should I learn to be more competitive?" is still on the table. To answer this question I analysed all of the job descriptions to find specific names of programming languages, tools and libraries.


```python
popular_languages = [
    "Python",
    "R,", "R ",
    "SQL",
    "Java",
    "C++",
    "Julia",
    "SAS",
    "Matlab",
    "Scala",
    "JavaScript",
    "Go.","Go,",
    "Ruby",
    "Perl",
    "Haskell",
    "Shell",
    "TypeScript",
    "Kotlin",
    "PHP",
    "Swift",
    "C#",
    "Lua",
    "Groovy",
    "VB.NET",
    "Objective-C",
    "Rust",
    "Julia",
    "F#",
    "Dart",
    "Fortran",
    "Clojure"
]

```


```python
popular_tools = [
    "Jupyter Notebook",
    "Anaconda",
    "TensorFlow",
    "Keras",
    "PyTorch",
    "Tableau",
    "Power BI",
    "Apache Spark",
    "Hadoop",
    "Kafka",
    "Amazon Web Services", 'AWS',
    "Google Cloud",
    "Azure", "Airflow", 
    "Docker", "Grafana",
    "Git",
    "GitHub", "github"
    "D3.js",
    "Excel",
    "MATLAB", "Matlab",
    "SAS",
    "HBase",
    "Hive",
    "Apache Pig",
    "Databricks",
    "RapidMiner",
    "KNIME",
    "Alteryx"
]

```


```python
popular_libraries = [
    "NumPy",
    "Pandas",
    "scikit-learn",
    "Matplotlib",
    "Seaborn",
    "TensorFlow",
    "Keras",
    "PyTorch",
    "NLTK",
    "SciPy",
    "Statsmodels",
    "XGBoost",
    "LightGBM",
    "CatBoost",
    "OpenCV",
    "Dask",
    "H2O",
    "Gensim",
    "NLTK",
    "Spacy",
    "BeautifulSoup",
    "Plotly",
    "Bokeh",
    "NetworkX",
    "Dash,", "Dash.", "Dash ",
    "PySpark",
    "Flask",
    "Django",
    "dplyr",
    "ggplot2",
    "caret",
    "tidyr",
    "data.table",
    "lubridate",
    "rpart",
    "randomForest",
    "glmnet",
    "caret",
    "e1071",
    "xgboost",
    "tidyverse",
    "stringr",
    "readr",
    "shiny",
    "forecast,", "forecast."
    "lattice",
    "broom",
    "rvest",
    "reshape2",
    "Matrix",
    "survival",
    "sparklyr",
    "ggvis",
    "feather",
    "knitr"
]
```


```python
# Initialize an empty list to store the languages
df_langs = []

# Initialize an empty dictionary to store the language counts
dict_langs = dict()

# Iterate over each job description
for descr in df2['Description']:
    # Iterate over each popular language
    for lang in popular_languages:
        # Check if the language is present in the description
        if lang in descr:
            # Update the count for the language in the dictionary
            if lang not in dict_langs.keys():
                dict_langs[lang] = 1
            else:
                dict_langs[lang] += 1

# Consolidate counts for 'R' and 'Go' by summing counts for different variations
dict_langs['R'] = dict_langs.get('R,', 0) + dict_langs.get('R ', 0)
dict_langs['Go'] = dict_langs.get('Go,', 0) + dict_langs.get('Go.', 0)

# Remove redundant keys from the dictionary
dict_langs.pop('R,')
dict_langs.pop('R ')
dict_langs.pop('Go,')
dict_langs.pop('Go.')

# Sort the dictionary based on the count values in descending order
dict_langs = dict(sorted(dict_langs.items(), key=operator.itemgetter(1), reverse=True))
```


```python
df_langs = pd.DataFrame({'key': dict_langs.keys(), 'value': dict_langs.values()})
fig = px.treemap(df_langs, path=['key'], values='value',
                  color='value', hover_data=['key'])
fig.show()
#df_langs
fig.write_html("plot17.html")
```
[plot17](plot17.png)
And the answers for these questions are placed on three following tree plots. The top 3 most popular languages for Data Science jobs are SQL, Python and R. So, these languages are the kinds that MUST HAVE in the Data Science field and should be learned by anyone who wants to be competitive in the job market in this field. Java, SAS, Scala, C++ and other languages are also important for Data Science roles.


```python
# Initialize an empty list to store the tools
df_langs = []

# Initialize an empty dictionary to store the tool counts
dict_tools = dict()

# Iterate over each job description
for descr in df2['Description']:
    # Iterate over each popular tool
    for tool in popular_tools:
        # Check if the tool is present in the description
        if tool in descr:
            # Update the count for the tool in the dictionary
            if tool not in dict_tools.keys():
                dict_tools[tool] = 1
            else:
                dict_tools[tool] += 1

# Consolidate counts for 'AWS', 'MATLAB' by summing counts for different variations
dict_tools['AWS'] = dict_tools.get('AWS', 0) + dict_tools.get('Amazon Web Services', 0)
# dict_tools['GitHub'] = dict_tools.get('GitHub', 0) + dict_tools.get('github', 0)
dict_tools['MATLAB'] = dict_tools.get('MATLAB', 0) + dict_tools.get('Matlab', 0)

# Remove redundant keys from the dictionary
dict_tools.pop('Amazon Web Services')
# dict_tools.pop('github')
dict_tools.pop('Matlab')

# Sort the dictionary based on the count values in descending order
dict_tools = dict(sorted(dict_tools.items(), key=operator.itemgetter(1), reverse=True))
```


```python
df_tools = pd.DataFrame({'key': dict_tools.keys(), 'value': dict_tools.values()})
fig = px.treemap(df_tools, path=['key'], values='value',
                  color='value', hover_data=['key'])
fig.show()
fig.write_html("plot18.html")
```
[plot18](plot18.png)
When it comes to tools, Microsoft Excel is the MUST HAVE tool for anyone who wants to work in the Data field, as also Power BI, Tableau, AWS, Azure and other important tools related with Data Engineering, Machine Learning and Version control process


```python
# Initialize an empty list to store the libraries
df_libs = []

# Initialize an empty dictionary to store the library counts
dict_libs = dict()

# Iterate over each job description
for descr in df2['Description']:
    # Iterate over each popular library
    for lib in popular_libraries:
        # Check if the library (case-insensitive) is present in the description
        if lib.lower() in descr.lower():
            # Update the count for the library in the dictionary
            if lib not in dict_libs.keys():
                dict_libs[lib] = 1
            else:
                dict_libs[lib] += 1

# Consolidate counts for 'Dash' by summing counts for different variations
dict_libs['Dash'] = dict_libs.get('Dash', 0) + dict_libs.get('Dash,', 0) + dict_libs.get('Dash ', 0)

# Remove redundant keys from the dictionary
dict_libs.pop('Dash,')
dict_libs.pop('Dash ')

# Sort the dictionary based on the count values in descending order
dict_libs = dict(sorted(dict_libs.items(), key=operator.itemgetter(1), reverse=True))
```


```python
df_libs = pd.DataFrame({'key': dict_libs.keys(), 'value': dict_libs.values()})
fig = px.treemap(df_libs, path=['key'], values='value',
                  color='value', hover_data=['key'])
fig.show()
fig.write_html("plot19.html")
```
[plot19](plot19.png)
According to libraries from the tree chart below we can find that such Machine Learning libraries as TensorFlow, PyTorch, data manipulation library Pandas, NumPy, sk-learn and others are highly important for our field. So, if you are not familiar with these libraries and are going to enter the market, start learning today, because hiring companies expect it from us in 2023.

## Conclusion

In Conclusion, I want to say thanks to all developers of libraries which I used for this work. I also thank to LinkedIn for their great job board and hope they will not ban me for this project, because the main goal I pursue is to share the knowledge which I got from the data to help people navigate in the field and if it will help anyone, these people will come to LinkedIn and find another job via this great professional network.

I want to say special thanks to my lovely wife, her great patience, support and also the laptop which I used for this project as a second station.

And I also want to thank my colleagues, professors, assistants and students of the University of Calgary which is providing the great Master of Data Science and Analytics program.

Hope you enjoyed this reading.

#If you have any questions, comments or just want to connect with me, please feel free to contact me via LinkedIn.


```python
# Concluding just a simple approach to create a word cloud of programming languages
from wordcloud import WordCloud
import matplotlib.pyplot as plt

# Concatenate the languages based on their frequencies
languages = ''
for lang in dict_langs:
    languages += (lang + ' ') * dict_langs[lang]

# Set the figure size
plt.subplots(figsize=(16, 8))

# Generate the word cloud
wordcloud = WordCloud(
    background_color='white',
    width=1024,
    height=384,
    collocations=False
).generate(' '.join(languages.split(' ')))

# Display the word cloud
plt.imshow(wordcloud)
plt.axis('off')
plt.show()

#plt.savefig('Plotly-World_Cloud.png')
```


    
![png](output_138_0.png)
    


# References
1. [Selenium](https://www.selenium.dev/) - Open Source toolkit for automation of web browser.
2. [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) - Python package for parsing HTML and XML documents.
3. [LinkedIn Job Board](https://www.linkedin.com/jobs/) - The initial source of this research.
4. [Plotly](https://plotly.com/python/) - One of the best visualization libraries for Python.
5. [OpenAI API Toolkit](https://openai.com/blog/openai-api) - Strong instrument to use LLM GPT-3.5 in your application.
6. [DeepLearning.ai](https://deeplearning.ai) - One of the best resourses to learn about Machine Learning and GPT Prompt Engineering
