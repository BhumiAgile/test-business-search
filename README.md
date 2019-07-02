# Business Search

 **Data scrubbing** refers to the procedure of modifying or removing incomplete, incorrect, inaccurately formatted, or repeated data in a database or record set. The key objective of data scrubbing is to make the data more accurate and consistent. Data scrubbing is also referred to as data cleansing.

## Table of contents

* [Getting Started](#getting-started)
* [Pre Requirements](#pre-requirements)
* [Set API Credentials](#set-api-credentials)
* [Data Availability Check](#data-availability-check)
* [Flow of the Script](#flow-of-the-script)
* [Business Data Formatting Rules](#business-data-formatting-rules)
* [Manage Data](#manage-data)
* [Conclusion](#conclusion)

## Getting Started

Here, the user has to download the actual data source for the project of business search. This includes the main script to clean and format data and also contains API collection to find or get missing required business data(email, domain, phone number, address etc.) of the company from the web.

## Pre Requirements

It is required to use `Python 3.x`, `pip3` and `virtualenv`. The installtion steps are given below.

1. First create virtual environment in your working directory using the given command.

    ```cmd
    $ virtualenv myvenv -p python3
    ```
    Here, myvenv is the name of the virtual environment.

2. Next activate the virtual environment.

    > In Ubuntu
        ```cmd
        $ source myvenv/bin/activate
        ```

    > In Window
        ```cmd
        $ \myvenv\Scripts\activate
        ```

3. Now install all the required packages and dependancies from the requirements.txt file, which is given in the project directory. Use following command for it.

    ```cmd
    $ pip install -r requirements.txt
    ```


## Set API Credentials

Here, we will need to perform searching for the business data of the company like, company address, phone number, emails, domain, etc. For that it is required to use various APIs to get missing business data. So we need credentials for the perticular API. The APIs used in this project are,

* Google Place API
* Facebook Place SEarch API
* Yelp API
* Yellow Pages API
* BBB API
* Google Search API
* Melissa API
* Hunter API

All the credentials should be stored into the **config.ini** file, located into the project directory.

To get api_key or access_token for the perticular API, the user has to signup in the perticular account. From the account dashboard, user will be able to get the access_key or token or api_key. Please do not share these credentials with other.

### Details of the API account

>Google Place API

- Make a new project and get the API key. For the entire process, the user can follow the documentation from [here](https://developers.google.com/places/web-service/get-api-key).
- Copy that **API key** and paste it into the `config.ini` file.

    ```
    [GOOGLE]
    api_key = #########################
    ```

>Facebook Place Search API

- Create an app and get access token. The user can follow the documentation from [here](https://developers.facebook.com/docs/facebook-login/access-tokens/).

- Copy that **Access Token** and paste it into the `config.ini` file.

    ```
    [FACEBOOK]
    access_token = #######################
    ```

>Yelp API

- The user can get the API key from [here](https://www.yelp.com/login?return_url=%2Fdevelopers%2Fv3%2Fmanage_app). In the `config.ini` file save it as follows.

    ```
    [YELP]
    api_key = #############################
    ```

>Melissa API

- Find the license key from [here](https://www.melissa.com/user/user_account.aspx) and paste it into the `config.ini` file.
- Set default count with 1000 as Melissa provides free 1000 credits per account.

    ```
    [MELISSA]
    license_key = ##########################
    count = 1000
    ```

>Hunter API

- Find the license key from [here](https://hunter.io/api_keys) and paste it into the `config.ini` file.

    ```
    [HUNTER]
    api_key = ####################
    ```

Note: There is no need to set credentials for Yellow Pages API, BBB API and Google Search API.

## Data Availability Check

Here, the regex code is used to find out business data availability into our data source(available business data). Based on this data availability check, the script will start the process to get missing data using various APIs.

>**Email Search Regex**
    ^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$

>**Domain Search Regex**
    http[s]?://(?:[a-zA-Z]|[0-9]|[$-_@.&+]|[!*\(\),]|(?:%[0-9a-fA-F][0-9a-fA-F]))+

>**Phone Search Regex**
    ((\(\d{3}\) ?)|(\d{3}-))?\d{3}-\d{4}

## Flow of the Script

* The process of collecting lead data includes various API. In our script, we use API to find missing business data (Domain, Name, Email, Phone, Address) in particular order.

* The API will be called if it provides data actually we required otherwise it will be skipped.

* To decide which API should be called, the script maintains a data dictionary for the data availability and also maintains the data dictionary for API response data.

* If particular data are missing for the company and the API (in order) is able to provide that missing data then only API will be called.

* If API doesn't give the data that actually we require then API will not be called. In the opposite case, If API gives particular data but we have already that one then API will not be called.


## Business Data Formatting Rules

Normally one should always think about the formatting of the data that is saved. The main goal of this project is to clean or format the business data. For that, the script handles various formatting cases such as data should be in a proper case, all abbreviations should be in the upper case, etc.

>Company Name / Address Formatting Rules

| Formatting Rule | Regex | Example |
| --------------- |------ | :------ |
| Maintain proper case(title case) for all company name | company_name.title() | **Input:** handyman networks<br />**Output:** Handyman Networks |
| Combine Acronym by removing spaces in between | `(\s|^)(?:[a-zA-Z]\.?(\s|$)){2,}` | **Input:** 1)  U S A Handyman Networks<br />2)  U. S. A. Handyman Networks<br />**Output:** USA Handyman Networks |
| Words (Corp, corp., corporation) should be removed or <br />not from the company name | `(?i)\s(corp\.?|corporation)$` | **Input:** 1) L & A CONSTRUCTION CORP 2) Club Corp<br />**Output:** 1) L&A Construction2) Club Corp|
| Remove word from comapny name like, LLC PVT etc. | `-` | **Input:** Watermarke Homes L L C<br />**Output:** Watermarke Homes |
| Find given words (which, dba, /, -) from company name <br />and remove this word and anything after this word | `(?i)\s(which|wall|dba|-|/)(\s|$)` | **Input:** Quick Fix - The Sliding Door Repair<br />**Output:** Quick Fix| 
| All () and inside () should be removed | `[(]{1,}\b(\w+)\b[)]{1,}` | **Input:** Surf To Snow Environmental Resource Management (s2serm)<br />**Output:** Surf To Snow Environmental Resource Management |
| Anything with _ should be flagged, the _ should <br />actually be a comma | `-` | **Input:** Coastal Refrigeration_heating & Conditioning<br />**Output:** Coastal Refrigeration, heating & Conditioning |
| Any number sequence with 'n' in the middle should<br />be writting 3-n-1 | `\b(\d\s[N]\s\d)\b` | **Input:** 3 N 1 Construction<br />**Output:** 3-n-1 Construction |
| In abbreviation before and after '&' should be <br />capitalized and remove space among them | `\b(\w{1,2}\s?[&]\s?\w{1,2})\b` | **Input:** S & b Engineers and Constructors<br />**Output:** S&B Engineers and Constructors|
| Any abbreviation or short name of the company before '&' <br />should be in all caps(capitalized) | `(\s|^)(\w{2}\s[&]\s)` | **Input:** Rj & Associates<br />**Output:** RJ & Associates |
| Convert all abbreviations into upper case | `(^\b[a-zA-Z0-9]{2,4}\b|\b[a-zA-Z]{2,4}\b$)` | **Input:** Osso PLUMBING & HEATING CORP<br />**Output:** OSSO Plumbing & Heating|
| The state name NY or NJ and roman numerals in <br />company name should be capitalized | `(?i)\b(^|\sny|nj|I|II|III|IV|V|`<br />`VI|VII|VIII|IX|X\s|$)\b` | **Input:** Tribro Cconstructio of ny III INC<br />**Output:** Tribro Cconstructio of NY III|
| Convert all given word into lower case.<br />The list of words (and / as / <br />as if / as long as / at / but / by / even if<br /> / for / from / if / if only / <br />in / into / like / near / now that <br />/ nor / of / off / on / on top of <br />/ once / onto / or / out of / over <br />/ past / so / so that / than / that <br />/ till / to / up / upon / with / when <br />/ yet / a / an) | `(?i)\b(\sthe\s)|(\sand\s)|(\sas\s)|(\sas\sif\s)|`<br />`(\sas\slong\sas\s)|(\sat\s)|(\sbut\s)|(\sby\s)|`<br />`(\seven\sif\s)|(\sfor\s)|(\sfrom\s)|`<br />`(\sif\s)|(\sif\sonly\s)|(\sin\s)|(\sinto\s)|`<br />`(\slike\s)|(\snear\s)|(\snow\sthat\s)|`<br />`(\snor\s)|(\sof\s)|(\soff\s)|`<br />`(\son\s)|(\son\stop\sof\s)|(\sonce\s)|`<br />`(\sonto\s)|(\sor\s)|(\sout\sof\s)|`<br />`(\sover\s)|(\spast\s)|(\sso\s)|`<br />`(\sso\sthat\s)|(\sthan\s)|(\sthat\s)|`<br />`(\still\s)|(\sto\s)|(\sup\s)|(\supon\s)`<br />`|(\swith\s)|(\swhen\s)|(\syet\s)|(\sa\s)|(\san\s)\b` | **Input:** Renewal By Anderson Of Los Angeles<br />**Output:** Renewal by Anderson of LOS Angeles |
| Words in the given list should be in title case.<br />(Ms,Miss,Mrs,Mr,Master,Fr,Rev,<br />Dr,Atty,Hon,Prof,Pres,VP,Gov,Ofc) | `(?i)(\s|^)(Ms|Miss|Mrs|Mr|Master`<br />`|Fr|Rev|Dr|Atty|Hon|Prof|Pres|VP|Gov|Ofc)(\s|$)` | **Input:** Landscape Specialties - Maintenance and Installation<br />**Output:** Landscape Specialties |
| Change ordinal number format for the address<br /> by removing space between them and <br />change abbreviation into lower case | `(?i)[0-9]+\s?(?:st|[nr]d|th)` | **Input:** 1ST<br />**Output:** 1st |
| If the address including any <br />Post box number then format 'PO' into <br />capitalized by removing spac between them | `(?i)([p]\.?\s?[o]\.?)(\s?|$)(box)` | **Input:** P o BOX 24 TCB<br />**Output:** PO BOX 24 TCB |
| Parse the street address to manage<br /> proper full street name with abbreviations | `-` | **Input:** 176 RUXTON STREET<br />**Output:** 176 Ruxton St |


## Manage Data

* Data management is the most important part of this entire process. The script first cleans existing data using formatting rules and then calls APIs to get missing data. The response data from the respective APIs will also be cleaned using given formatting rules.

* All business data will be stored into local CSV file first to manage the cleaning process. After that, it will be stored in the spreadsheet. At last, data will be moved into the air table. This way all data will be managed and organized.

## Conclusion

