## by Ryan Jeon


## Project Workflow: 
1. Ask an interesting question: 
2. Obtain the data
3. Explore the data
4. Communicate and visualize the results
5. Future steps 

## 1. Ask as interesting question: 
"Is the Iowa State University ABE program teaching the right skills that are needed for ABE careers on Indeed.com?"



<p align="center">
    <img alt="3" src="https://user-images.githubusercontent.com/69263707/144760951-05c80c70-adfa-46b9-ba74-9524fa5a4746.png" width="500">

</p>



## 2. Obtain the data:
I want to find keywords that are used in the Indeed.com and ISU ABE course catalog: 
  - I will be using the Indeed.com homepage for my career keyword data.
  - I will also be using the ISU ABE course catalog for my course key word data. 






## 3. Exploring the data:
My plan is to use a variety of webscraping functions (beautiful soup, nltk, requests) to explore what kind of data I can obtain from the two websites. 
Then, using nltk, I will be able to use seaborn to plot the frequencies of each keyword 


Using right click, and then going to "inspect" will show you the "matrix" or the HTML version of the website. 
<img width="722" alt="Screen Shot 2021-12-05 at 1 37 35 PM" src="https://user-images.githubusercontent.com/69263707/144761012-bfc0a8c0-1112-4cdd-ade3-baa59987180f.png">

<img width="2272" alt="Screen Shot 2021-12-05 at 1 42 08 PM" src="https://user-images.githubusercontent.com/69263707/144761165-344b546c-c56a-41c0-bb27-56d716611799.png">


Clicking the very left button (the mouse) on the tool bar will give you an interactive look at how the website is coded.

<img width="691" alt="Screen Shot 2021-12-05 at 1 38 40 PM" src="https://user-images.githubusercontent.com/69263707/144761066-47fd2c78-d573-4e73-ab6b-1d7102cea859.png">


Hovering your mouse over a job title highlights the following line

<img width="799" alt="Screen Shot 2021-12-05 at 1 40 37 PM" src="https://user-images.githubusercontent.com/69263707/144761131-a4da2dcc-fc04-42a8-9b76-a7eafb5b6642.png">


## 3. User Defined Functions for Exploring data: 

### Auxilliary Function(s): 

```yml
# code needed to extract the name of the df

def get_df_name(df):
    name =[x for x in globals() if globals()[x] is df][0]
    return name

```


### Course Catalog Function:

```yml

from bs4 import BeautifulSoup
import requests
import pandas as pd

def course_cat(URL):
    df_courses = pd.DataFrame(columns = ["Descriptions"])
    
    headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.2 Safari/605.1.15'}
    r = requests.get(URL, headers = headers)
    soup = BeautifulSoup(r.text, 'html.parser')
    
    descriptions = soup.find_all(class_ = "prereq")
    for description in descriptions:
        description_list = description.text
        df_courses.loc[len(df_courses.index)] = [description_list]
        
    df_course_names = pd.DataFrame(columns = ["Names"])
    
    names = soup.find_all(class_ = "toggle-accordion")
    for name in names:
        coursename_list = name.text
        df_course_names.loc[len(df_course_names.index)] = [coursename_list]
    compiled = pd.concat([df_course_names, df_courses], axis=1, join='inner')
    
    return compiled

```
We are going to use the link to the ABE course catalog:

```
URL = 'https://catalog.iastate.edu/azcourses/abe/'
```

```
ABE = course_cat(URL)
ABE
```
<img width="718" alt="Screen Shot 2021-12-05 at 6 55 23 AM" src="https://user-images.githubusercontent.com/69263707/144747440-3407f455-b77a-4ac9-8eef-9f21f4ec0ab6.png">



### Indeed Job Postings Dataset Function:
```yml
def IndeedPostings(URL_from_indeed):
    df = pd.DataFrame(columns = ["Job_Titles"])
    df2 = pd.DataFrame(columns = ["Company"])
    df3 = pd.DataFrame(columns = ["Location"])
    df4 = pd.DataFrame(columns = ["Job_Description"])
    df5 = pd.DataFrame(columns = ["Salary (If Available)"])
    df6 = pd.DataFrame(columns = ["Link"])


    for pagenumber in range(0,500, 1):
        r = requests.get('https://www.indeed.com/jobs?q=agricultural%20engineering&start={}'.format(pagenumber), headers = headers)
        soup = BeautifulSoup(r.text, 'html.parser')
        titles = soup.select("h2 span") 
        # select all span tags under the umbrella of h2 tags 
        companies = soup.find_all(class_ = "companyName")
        locations = soup.find_all(class_ = "companyLocation")
        descriptions = soup.find_all(class_ = "job-snippet")
        salaries = soup.find_all(class_ = "salary-snippet")
        URLs = soup.find_all('a', attrs = {'class' : 'tapItem'})

        for title in titles:
            titles_list = title.text
            # the gotcha here is that there are "news" scattered so we have to remove those first before concatenating our data
            df.loc[len(df.index)] = [titles_list]
            df = df[df.Job_Titles != "new"]
        
        
        for company in companies:
            company_list = company.text
            df2.loc[len(df2.index)] = [company_list]       
        
        for location in locations:
            location_list = location.text
            df3.loc[len(df3.index)] = [location_list]
    
        for description in descriptions:
            description_list = description.text
            df4.loc[len(df4.index)] = [description_list]
            df4['Job_Description'] = df4['Job_Description'].str.replace(r'\n', '', regex=True)
        # Another gotcha is that you have to remove the /n in each row- but looping can take considerable more time if we used hundreds of pages
    
        for salary in salaries:
            salary_list = salary.text
            df5.loc[len(df5.index)] = [salary_list]
    
        for URL in URLs:
            base = 'www.indeed.com'
            link = URL.attrs['href']
            new_URL = base + link
            df6.loc[len(df6.index)] = [new_URL]
    indeed_table = pd.concat([df, df2, df3, df4, df5, df6], axis=1)
    return(indeed_table)

```
```
URL_indeed = 'https://www.indeed.com/jobs?q=agricultural%20engineer&start={pagenumber}'
indeed_table = IndeedPostings(URL_indeed)
indeed_table.head(10)
```

<img width="1090" alt="Screen Shot 2021-12-05 at 6 58 25 AM" src="https://user-images.githubusercontent.com/69263707/144747526-9b652985-d5b1-48fb-aff8-8897af53457b.png">

### Good practice to see what kind of size dataset we are working with 
<img width="1116" alt="Screen Shot 2021-12-05 at 6 58 36 AM" src="https://user-images.githubusercontent.com/69263707/144747528-ef5a9ea3-229f-4858-99a1-e0968a831980.png">


```
indeed_figure(indeed_table)
```


## 4. Communicate and visualize the results:


### Course Catalog Frequency Plot Function:
```yml
import seaborn as sns
import matplotlib.pyplot as plt
import nltk
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords
from nltk.tokenize import sent_tokenize

def course_cat_figure(compiled):
    final_list = []
    stop_words = stopwords.words('english')
    newStopWords = ['department','curriculum', 'within', 'permission', 'introduction', 'credits','216Engineering', 'equivalent8', 'week-course', 'course', 'offered', 'student', 'satisfactory-fail', 'prereq','credit', 'enrollment', '165Introduction']
    stop_words.extend(newStopWords)
    for row in compiled.iterrows():
        words = word_tokenize(row[1]['Descriptions'])
        for word in words:
            if word.lower() not in stop_words:
                if len(word) > 5:
                    final_list.append(word)
                
    text = nltk.Text(final_list)
    all_fdist = nltk.FreqDist(text).most_common(30)
    all_fdist = pd.Series(dict(all_fdist))
    fig, ax = plt.subplots(figsize=(20,10))

    name = get_df_name(compiled)
    bar_plot = sns.barplot(x=all_fdist.values, y=all_fdist.index, orient='h', ax=ax)
    plt.title('Frequencies of the Most Common Words in the ' + name + ' Course Catalogue \n (12/04/21) \n', fontsize = 24)
    plt.xlabel('Frequency', fontsize=18)
    plt.ylabel('', fontsize=18)
        
 ```
```
course_cat_figure(ABE)
```

![download-3](https://user-images.githubusercontent.com/69263707/144747456-5bb02fbd-85f4-4761-865c-1d871334c7fa.png)

### I was also curious to see what other "similar" majors might look like

<img width="1114" alt="Screen Shot 2021-12-05 at 6 56 51 AM" src="https://user-images.githubusercontent.com/69263707/144747492-e1f4ce73-d9d8-41d4-b23c-512192c2579c.png">
<img width="1087" alt="Screen Shot 2021-12-05 at 6 57 15 AM" src="https://user-images.githubusercontent.com/69263707/144747493-5e966f73-e1b8-45b3-9cd0-72359c96ad38.png">
<img width="1083" alt="Screen Shot 2021-12-05 at 6 57 06 AM" src="https://user-images.githubusercontent.com/69263707/144747494-d326c1c6-a36c-4215-8b54-7898e548f7b0.png">


#### Indeed Data Frequency Figure:
``` yml
def indeed_figure(indeed_table):
    final_list = []

    stop_words = stopwords.words('english')
    newStopWords = ['degree', 'experience', 'Experience', 'provide', 'and/or','including', 'related', 'located', 'Center', 'numerous','throughout', 'equivalent', 'week-course', 'course', 'offered', 'student', 'satisfactory-fail', 'prereq','credit', 'enrollment', '165Introduction']
    stop_words.extend(newStopWords)
    for row in indeed_table.iterrows():
        words = word_tokenize(row[1]['Job_Description'])
        for word in words:
            if word.lower() not in stop_words:
                if len(word) > 5:
                    final_list.append(word)
                
    text = nltk.Text(final_list)
    all_fdist = nltk.FreqDist(text).most_common(30)
    all_fdist = pd.Series(dict(all_fdist))
    fig, ax = plt.subplots(figsize=(20,10))

    bar_plot = sns.barplot(x=all_fdist.values, y=all_fdist.index, orient='h', ax=ax)
    plt.title('Frequencies of the Most Common Words in Indeed.com ABE Job Postings \n (12/04/21) \n', fontsize = 24)
    plt.xlabel('Frequency', fontsize=18)
    plt.ylabel('', fontsize=18)
 ```

![download-5](https://user-images.githubusercontent.com/69263707/144755928-a0d3303b-c0f3-4d5d-9a9c-e96c1b2e02f0.png)


## Now we can see what keywords were most commonly used in the ABE job postings on Indeed.com
### Lets look at the list of words again:

## final List of Words for Indeed Job Descriptions: 
```
final_list = []

stop_words = stopwords.words('english')
newStopWords = ['degree', 'experience', 'Experience', 'provide', 'and/or','including', 'related', 'located', 'Center', 'numerous','throughout', 'equivalent', 'week-course', 'course', 'offered', 'student', 'satisfactory-fail', 'prereq','credit', 'enrollment', '165Introduction']
stop_words.extend(newStopWords)
for row in indeed_table.iterrows():
    words = word_tokenize(row[1]['Job_Description'])
    for word in words:
        if word.lower() not in stop_words:
            if len(word) > 5:
                final_list.append(word)
text = nltk.Text(final_list)                
               
all_Indeed = nltk.FreqDist(text).most_common(30)
all_Indeed = pd.Series(dict(all_Indeed))
all_Indeed = pd.DataFrame({'Word':all_Indeed.index, 'Count':all_Indeed.values})

print(all_Indeed)
```



### final List of Words for ABE Course Catalogue Descriptions: 
```
final_list = []

stop_words = stopwords.words('english')
newStopWords = ['department','curriculum', 'within', 'permission', 'introduction', 'credits','216Engineering', 'equivalent8', 'week-course', 'course', 'offered', 'student', 'satisfactory-fail', 'prereq','credit', 'enrollment', '165Introduction']
stop_words.extend(newStopWords)
for row in ABE.iterrows():
    words = word_tokenize(row[1]['Descriptions'])
    for word in words:
        if word.lower() not in stop_words:
            if len(word) > 5:
                final_list.append(word)
text = nltk.Text(final_list)                
all_ABE = nltk.FreqDist(text).most_common(30)
all_ABE = pd.Series(dict(all_ABE))
all_ABE = pd.DataFrame({'Word':all_ABE.index, 'Count':all_ABE.values})
print(all_ABE)
```


<p align="center">
    <img alt="3" src="https://user-images.githubusercontent.com/69263707/144747581-d7050751-c1e0-458b-89a6-269062a81335.png" width="300">
    <img alt="3" src="https://user-images.githubusercontent.com/69263707/144747578-e3dda20c-63a7-4714-96e3-ba4365845655.png" width="400">                                                                                                                                        

</p>


### 4. Continued (Using Geopandas):
```
#indeed_table['Location']
indeed_table['Location_split'] = indeed_table['Location'].str.split('+').str[0]
indeed_table['Location_split2'] = indeed_table['Location_split'].str.replace('\d+', '')
indeed_table[['City', 'State']] = indeed_table['Location_split2'].str.split(',', expand=True)
indeed_table_final = indeed_table.drop(['Location_split', 'Location_split2', 'Location', 'test1', 'test2'], axis=1)
indeed_table_final['State'].str.split(' ', expand=True)
test = indeed_table_final['State'].str.split(' ', expand=True)
test2 = test[1].str.split('•', expand=True)
state_counts = test2[0].value_counts()
state_counts
```

<img width="282" alt="Screen Shot 2021-12-05 at 7 11 52 AM" src="https://user-images.githubusercontent.com/69263707/144747976-73d83673-f877-4ab3-966b-44d8a84bdb71.png">

```
# Import geopandas package
import geopandas as gpd
# Read in shapefile and examine data
usa = gpd.read_file('/Users/ryanjeon/Desktop/ABE516/cb_2018_us_state_20m/cb_2018_us_state_20m.shp')
states = usa.merge(state_count, left_on = 'STUSPS', right_on = 'State')
states = states.loc[~states['NAME'].isin(['Alaska', 'Hawaii'])]
fig, ax = plt.subplots(1, figsize=(15, 15))
plt.xticks(rotation=90)
pop_states.plot(column="Count", cmap="Reds", linewidth=0.4, ax=ax, edgecolor=".4")
bar_info = plt.cm.ScalarMappable(cmap="Reds", norm=plt.Normalize(vmin=0, vmax=120))
bar_info._A = []
cbar = fig.colorbar(bar_info)
```

<img width="842" alt="Screen Shot 2021-12-05 at 7 13 08 AM" src="https://user-images.githubusercontent.com/69263707/144748022-aed39b68-090b-4f1d-b957-aaa26f5bc9eb.png">

# Discussion:

Based on these qualitative results, I argue that ABE is doing a great job at teaching students the skills necessary for careers. 
The following key words between Indeed.com job postings and ABE course catalogues are shared in the top 30:
- engineering
- systems
- agricultural
- design
- research 
- management
- quality 
- techniques

I would say that the only key word that I found on Indeed.com that should be mentioned more in the ABE course catalogue would be:
- equipment

It makes sense, since a lot of engineers wear specialized equipment for their work. Perhaps we need to include this word in the catalogue or discuss it more in some of the courses. 

Using geopandas, I also found that it also looks like most of the jobs are centered around the midwest, particularly in Iowa. If that result was intended or erroneous can be up for discussion, but it seems like most of the job opportunities are located in Iowa. 

# Course Concepts:
- Webscraping
- Data summary using basic Pandas functions
- Plotting using Seaborn
- NLTK package

## 5. Class Task:
  - I would also like to use the course catalogue for other top ABE schools (Purdue and Cornell)
  - Plot geopandas by city, not by state (like a COVID cases map) 
