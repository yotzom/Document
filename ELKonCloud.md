# Elasticsearch、Logstash及Kibana雲端架設安裝流程

### 1. 架構圖
### 2. Elasticsearch 7.7.0 安裝
### 3. Kibana 7.7.0 安裝
### 4. Logstash
### 5. Filebeat
### 6. Troubleshooting
--- 
## 1. 架構圖

![ELK架構圖](https://github.com/yotzom/Document/blob/master/ELKonCloud_img/ELKstucture.png "ELK架構圖")

### 1.1 在這裡簡單介紹下整個架構的連接架構:
> Elasticsearch作為整個架構中的資料庫，所以只要掛掉或連線不通Kibana就會跟著掛掉!<BR>
> Kibana為架構中的唯一網頁介面，在裡面可以視覺化Elasticsearch中的資料。<BR>
> Logstash可以直接當資料的收集器或者也可以當filebeat的中繼器。<BR>
> Filebeat做為輕量化的收集器可取代在每台客機上安裝Logstash，可以讓客機的額外負載降到最低(安裝包大小約10M)。<BR>
  
### 1.2 雲端服務虛擬機
+ Google Cloud Platform - g1-small
> + CPU : 1 core
> + RAM : 1.7 G
> + Disk : 10 G
+ Amazon Web Services - t2.micro
> + CPU : 1 core
> + RAM : 1 G
> + Disk : 30 G
+ Microsoft Azure - 標準B1s
> + CPU : 1 core
> + RAM : 1 G
> + Disk : 30 G

這裡選擇的VM幾乎都是倒數第一或第二的規格，其實這樣的規格在加上SWAP就可以穩定同時Run 2~3個程式了，<BR>
再上去就會有多餘的性能浪費了 (其實是維護費用變高會付不起T_T <BR>
3個平台相比起來GCP大方地給了別的平台兩倍的RAM，RAM大小在VM終至關重要啊!!!
  
### 1.3 作業系統版本
+ GCP : Ubuntu 20.04 LTS
+ AWS : Ubuntu 20.04 LTS
+ AZURE : Ubuntu 18.04.4 LTS

### 1.4 軟體版本
+ Elastixsearch : 7.7.0
+ Kibana : 7.7.0
+ Logstash : 7.7.0
+ FileBeat : 7.7.0 <BR>
  
**注意:以上4個軟體版本最好保持同一版本，最多只能小版本號不一樣(Ex. 7.7.0 7.7.1)，不然極有可能導致一堆不可預期的Bug，此為官方特別提醒的!**
