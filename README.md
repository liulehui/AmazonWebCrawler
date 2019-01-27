 
### 12.31 理论课
#### Agenda
* 互联网广告形式
* 广告数据结构
* 搜索广告基本流程
* 爬虫
* 搜索日志数据

#### 互联网广告形式
1. search ads 搜索广告  
logic: match keywords to user's query  
format: text, image  
position: main line, side bar, etc.   
2. native ads 原生广告  
logic: match keywords to web page or APP context
format: text, image, style should also match context of web page  
position: embedded in original content
3. display ads 显示广告  
logic: match user's demographic, interests to ad's category interests collected from user behavior:click engage time  
format: image, animation, video, audio  
position: sidebar, top, bottom of page, etc..
  
**Common:**    
logic: match ads to something
something: users's query, interest, demographic, context of web page  
**click rate**:   
native > search > display

#### 广告数据结构
Campaign  
● A campaign focuses on a theme or a group of products  
● set a budget  
● choose your audience  
● write your ad including keywords, ad content 
**Quiz:**   
**click rate = click / impression**   
**potential clicks related to pricing algorithm** 

##### data structure
**ad**  

* AdID  
* CampaignID  
* Keywords  
* Bid  
* Description  
* Landing Page  
  
**compaign**

* CampaignID
* Budget (reduce as ad got click) 

#### search ads 基本流程
1. query understand   
2. select ads
3. filter ads
4. rank ads
5. select top K ads
6. pricing
7. allocate ads

##### query understand  
cleaning  
● remove stop words: a, an, the ...  
● remove ending s, ‘s  

query intent prediction  
● predict user’s intent  
● buy harry potter dvd online -> harry potter dvd  
● need query history log  

query expansion    
● nike running shoe -> nike running sneakers  
● software developer -> software engineer  

result: a list of query

##### select ads
● send query understand result to index and select as much candidates as possible  
● apply information retrieval algorithm on index server  
● how to make it scalable ?  
● what if QPS > 10 K ? ( distributed ads index server ) 

##### rank ads  
● given ad candidates, how to calculate rank score  
● a good rank score should reflect  
★ relevance between query and ad  
★ click probability  
★ bid price  

##### Pricing

* how to charge advertiser
* is it fair to charge them by bid price?  
**Quiz:**  
cost per click increase is good not bad? 

#### Web Crawler
Web crawling is about making requests and extracting data from the response.  
##### Where to start crawling

* start with feeds file
* a list of website
* request url with different parameters
* sample feeds file 

| Query | Bid | Campaign ID |
| ------ | ------ | ------ |
| sofa | 12.5 | 8022 |
| corner sofa | 13.6 | 8023 |
| leather sofa | 13.8 | 8024 |

##### Network I/O is the bottleneck

* need multithread

##### avoid bot detection

* What if website like amazon block your IP?
* http 503 service unavailable
* Spoof http header: User Agent, Accept, Accept-Encoding
* Proxy Service: Rotate IPs using a list of over 100+ proxy servers  

##### how to be resilient
● What if crawler crash due to uncaught exception?   
● How to avoid crawl same url ?  
● How to handle url failed to crawl ?     
To tackle these problems:  
● log crawled url  
● log failed url  
● log exception  

#### Search log data

* Device IP (random generated)
* Device ID (random generated)
* Session ID 
* Query
* Ad ID
* Clicked   (0/1)
* Ad category
* Query category

**Goal:**  
Generate click log for query intent prediction and click prediction  
**method**  
reverse engineering  

##### positive feature

* IP
* device_id (some IP like click ads )
* Ad_id
* queryCategory_AdsCategory match
* query_campaign_ID match
* query_Ad_id match 

##### Nagative feature

* mismatched query_category and ads_category
* mismatched query_campaignID
* mismatched query_adid
* lowest campaignID weight, lowest adid weight per query group 


##### Steps:  
Step 1: Segment users to 4 level  
● 10000 IP, 5 device id per IP, user = ip + device_id  
● level 0: 5% of user who click for each query  
● level 1: 5% of IP whose 1st 2 device id click for each query,rest 3 device_id click on 70% of query, rest of 30% query no click
● level 2: 40% of IP from where we random select 1 device_id click for 50% of query, rest of 50% query no click, rest of 4 device_id no click  
● level 3: 50% never click   

Step 2: assign campaign ID, Ad Id, Click(0/1),Ad category_Query category(0/1) to user  
● assign weight to campaign ID, Ad IDs  
● group ad data by Query group Id    
How to calculate weight for each Ads, campaign?  
● For each ad per campaign , calculate relevance between ad and query, ad_weight_i = relevance_i / sum(relevance_i) , i is index of ad for current campaign  
● For each campaign per query group, campaign_weight_j = sum of ad relevance in current campaign /total_relevance_cross_campaign    

**How to assign weighed campaignID, AdId to user?**  
Weighted Random sampling  
Given n item, each one has weight w[i], these weights form the unnormalized probability distribution we want to sample from, each item should have probability w[i]/sum(w) of being chosen.