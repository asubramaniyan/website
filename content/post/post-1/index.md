---
title: 'Super De-Duper: Identifying duplicates in messy data'
subtitle: Super De-Duper is the data product I built in 4 weeks during the insight data science fellowship 
summary: 
authors:

categories:
date: 
lastmod: 
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Placement options: 1 = Full column width, 2 = Out-set, 3 = Screen-width
# Focal point options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
image:
  placement: 2
  caption: 'Image credit: [**Unsplash**](https://unsplash.com/photos/CpkOjOcXdUY)'
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---


As we data scientists know, the quality of the data can make or break a data product. Poor quality data has led to a loss of ~611B US dollars per year just in the field of marketing alone [1]. Another report cites duplicate data as the mother of all data quality problems [2]. Duplicate data issues are faced by many companies, who have their data collected from different CRM platforms and accounting systems as the data in these platforms come in different formats. Duplicate records when not removed can negatively affect data products. For example, customer segmentation analysis can report the same customer segmented into multiple tiers when the duplicate records are not removed. The goal of the Super De-Duper project is to identify duplicates records and provide a “Dupe” score, a custom metric that quantifies the duplicity of the records.

I consulted with a startup company in Boston, Tally Street, who provide virtual accountant services to B2B companies and worked on their customer profile data to remove duplicates. The main features of the data that I used for this project are the name of the company, their address, and primary phone number. There are several duplicates in the dataset, one example is shown below:

| Full name      | Company name | Address line1 | Address line 2 | City | State | Zipcode | Phone number |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| Curtis Corrado Insurance Agency (deleted) | Curtis Corrado Insurance Agency | 5975 S Quebec St. | Suite 209 | Centennial | CO | 80111 | (303) 220-7688 |
| Curtis Corrado Insurance Agency | Farmers Insurance-Curt Corrado Agency | 5975 S Quebec St | #209 | Centennial | CO | 80111 | 3032207688 |

Just by looking at these records, we can say that the second record is a duplicate of the first, but this is not an easy task for a computer. So, the goal here is to create an algorithm to identify duplicates.

## Data Challenges:

The two main challenges that I faced in this dataset are (a) very messy data and (b) >50% missing data.

### Messy data 

The data, particularly in the address fields such as city, state, and the country is very messy. For example, a snapshot of the unique value observed in the billing address feature “country” is shown below:

 ![](deduplicate_country_name.png)

For a feature named country, we would expect the feature to contain the name of the countries, for example - the USA, Mexico. But, this feature, in addition to the country name, also contains other metadata such as city name, company name, phone number, email address, so on. It should be noted that even though we have separate fields for them, the data is pretty much jumbled around all the fields.
The data, particularly in the address fields such as city, state and country is very messy. For example, a snapshot of the unique value observed in the billing address feature “country” is shown below: 

### >50% missing data
The second challenge to attain our end goal of finding duplicates is - missing data. More than 50% of the data is missing in this data set. Even after data cleaning (will be explained later), only 42% of the data has non-missing values in all features.

![](deduplicate_missing_data.png)

## Approach:

Considering the diverse nature of data in the three main features (name, address, and phone number) together with the data challenges mentioned above, I decided to create three different pipelines for each of these features to overcome these challenges. They are as follows:

### 1. Data pipeline for company name: 
	
The company profile data set had three features for the name of the company: display name, fully qualified name, and company name. From the analysis, it was clear that the feature ‘display name’ is a redundant feature, and thus, it was removed. The other two name features are retained as they provide varied information.

After feature selection, the data is converted into a vector using TF-IDF vectorization with character embedding. There are duplicates in the dataset because of a spelling error, that I chose to perform character embedding. Also, a lot of companies have ‘ltd’ and ‘Co’ at the end and TF-IDF gives less importance to that.	
Finally, to find duplicate names in the dataset, cosine similarity is chosen. Note that the char embedding resulted in 300K+ features, where Euclidean distance will fail. 

### 2. Data pipeline for Billing address features:

As shown before, there are 10 features available under the billing address, but the data present in these fields are very messy (all jumbled across features). To overcome this challenge, I decided to clean all the columns and combine them into one feature and convert them into geocoordinates.

	Data cleaning:
		- Removing punctuations
		- Removing company names
		- Removing people names
		- Removing emails address
		- Removing phone/fax numbers

Now, the cleaned address is converted into geo-coordinates (latitude and longitude). For this, I utilized geopy together with bing maps. 

Finally, Haversine distance was used to calculate the distance between two geo-coordinates in miles, and cut off of 1 mile was used. If the distance calculates is less than 1 mile, the address is a match.

After data cleaning and combining all columns, the address becomes

### 3. Data pipeline for phone numbers

Data was cleaned to convert all phone number into one format. The extensions are also removed from the phone number and the phone numbers were compared directly with each other. Here the metric was set to 1 whenever the phone number matches and 0 when it is not. 

## Defining the custom metric, the Dupe score:

The three data pipeline for the features - company names, address, and phone number creates 3 metrics. All these three metrics are combined in one metric called the Dupe score. This dupe score ranges from 0 to 1 and quantifies whether a record is duplicate or not. The dupe score is calculated as follows:

Case (1): Weighted average of name, address, and phone number 
Case (2): Weighted average of name and phone number 
Case (3): Weighted average of name and address 
Case (4): Names score

Because the data has >50% missing, one dupe score ideation is not possible and has o be customized for records with varying missing features.

## Evaluation

Evaluating the model for performance is very crucial, particularly for an unsupervised setting where labeled data is not available. To check the model’s performance, I created a validation dataset by randomly sampling 100 records from the data and introduced different types of duplicates as below:
	
	Original record: Curtis Corrado Insurance Agency
	Exact duplicate: Curtis Corrado Insurance Agency
	Duplicates with spelling error: Curt**e**s Corrado Insurance Agency, Curtis C**a**rrado Insurance Agency
	Duplicates with reversed word order of the name: Agency Insurance Corrado Curtis

All of these duplicate records were created with and without address. Altogether, I generated about 800 duplicate records and combined them with the original dataset, and ran the model on this data. The model was run at a different cut-off of the Dupe score - 0.6 - 1 at a frequency of 0.5. The resulting precision-recall curve is shown below:

 ![](deduplicate_precision_recall.png)

At a 0.7 Dupe cut-off score, an F1-score of 0.96 was obtained, which is pretty decent. 

## Results

The same records we referred at the beginning of the blog post when passed through the model identifies the duplicate record with a dupe score of 0.89. The model also provides scores for the three main features, which provide interpretability for the Dupe score.

| Dupe score | names score | Address score | Phone score | Company name | Geo co-ordinates | Phone number |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| 1 | 1 | 1 | 1 | curtis corrado insurance agency | (0.6912930819093975, -1.8309467973299316) | 3032207688 |
| **0.89** | 0.682 | 1 | 1 | farmers insurance-curt corrado agency | (0.6912930819093975, -1.8309467973299316) | 3032207688 |

## Python package + REST API:

I have created a python package called deduplicates which identifies duplicates records through the logic mentioned above.

Tally Street, the company I worked with a javascript shop. My codebase is in python. For them to use my code in the javascript environment, I created a REST API using Flask in python. Now, they can access my code from javascript through this API. Basically, the javascript code will send the record number for which they need duplicates through API to the python package. The find_duplicates module in the python package identifies the duplicate records and sends the duplicate record IDs and the score back to the javascript through API.

![](deduplicate_deliverable)


## References:

1. https://cognitiveseo.com/blog/13094/poor-marketing-data/
2. https://dzone.com/articles/dirty-disparate-duplicated-data-how-to-stop-the-3


