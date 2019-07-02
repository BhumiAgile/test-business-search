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
| Combine Acronym by removing spaces in between | <code>(\s&#124;^)(?:[a-zA-Z]\.?(\s&#124;$)){2,}</code> | **Input:** 1)  U S A Handyman Networks<br />2)  U. S. A. Handyman Networks<br />**Output:** USA Handyman Networks |
| Words (Corp, corp., corporation) should be removed or <br />not from the company name | <code>(?i)\s(corp\.?&#124;corporation)$</code> | **Input:** 1) L & A CONSTRUCTION CORP 2) Club Corp<br />**Output:** 1) L&A Construction2) Club Corp|
| Remove word from comapny name like, LLC PVT etc. | `-` | **Input:** Watermarke Homes L L C<br />**Output:** Watermarke Homes |
| Find given words (which, dba, /, -) from company name <br />and remove this word and anything after this word | <code>(?i)\s(which&#124;wall&#124;dba&#124;-&#124;/)(\s&#124;$)</code> | **Input:** Quick Fix - The Sliding Door Repair<br />**Output:** Quick Fix| 
| All () and inside () should be removed | <code>[(]{1,}\b(\w+)\b[)]{1,}</code> | **Input:** Surf To Snow Environmental Resource Management (s2serm)<br />**Output:** Surf To Snow Environmental Resource Management |
| Anything with _ should be flagged, the _ should <br />actually be a comma | `-` | **Input:** Coastal Refrigeration_heating & Conditioning<br />**Output:** Coastal Refrigeration, heating & Conditioning |
| Any number sequence with 'n' in the middle should<br />be writting 3-n-1 | <code>\b(\d\s[N]\s\d)\b</code> | **Input:** 3 N 1 Construction<br />**Output:** 3-n-1 Construction |
| In abbreviation before and after '&' should be <br />capitalized and remove space among them | <code>\b(\w{1,2}\s?[&]\s?\w{1,2})\b</code> | **Input:** S & b Engineers and Constructors<br />**Output:** S&B Engineers and Constructors|
| Any abbreviation or short name of the company before '&' <br />should be in all caps(capitalized) | <code>(\s&#124;^)(\w{2}\s[&]\s)</code> | **Input:** Rj & Associates<br />**Output:** RJ & Associates |
| Convert all abbreviations into upper case | <code>(^\b[a-zA-Z0-9]{2,4}\b&#124;\b[a-zA-Z]{2,4}\b$)</code> | **Input:** Osso PLUMBING & HEATING CORP<br />**Output:** OSSO Plumbing & Heating|
| The state name NY or NJ and roman numerals in <br />company name should be capitalized | <code>(?i)\b(^&#124;\sny&#124;nj&#124;I&#124;II&#124;III&#124;IV&#124;V&#124;</code><br /><code>VI&#124;VII&#124;VIII&#124;IX&#124;X\s&#124;$)\b</code> | **Input:** Tribro Cconstructio of ny III INC<br />**Output:** Tribro Cconstructio of NY III|
| Convert all given word into lower case.<br />The list of words (and / as / <br />as if / as long as / at / but / by / even if<br /> / for / from / if / if only / <br />in / into / like / near / now that <br />/ nor / of / off / on / on top of <br />/ once / onto / or / out of / over <br />/ past / so / so that / than / that <br />/ till / to / up / upon / with / when <br />/ yet / a / an) | <code>(?i)\b(\sthe\s)&#124;(\sand\s)&#124;(\sas\s)&#124;(\sas\sif\s)&#124;</code><br /><code>(\sas\slong\sas\s)&#124;(\sat\s)&#124;(\sbut\s)&#124;(\sby\s)&#124;</code><br /><code>(\seven\sif\s)&#124;(\sfor\s)&#124;(\sfrom\s)&#124;</code><br /><code>(\sif\s)&#124;(\sif\sonly\s)&#124;(\sin\s)&#124;(\sinto\s)&#124;</code><br /><code>(\slike\s)&#124;(\snear\s)&#124;(\snow\sthat\s)&#124;</code><br /><code>(\snor\s)&#124;(\sof\s)&#124;(\soff\s)&#124;</code><br /><code>(\son\s)&#124;(\son\stop\sof\s)&#124;(\sonce\s)&#124;</code><br /><code>(\sonto\s)&#124;(\sor\s)&#124;(\sout\sof\s)&#124;</code><br /><code>(\sover\s)&#124;(\spast\s)&#124;(\sso\s)&#124;</code><br /><code>(\sso\sthat\s)&#124;(\sthan\s)&#124;(\sthat\s)&#124;</code><br /><code>(\still\s)&#124;(\sto\s)&#124;(\sup\s)&#124;(\supon\s)</code><br /><code>&#124;(\swith\s)&#124;(\swhen\s)&#124;(\syet\s)&#124;(\sa\s)&#124;(\san\s)\b</code> | **Input:** Renewal By Anderson Of Los Angeles<br />**Output:** Renewal by Anderson of LOS Angeles |
| Words in the given list should be in title case.<br />(Ms,Miss,Mrs,Mr,Master,Fr,Rev,<br />Dr,Atty,Hon,Prof,Pres,VP,Gov,Ofc) | <code>(?i)(\s&#124;^)(Ms&#124;Miss&#124;Mrs&#124;Mr&#124;Master</code><br /><code>&#124;Fr&#124;Rev&#124;Dr&#124;Atty&#124;Hon&#124;Prof&#124;Pres&#124;VP&#124;Gov&#124;Ofc)(\s&#124;$)</code> | **Input:** Landscape Specialties - Maintenance and Installation<br />**Output:** Landscape Specialties |
| Change ordinal number format for the address<br /> by removing space between them and <br />change abbreviation into lower case | <code>(?i)[0-9]+\s?(?:st&#124;[nr]d&#124;th)</code> | **Input:** 1ST<br />**Output:** 1st |
| If the address including any <br />Post box number then format 'PO' into <br />capitalized by removing spac between them | <code>(?i)([p]\.?\s?[o]\.?)(\s?&#124;$)(box)</code> | **Input:** P o BOX 24 TCB<br />**Output:** PO BOX 24 TCB |
| Parse the street address to manage<br /> proper full street name with abbreviations | `-` | **Input:** 176 RUXTON STREET<br />**Output:** 176 Ruxton St |


## Manage Data

* Data management is the most important part of this entire process. The script first cleans existing data using formatting rules and then calls APIs to get missing data. The response data from the respective APIs will also be cleaned using given formatting rules.

* All business data will be stored into local CSV file first to manage the cleaning process. After that, it will be stored in the spreadsheet. At last, data will be moved into the air table. This way all data will be managed and organized.
