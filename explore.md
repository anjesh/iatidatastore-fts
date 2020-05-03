- [Setup necessary libraries](#orgdd3acdb)
- [Load libraries](#orga4aa033)
- [Define helper functions](#org5ed5571)
- [Data Exploration from datastore](#orgd0ff535)
  - [Explore Specific Activity](#orga706af1)
    - [Prepare query](#org83ca7d2)
    - [Explore transactions](#org1c8aab4)
  - [Explore Publisher](#orgc4dadf4)
    - [Prepare query](#org5b76752)
    - [Making API Call](#orgcbed3f2)
    - [Explore All Transactions](#orgd325f4a)
    - [Explore activities](#org2bdae7c)
    - [Explore by recipient countries](#orgf37f31b)
  - [Explore Country](#org8b22a16)
    - [Prepare query](#org52a25bd)
    - [Make API Call](#org814045f)
    - [Activity Structure](#org4ca2ad0)
    - [Publishers](#org15aefbc)



<a id="orgdd3acdb"></a>

# Setup necessary libraries

```sh
pip install requests
pip install pandas
pip install arrow
pip install matplotlib
pip install tabulate
```


<a id="orga4aa033"></a>

# Load libraries

```python
import requests as rq
import pandas as pd
import arrow
from tabulate import tabulate
```


<a id="org5ed5571"></a>

# Define helper functions

```python
import os
import hashlib
import json

def get_data(url, cache = True):
  cache_file = get_cache_filepath(get_hash(url))
  print(os.path.isfile(cache_file))
  if cache and os.path.isfile(cache_file):
    with open(cache_file, 'r') as fp:
      return(fp.read())
  content = rq.get(url)
  with open(cache_file, 'w') as fp:
    fp.write(content.text)
  return(content.text)
 
def get_out_filepath(filename):
  return(os.path.join(project_out_path, filename))

def get_cache_filepath(filename):
  return(os.path.join(project_cache_path, filename))

def get_hash(url):
  return(hashlib.md5(url.encode('utf-8')).hexdigest())
  
```


<a id="orgd0ff535"></a>

# Data Exploration from datastore


<a id="orga706af1"></a>

## Explore Specific Activity


<a id="org83ca7d2"></a>

### Prepare query

```python
iati_activity = "GB-GOV-1-300417-108"
activity_query_string = (f"https://iati.cloud/search/activity?q=" # return activitiy
        f"iati_identifier:{iati_activity} " # that matches the given iati_identifuer
        f"&fl=iati_identifier,title_narrative_text,description_narrative_text,humanitarian,reporting_org_ref, "
        f"activity_date_start_planned,activity_date_start_actual,activity_date_end_planned,activity_date_end_actual,activity_status_code,"
        f"transaction,budget" # returning the following fields 
        f"&wt=json" # in JSON format
        f"&rows=1") # and return a given number of rows
print(f"{activity_query_string.replace(' ', '%20')}")
```

    https://iati.cloud/search/activity?q=iati_identifier:GB-GOV-1-300417-108%20&fl=iati_identifier,title_narrative_text,description_narrative_text,humanitarian,reporting_org_ref,%20activity_date_start_planned,activity_date_start_actual,activity_date_end_planned,activity_date_end_actual,activity_status_code,transaction,budget&wt=json&rows=1

```python
out_json = json.loads(get_data(activity_query_string))
print(out_json["response"]["numFound"])
print(out_json["response"]["docs"][0]["title_narrative_text"][0])
```

    True
    1
    Funding to support Standby Partnerships in Yemen


<a id="org1c8aab4"></a>

### Explore transactions

```python
def process_transactions(transaction_dict_array):
    loaded_transaction_dict = [json.loads(x) for x in transaction_dict_array]
    df = pd.DataFrame(data = [{
        'iati_identifier': transaction['iati_identifier'],
        'type': transaction['transaction_type']['name'],
        'ref': transaction['ref'],
        'value': transaction['value'],
        'currency': transaction['currency']['code']
        } for transaction in loaded_transaction_dict])
    return(df)
transaction_df  = process_transactions(out_json["response"]["docs"][0]["transaction"])
transaction_df.to_csv(get_out_filepath("activity_transactions.csv"))
print(tabulate(transaction_df, headers = "keys", tablefmt = "orgtbl"))
```

|    | iati<sub>identifier</sub> | type                | ref     | value  | currency |
|--- |------------------------- |------------------- |------- |------ |-------- |
| 0  | GB-GOV-1-300417-108       | Disbursement        | 8758400 | 18128  | GBP      |
| 1  | GB-GOV-1-300417-108       | Disbursement        | 8750340 | 28892  | GBP      |
| 2  | GB-GOV-1-300417-108       | Disbursement        | 8709648 | 23579  | GBP      |
| 3  | GB-GOV-1-300417-108       | Disbursement        | 8709487 | 45853  | GBP      |
| 4  | GB-GOV-1-300417-108       | Disbursement        | 8709480 | 44957  | GBP      |
| 5  | GB-GOV-1-300417-108       | Disbursement        | 8383628 | 31874  | GBP      |
| 6  | GB-GOV-1-300417-108       | Disbursement        | 8345566 | 1760   | GBP      |
| 7  | GB-GOV-1-300417-108       | Disbursement        | 8318581 | 25793  | GBP      |
| 8  | GB-GOV-1-300417-108       | Disbursement        | 8317643 | 25843  | GBP      |
| 9  | GB-GOV-1-300417-108       | Disbursement        | 8316982 | 10749  | GBP      |
| 10 | GB-GOV-1-300417-108       | Disbursement        | 8211312 | 26330  | GBP      |
| 11 | GB-GOV-1-300417-108       | Disbursement        | 8211300 | 14105  | GBP      |
| 12 | GB-GOV-1-300417-108       | Disbursement        | 8201438 | 9906   | GBP      |
| 13 | GB-GOV-1-300417-108       | Disbursement        | 8201409 | 21839  | GBP      |
| 14 | GB-GOV-1-300417-108       | Disbursement        | 8199904 | 13481  | GBP      |
| 15 | GB-GOV-1-300417-108       | Disbursement        | 8199903 | 11820  | GBP      |
| 16 | GB-GOV-1-300417-108       | Disbursement        | 8195424 | -9906  | GBP      |
| 17 | GB-GOV-1-300417-108       | Disbursement        | 8195422 | -21839 | GBP      |
| 18 | GB-GOV-1-300417-108       | Disbursement        | 8192114 | 21839  | GBP      |
| 19 | GB-GOV-1-300417-108       | Disbursement        | 8192115 | 9906   | GBP      |
| 20 | GB-GOV-1-300417-108       | Outgoing Commitment | REFCO3  | 41402  | GBP      |
| 21 | GB-GOV-1-300417-108       | Outgoing Commitment | REFCO2  | 228170 | GBP      |
| 22 | GB-GOV-1-300417-108       | Outgoing Commitment | REFCO1  | 104056 | GBP      |

1.  Transactions by type

    ```python
    transaction_df["value"] = pd.to_numeric(transaction_df["value"])
    print(tabulate(transaction_df.groupby("type").agg({'iati_identifier':'count','value':'sum'}).rename(columns={"iati_identifier":"instances"}),
                  headers = "keys",
                  tablefmt="orgtbl"))
      
    ```
    
    | type                | instances | value  |
    |------------------- |--------- |------ |
    | Disbursement        | 20        | 354909 |
    | Outgoing Commitment | 3         | 373628 |


<a id="orgc4dadf4"></a>

## Explore Publisher


<a id="org5b76752"></a>

### Prepare query

-   Preparing query to get all the activities data of the publisher

```python
iati_publisher = "GB-GOV-1"
publisher_query_string = (f"https://iati.cloud/search/activity?q=" # return activitiy
        f"reporting_org_ref:{iati_publisher} " # that matches the given iati_publisher
        f"&fl=iati_identifier,title_narrative_text,description_narrative_text,humanitarian,reporting_org_ref, "
        f"activity_date_start_planned,activity_date_start_actual,activity_date_end_planned,activity_date_end_actual,activity_status_code,"
        f"transaction,budget,recipient_country_code" # returning the following fields 
        f"&wt=json" # in JSON format
        f"&rows=19278") # and return a given number of rows
print(f"{publisher_query_string.replace(' ', '%20')}")
```

    https://iati.cloud/search/activity?q=reporting_org_ref:GB-GOV-1%20&fl=iati_identifier,title_narrative_text,description_narrative_text,humanitarian,reporting_org_ref,%20activity_date_start_planned,activity_date_start_actual,activity_date_end_planned,activity_date_end_actual,activity_status_code,transaction,budget,recipient_country_code&wt=json&rows=19278


<a id="orgcbed3f2"></a>

### Making API Call

-   Get the data from the cache if available, else make an API call

```python
publisher_out_json = json.loads(get_data(publisher_query_string))
print(publisher_out_json["response"]["numFound"])
print(publisher_out_json["responseHeader"]["params"]["rows"])
```

    False
    19278
    19278


<a id="orgd325f4a"></a>

### Explore All Transactions

1.  Transaction Structure

    ```python
    print(publisher_out_json["response"]["docs"][0]["transaction"][0])
    ```
    
    ```json
    {
        "recipient_regions": [],
        "recipient_countries": [],
        "iati_identifier": "GB-1-204310-101",
        "sectors": [],
        "tied_status": {
            "name": "Untied",
            "code": "5"
        },
        "aid_type": null,
        "finance_type": {
            "name": "Standard grant",
            "code": "110"
        },
        "flow_type": {
            "name": "ODA",
            "code": "10"
        },
        "recipient_region": null,
        "recipient_country": null,
        "sector": null,
        "disbursement_channel": null,
        "receiver_organisation": {
            "narratives": [
                {
                    "language": {
                        "name": "English",
                        "code": "en"
                    },
                    "text": "Correction"
                }
            ],
            "receiver_activity_id": null,
            "receiver_activity": null,
            "type": null,
            "ref": "Not available"
        },
        "provider_organisation": {
            "narratives": [
                {
                    "language": {
                        "name": "English",
                        "code": "en"
                    },
                    "text": "UK - Department for International Development (DFID)"
                }
            ],
            "provider_activity_id": null,
            "type": null,
            "ref": "GB-GOV-1"
        },
        "description": {
            "narratives": [
                {
                    "language": {
                        "name": "English",
                        "code": "en"
                    },
                    "text": "Technical & Advisory services"
                }
            ]
        },
        "currency": {
            "name": "Pound Sterling",
            "code": "GBP"
        },
        "value_date": "2014-07-11",
        "value": "-300000.00",
        "transaction_date": "2014-07-11",
        "transaction_type": {
            "name": "Expenditure",
            "code": "4"
        },
        "humanitarian": null,
        "ref": "5926209"
    }
    ```

2.  Prepare transaction

    ```python
    publisher_transactions_list = []
    for x in publisher_out_json["response"]["docs"]:
        if 'transaction' in x:
            publisher_transactions_list.append(process_transactions(x['transaction']))
    #publisher_transactions_list = [process_transactions(x['transaction']) if 'transacion' in x else "" for x in out_json['response']['docs']]
    publisher_transactions_df = pd.concat(publisher_transactions_list)
    publisher_transactions_df.to_csv(get_out_filepath("publishers_transactions.csv"))
    ```

3.  Group by transaction types

    ```python
    publisher_transactions_df["value"] = pd.to_numeric(publisher_transactions_df["value"])
    print(tabulate(publisher_transactions_df.groupby("type").agg({'iati_identifier':'count','value':'sum'}).rename(columns={"iati_identifier":"instances"}),
                  headers = "keys",
                  tablefmt="orgtbl",
                  floatfmt=("", ".0f",".2f")))
    ```
    
    | type                | instances | value          |
    |------------------- |--------- |-------------- |
    | Disbursement        | 95136     | 79592134014.00 |
    | Expenditure         | 54145     | 17913803669.00 |
    | Interest Payment    | 23        | -681543.00     |
    | Outgoing Commitment | 14084     | 58484408675.00 |
    | Purchase of Equity  | 126       | 3275344889.00  |


<a id="org2bdae7c"></a>

### Explore activities

1.  activity structure

    ```python
    # print(json.dumps(out_json["response"]["docs"][0]["transaction"][0]))
    activity = out_json["response"]["docs"][0]
    activity['transaction'] = ""
    activity['budget'] = ""
    print(json.dumps(activity))
    ```
    
    ```json
    {
        "humanitarian": "0",
        "activity_status_code": "4",
        "title_narrative_text": [
            "Providing quick impact support to the Libyan Government in delivering disarmament activity."
        ],
        "activity_date_end_planned": "2014-09-30",
        "budget": "",
        "activity_date_start_planned": "2013-10-15",
        "recipient_country_code": [
            "LY"
        ],
        "activity_date_end_actual": "2014-09-30",
        "reporting_org_ref": "GB-GOV-1",
        "iati_identifier": "GB-1-204310-101",
        "activity_date_start_actual": "2013-10-15",
        "transaction": "",
        "description_narrative_text": [
            "This activity (Providing quick impact support to the Libyan Government in delivering disarmament activity.) is a component of Providing support to the development of Libyan government counter-proliferation activity reported by DFID, with a funding type of 'Procurement of Services' and a budget of £1,832,812. This component benefits Libya, and works in the following sector(s): Civilian peace-building, conflict prevention and resolution. , with the following implementing partners: Foreign and Commonwealth Office. The start date is 15-10-2013 and the end date is 30-09-2014."
        ]
    }
    ```

2.  count by humanitarian

    1.  Prepare activity by humanitarian
    
        ```python
        activity_df = pd.DataFrame(data = [{
            'iati_identifier': act['iati_identifier'],
            'humanitarian': act['humanitarian']
        } for act in out_json["response"]["docs"]])
        # activity_df.to_csv("cache/publisher_activity_row.csv")
        # print(activity_df.shape)
        print(activity_df.groupby('humanitarian').count())
        ```


<a id="orgf37f31b"></a>

### Explore by recipient countries

1.  Extract Country codelist

    ```python
    from io import StringIO
    s = get_data("http://reference.iatistandard.org/203/codelists/downloads/clv1/codelist/Country.csv")
    cl_country = pd.read_csv(StringIO(s))
    cl_country = cl_country.drop(cl_country.columns[[2,3,4,5,6]], axis=1)
    ```

2.  Activities count by recipient countries

    ```python
    activity_country_df = pd.DataFrame(data = [{
        'iati_identifier': act['iati_identifier'],
        'recipient_country_code': act['recipient_country_code'] if "recipient_country_code" in act else ""
    } for act in publisher_out_json["response"]["docs"]])
    activity_country_df = activity_country_df.explode('recipient_country_code')
    # activity_df.to_csv("cache/publisher_activity_row.csv")
    # print(activity_country_df.shape)
    activity_country_frames = pd.merge(left = activity_country_df, right = cl_country, how = "left", left_on = "recipient_country_code", right_on = "code")
    #print(activity_country_frames.fillna(' NA (missing country)').groupby('name').count()["iati_identifier"])
    
    print(tabulate(activity_country_frames.fillna(' NA').groupby('name').agg({'code':'count'}).rename(columns = {"code":"count"}),
    headers = "keys",
    tablefmt = "orgtbl"))
    ```
    
    <div class="out">
    | name                                                       | count |
    |---------------------------------------------------------- |----- |
    | NA                                                         | 7512  |
    | Afghanistan                                                | 455   |
    | Albania                                                    | 10    |
    | American Samoa                                             | 2     |
    | Angola                                                     | 16    |
    | Armenia                                                    | 8     |
    | Bahamas (the)                                              | 2     |
    | Bahrain                                                    | 2     |
    | Bangladesh                                                 | 470   |
    | Benin                                                      | 2     |
    | Bosnia and Herzegovina                                     | 53    |
    | Brazil                                                     | 15    |
    | Burkina Faso                                               | 5     |
    | Burundi                                                    | 81    |
    | Cabo Verde                                                 | 2     |
    | Cambodia                                                   | 64    |
    | Cameroon                                                   | 27    |
    | Central African Republic (the)                             | 44    |
    | Chad                                                       | 13    |
    | Chile                                                      | 3     |
    | China                                                      | 98    |
    | Colombia                                                   | 1     |
    | Congo (the Democratic Republic of the)                     | 323   |
    | Congo (the)                                                | 4     |
    | Cuba                                                       | 2     |
    | CÃ´te d'Ivoire                                             | 10    |
    | Dominica                                                   | 9     |
    | Ecuador                                                    | 3     |
    | Egypt                                                      | 17    |
    | Eritrea                                                    | 14    |
    | Ethiopia                                                   | 533   |
    | Fiji                                                       | 2     |
    | Gambia (the)                                               | 20    |
    | Georgia                                                    | 8     |
    | Ghana                                                      | 350   |
    | Greece                                                     | 7     |
    | Guatemala                                                  | 5     |
    | Guinea                                                     | 2     |
    | Guyana                                                     | 24    |
    | Haiti                                                      | 54    |
    | Honduras                                                   | 2     |
    | India                                                      | 442   |
    | Indonesia                                                  | 157   |
    | Iran (Islamic Republic of)                                 | 1     |
    | Iraq                                                       | 79    |
    | Jamaica                                                    | 41    |
    | Japan                                                      | 4     |
    | Jordan                                                     | 101   |
    | Kenya                                                      | 530   |
    | Kosovo                                                     | 65    |
    | Kyrgyzstan                                                 | 56    |
    | Lao People's Democratic Republic (the)                     | 11    |
    | Lebanon                                                    | 94    |
    | Lesotho                                                    | 26    |
    | Liberia                                                    | 64    |
    | Libya                                                      | 85    |
    | Madagascar                                                 | 7     |
    | Malawi                                                     | 434   |
    | Maldives                                                   | 2     |
    | Mali                                                       | 11    |
    | Mauritania                                                 | 2     |
    | Moldova (the Republic of)                                  | 48    |
    | Mongolia                                                   | 2     |
    | Montserrat                                                 | 198   |
    | Mozambique                                                 | 316   |
    | Myanmar                                                    | 300   |
    | Nepal                                                      | 446   |
    | Nicaragua                                                  | 11    |
    | Niger (the)                                                | 10    |
    | Nigeria                                                    | 509   |
    | Pakistan                                                   | 564   |
    | Palestine, State of                                        | 191   |
    | Papua New Guinea                                           | 6     |
    | Peru                                                       | 8     |
    | Philippines (the)                                          | 17    |
    | Pitcairn                                                   | 36    |
    | Russian Federation (the)                                   | 12    |
    | Rwanda                                                     | 298   |
    | Saint Helena, Ascension and Tristan da Cunha               | 177   |
    | Saint Lucia                                                | 2     |
    | Samoa                                                      | 2     |
    | Senegal                                                    | 6     |
    | Serbia                                                     | 44    |
    | Sierra Leone                                               | 494   |
    | Solomon Islands                                            | 4     |
    | Somalia                                                    | 398   |
    | South Africa                                               | 95    |
    | South Sudan                                                | 275   |
    | Sri Lanka                                                  | 48    |
    | Sudan (the)                                                | 298   |
    | Syrian Arab Republic                                       | 190   |
    | Tajikistan                                                 | 119   |
    | Tanzania, United Republic of                               | 462   |
    | Thailand                                                   | 2     |
    | Timor-Leste                                                | 2     |
    | Togo                                                       | 2     |
    | Tunisia                                                    | 5     |
    | Turkey                                                     | 17    |
    | Turks and Caicos Islands (the)                             | 15    |
    | Uganda                                                     | 389   |
    | Ukraine                                                    | 64    |
    | United Arab Emirates (the)                                 | 2     |
    | United Kingdom of Great Britain and Northern Ireland (the) | 16    |
    | Uzbekistan                                                 | 2     |
    | Vanuatu                                                    | 4     |
    | Venezuela (Bolivarian Republic of)                         | 4     |
    | Viet Nam                                                   | 98    |
    | Virgin Islands (British)                                   | 2     |
    | Yemen                                                      | 201   |
    | Zambia                                                     | 300   |
    | Zimbabwe                                                   | 239   |
    
    </div>


<a id="org8b22a16"></a>

## Explore Country


<a id="org52a25bd"></a>

### Prepare query

-   Preparing query to get all the activities data of the publisher

```python
iati_country = "NP"
country_url = (f"https://iati.cloud/search/activity?q=" # return activitiy
        f"recipient_country_code:{iati_country} " # that matches the given iati_country
        f"&fl=iati_identifier,title_narrative_text,description_narrative_text,humanitarian,reporting_org_ref,reporting_org_narrative,"
        f"activity_date_start_planned,activity_date_start_actual,activity_date_end_planned,activity_date_end_actual,activity_status_code,"
        f"transaction,budget,recipient_country_code" # returning the following fields 
        f"&wt=json" # in JSON format
        f"&rows=5721") # and return a given number of rows
print(f"{country_url.replace(' ', '%20')}")
print(get_hash(country_url))
```

    https://iati.cloud/search/activity?q=recipient_country_code:NP%20&fl=iati_identifier,title_narrative_text,description_narrative_text,humanitarian,reporting_org_ref,reporting_org_narrative,activity_date_start_planned,activity_date_start_actual,activity_date_end_planned,activity_date_end_actual,activity_status_code,transaction,budget,recipient_country_code&wt=json&rows=5721
    efc00cba3a9ae28eb825c3e988ba3f3f


<a id="org814045f"></a>

### Make API Call

```python
country_out_json = json.loads(get_data(country_url))
print(country_out_json["response"]["numFound"])
print(country_out_json["response"]["docs"][0]["title_narrative_text"][0])
```

    True
    5721
    Support to International Crisis Group


<a id="org4ca2ad0"></a>

### Activity Structure

```python
print(json.dumps(country_out_json["response"]["docs"][0]))
```

```json
{
    "humanitarian": "0",
    "activity_status_code": "4",
    "activity_date_start_planned": "2008-04-01",
    "reporting_org_narrative": [
        "UK - Department for International Development (DFID)"
    ],
    "activity_date_end_planned": "2012-03-31",
    "title_narrative_text": [
        "Support to International Crisis Group"
    ],
    "recipient_country_code": [
        "NP"
    ],
    "reporting_org_ref": "GB-GOV-1",
    "iati_identifier": "GB-1-114041",
    "activity_date_start_actual": "2008-08-22",
    "description_narrative_text": [
        "To support and reinforce the efforts of National Governments, as well as the UN, European Union and other international and regional organisations to build lasting peace and prevent a renewed outbreak of conflict in Nepal."
    ]
}
```


<a id="org15aefbc"></a>

### Publishers

```python
countrywise_activities_df = pd.DataFrame([{
    "iati_identifier": act["iati_identifier"],
    "reporting_org": act["reporting_org_narrative"][0],
    "humanitarian": act["humanitarian"]
} for act in country_out_json["response"]["docs"]])
```

1.  Reporting org

    ```python
    print(tabulate(countrywise_activities_df.groupby("reporting_org").agg({"humanitarian":"count"}).rename(columns = {"humanitarian":"count"}),
    headers = "keys",
    tablefmt = "orgtbl"))
    ```
    
    <div class="out">
    | reporting<sub>org</sub>                                                                                     | count |
    |----------------------------------------------------------------------------------------------------------- |----- |
    | Aapasi Sahayog Kendra Syangja Nepal (ASK Nepal)                                                             | 2     |
    | Adam Smith International Limited                                                                            | 1     |
    | Age International UK (HelpAge International UK)                                                             | 2     |
    | Aidsfonds - Soa Aids Nederland                                                                              | 11    |
    | Anti-Slavery International                                                                                  | 1     |
    | Asian Development Bank                                                                                      | 76    |
    | Australia - Department of  Foreign Affairs and Trade                                                        | 178   |
    | Avocats Sans Frontières                                                                                     | 2     |
    | BBC Media Action                                                                                            | 3     |
    | BRITISH COUNCIL                                                                                             | 2     |
    | Backward Society Education (BASE)                                                                           | 14    |
    | BasicNeeds                                                                                                  | 1     |
    | British Red Cross                                                                                           | 5     |
    | Burns Violence Survivors - Nepal                                                                            | 1     |
    | CARE International UK                                                                                       | 7     |
    | CARE Nepal                                                                                                  | 3     |
    | CATHOLIC RELIEF SERVICES                                                                                    | 2     |
    | CDC Group plc                                                                                               | 1     |
    | Canada - International Development Research Centre/Centre de recherches pour le développement international | 72    |
    | Cardno Emerging Markets                                                                                     | 1     |
    | Carers Worldwide                                                                                            | 1     |
    | Catholic Agency for Overseas Development                                                                    | 1     |
    | Center for Research on Environment Health and Population Activities (CREHPA)                                | 1     |
    | ChildHopeUK                                                                                                 | 1     |
    | Christian Aid                                                                                               | 32    |
    | Coffey International Development Limited, a Tetra Tech Company                                              | 3     |
    | Concern Worldwide                                                                                           | 1     |
    | Concern Worldwide UK                                                                                        | 1     |
    | Crown Agents Limited                                                                                        | 5     |
    | DAI Europe                                                                                                  | 2     |
    | DECP                                                                                                        | 1     |
    | Daria Alexeeva                                                                                              | 1     |
    | Department for Environment, Food, and Rural Affairs                                                         | 1     |
    | Development Initiatives Poverty Research Limited                                                            | 3     |
    | Disability and Development Partners                                                                         | 2     |
    | Disasters Emergency Committee                                                                               | 1     |
    | EMMS International                                                                                          | 4     |
    | Ecorys UK                                                                                                   | 1     |
    | Energy Saving Trust                                                                                         | 1     |
    | Enhanced Integrated Framework                                                                               | 5     |
    | European Commission - Directorate-General for International Cooperation and Development                     | 451   |
    | European Commission - Humanitarian Aid & Civil Protection                                                   | 58    |
    | European Commission - Service for Foreign Policy Instruments                                                | 12    |
    | FMO                                                                                                         | 1     |
    | Federal Ministry for Economic Cooperation and Development                                                   | 90    |
    | Find Your Feet                                                                                              | 2     |
    | Finland - Ministry for Foreign Affairs                                                                      | 679   |
    | Food and Agriculture Organization (FAO)                                                                     | 30    |
    | Free Press Unlimited                                                                                        | 1     |
    | Gavi, the vaccine alliance                                                                                  | 16    |
    | Germany - Federal Foreign Office                                                                            | 14    |
    | Girls’ Education Challenge – Fund Manager PwC                                                               | 2     |
    | Global Affairs Canada                                                                                       | 101   |
    | Global Green Growth Institute (GGGI)                                                                        | 1     |
    | GlobalGiving.org                                                                                            | 488   |
    | Handicap International                                                                                      | 3     |
    | Handicap International Federation                                                                           | 1     |
    | HelpAge International                                                                                       | 10    |
    | ICCO Foundation                                                                                             | 2     |
    | ICF Consulting Services Limited                                                                             | 2     |
    | IDEO.org                                                                                                    | 1     |
    | IMC WORLDWIDE                                                                                               | 10    |
    | INTOSAI Development Initiative                                                                              | 3     |
    | InterAction's NGO Aid Map                                                                                   | 165   |
    | International Alert                                                                                         | 14    |
    | International Child Development Initiatives                                                                 | 2     |
    | International Federation of the Red Cross and Red Crescent (IFRC)                                           | 3     |
    | International Finance Corporation                                                                           | 20    |
    | International Fund for Agricultural Development (IFAD)                                                      | 18    |
    | International Labour Organization (ILO)                                                                     | 64    |
    | International Network of People who Use Drugs (INPUD)                                                       | 1     |
    | International Organization for Migration (IOM)                                                              | 9     |
    | International Rescue Committee Inc.                                                                         | 1     |
    | Ireland - Department of Foreign Affairs and Trade                                                           | 18    |
    | Itad Limited                                                                                                | 1     |
    | KPMG Advisory Limited Tanzania                                                                              | 1     |
    | Landell Mills                                                                                               | 1     |
    | Learning for Life                                                                                           | 1     |
    | Mainline M&E                                                                                                | 1     |
    | Malaria Consortium                                                                                          | 2     |
    | MannionDaniels                                                                                              | 2     |
    | Marie Stopes International                                                                                  | 1     |
    | Mercy Corps Europe                                                                                          | 5     |
    | Millennium Challenge Corporation                                                                            | 63    |
    | Ministry of Foreign Affairs of  Japan                                                                       | 4     |
    | Ministry of Foreign Affairs, Denmark                                                                        | 227   |
    | Ministry of interior of the Slovak Republic                                                                 | 1     |
    | Mondiaal FNV                                                                                                | 1     |
    | Mott MacDonald Limited                                                                                      | 1     |
    | NGO Federation of Nepal                                                                                     | 1     |
    | Nepal Disabled Human Rights Center                                                                          | 1     |
    | Nepal Netra Jyoti Sangh                                                                                     | 1     |
    | Netherlands - Ministry of Foreign Affairs                                                                   | 8     |
    | Netherlands Enterprise Agency                                                                               | 23    |
    | Netherlands Red Cross                                                                                       | 8     |
    | New Zealand - Ministry of Foreign Affairs and Trade - New Zealand Aid Programme                             | 18    |
    | Norad - Norwegian Agency for Development Cooperation                                                        | 102   |
    | Options Consultancy Services                                                                                | 2     |
    | Orbis Charitable Trust                                                                                      | 1     |
    | Orbis International                                                                                         | 1     |
    | Other ministries                                                                                            | 1     |
    | Overseas Development Institute                                                                              | 2     |
    | Oxfam GB                                                                                                    | 83    |
    | Oxfam Novib                                                                                                 | 19    |
    | Oxford Policy Management                                                                                    | 5     |
    | PHASE Worldwide                                                                                             | 4     |
    | PRACTICA Foundation                                                                                         | 2     |
    | PUM Netherlands                                                                                             | 1     |
    | Palladium International Ltd (UK)                                                                            | 2     |
    | People in Need                                                                                              | 7     |
    | Plan International Netherlands                                                                              | 5     |
    | Plan International UK                                                                                       | 8     |
    | PricewaterhouseCoopers Private Limited India                                                                | 2     |
    | PwC                                                                                                         | 2     |
    | RAIN foundation                                                                                             | 4     |
    | RUAF Foundation                                                                                             | 1     |
    | Raleigh International                                                                                       | 1     |
    | Republic of Korea                                                                                           | 101   |
    | Restless Development                                                                                        | 1     |
    | Rutgers                                                                                                     | 1     |
    | SECOURS CATHOLIQUE - CARITAS FRANCE                                                                         | 3     |
    | Saferworld                                                                                                  | 3     |
    | Save the Children International                                                                             | 16    |
    | Save the Children UK                                                                                        | 16    |
    | Sightsavers                                                                                                 | 1     |
    | Simavi                                                                                                      | 5     |
    | Slovak Aid                                                                                                  | 1     |
    | Social Development Direct Limited                                                                           | 1     |
    | South Asian Women Development Forum                                                                         | 2     |
    | Stichting fondsbeheer DGGF lokaal MKB                                                                       | 1     |
    | Street Child                                                                                                | 1     |
    | Sweden, through Swedish International Development Cooperation Agency (Sida)                                 | 418   |
    | Switzerland - Swiss Agency for Development and Cooperation (SDC)                                            | 203   |
    | Terre des Hommes Netherlands                                                                                | 8     |
    | The Global Fund to Fight AIDS, Tuberculosis and Malaria                                                     | 19    |
    | The Global Network of People Living with HIV (GNP+)                                                         | 1     |
    | The Joint United Nations Programme on HIV and AIDS (UNAIDS) Secretariat                                     | 20    |
    | The Leprosy Mission England and Wales                                                                       | 1     |
    | The Louis Berger Group, Inc.                                                                                | 1     |
    | The OPEC Fund for International Development (OFID)                                                          | 22    |
    | The World Bank                                                                                              | 42    |
    | Trianglen                                                                                                   | 44    |
    | UK - Department for Business, Energy and Industrial Strategy (BEIS)                                         | 1     |
    | UK - Department for International Development (DFID)                                                        | 446   |
    | UK - Department of Health (DH)                                                                              | 1     |
    | UK - Foreign & Commonwealth Office                                                                          | 20    |
    | UN Pooled Funds                                                                                             | 56    |
    | UN Women                                                                                                    | 9     |
    | United Mission to Nepal                                                                                     | 1     |
    | United Nations Capital Development Fund                                                                     | 37    |
    | United Nations Central Emergency Response Fund (CERF)                                                       | 67    |
    | United Nations Development Programme (UNDP)                                                                 | 91    |
    | United Nations Educational, Scientific and Cultural Organization (UNESCO)                                   | 36    |
    | United Nations Environment Programme (UNEP)                                                                 | 2     |
    | United Nations High Commissioner for Refugees (UNHCR)                                                       | 6     |
    | United Nations Industrial Development Organization (UNIDO)                                                  | 10    |
    | United Nations Office for Project Services (UNOPS)                                                          | 9     |
    | United Nations Office for the Coordination of Humanitarian Affairs (OCHA)                                   | 1     |
    | United Nations Population Fund                                                                              | 17    |
    | United Nations World Food Programme (WFP)                                                                   | 26    |
    | United States                                                                                               | 436   |
    | University of Amsterdam                                                                                     | 2     |
    | Voluntary Service Overseas                                                                                  | 5     |
    | WASH Alliance International                                                                                 | 2     |
    | WASTE advisers on urban environment and development                                                         | 1     |
    | WSM-World Solidarity                                                                                        | 1     |
    | WWF-UK                                                                                                      | 2     |
    | WYG International                                                                                           | 1     |
    | Water Integrity Network Association                                                                         | 1     |
    | WaterAid                                                                                                    | 1     |
    | Women Engage for a Common Future                                                                            | 1     |
    | Women's Refugee Commission                                                                                  | 1     |
    | World Health Organization                                                                                   | 112   |
    | World Vision International                                                                                  | 1     |
    | World Vision UK                                                                                             | 1     |
    | YoungInnovations                                                                                            | 2     |
    | Zoological Society of London                                                                                | 3     |
    | iDE                                                                                                         | 3     |
    | interburns                                                                                                  | 2     |
    | kidasha                                                                                                     | 1     |
    | market and employment for peace and stability                                                               | 1     |
    
    </div>

2.  Activities count by humanitarian

    ```python
    print(tabulate(countrywise_activities_df.groupby("humanitarian").agg({"humanitarian":"count"}).rename(columns = {"humanitarian":"count"}),
    headers = "keys",
    tablefmt = "orgtbl"))
    
    ```
    
    <div class="out">
    | humanitarian | count |
    |------------ |----- |
    | 0            | 5189  |
    | 1            | 532   |
    
    </div>

3.  Transactions by type

    ```python
    country_transactions_list = []
    for x in country_out_json["response"]["docs"]:
        if 'transaction' in x:
            country_transactions_list.append(process_transactions(x['transaction']))
    #publisher_transactions_list = [process_transactions(x['transaction']) if 'transacion' in x else "" for x in out_json['response']['docs']]
    country_transactions_df = pd.concat(country_transactions_list)
    #publisher_transactions_df.to_csv(get_out_filepath("publishers_transactions.csv"))
    ```
    
    ```python
    pd.options.display.float_format = '{:,.2f}'.format
    country_transactions_df["value"] = pd.to_numeric(country_transactions_df["value"], errors = "ignore")
    print(tabulate(country_transactions_df.groupby("type").agg({'iati_identifier': 'count', 'value': 'sum'}).rename(columns={'iati_identifier':'instances'}),
                   headers="keys",
                   tablefmt="orgtbl",
                   floatfmt=("", ".0f",".2f")))
    
    ```
    
    | type                | instances | value           |
    |------------------- |--------- |--------------- |
    | Disbursement        | 22145     | 35835720236.69  |
    | Expenditure         | 6844      | 6458383441.47   |
    | Incoming Commitment | 633       | 886760704.35    |
    | Incoming Funds      | 3863      | 164381488476.33 |
    | Interest Payment    | 1862      | -9994265.39     |
    | Loan Repayment      | 347       | 661946609.95    |
    | Outgoing Commitment | 6570      | 50350784206.72  |
    | Reimbursement       | 52        | -3281328.66     |