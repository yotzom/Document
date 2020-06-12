# Elasticsearch、Logstash及Kibana雲端架設安裝流程

## 架構圖

![Alt text](https://github.com/yotzom/Document/blob/master/ELKonCloud_img/%E6%9E%B6%E6%A7%8B.png "ELK架構圖")
## 軟體版本

+ Elastixsearch : 7.7.0
+ Kibana : 7.7.0
+ Logstash : 7.7.0
+ FileBeat : 7.7.0 <BR>
  
**注意:以上4個軟體版本最好保持同一版本，最多只能小版本號不一樣(Ex. 7.7.0 7.7.1)，不然極有可能導致一堆不可預期的Bug，此為官方特別提醒的!**
  
### 在這裡簡單介紹下整個架構的連接架構:
> Elasticsearch作為整個架構中的資料庫，所以只要掛掉或連線不通Kibana就會跟著掛掉<BR>
> Kibana為架構中的唯一網頁介面，在裡面可以視覺化Elasticsearch中的資料<BR>
> Logstash可以直接當資料的收集器或者也可以當filebeat的中繼器<BR>
> Filebeat做為輕量化的收集器可取代在美台客機上安裝Logstash，可以讓客機的額外負載降到最低(安裝包大小約10M)<BR>
