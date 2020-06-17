# Elasticsearch、Logstash及Kibana雲端架設安裝流程

### 1. [ELK stack介紹]()
### 2. [架構圖](https://github.com/yotzom/Document/blob/master/ELKonCloud.md#2-%E6%9E%B6%E6%A7%8B%E5%9C%96-1)
### 3. [虛擬機&軟體版本](https://github.com/yotzom/Document/blob/master/ELKonCloud.md#3-%E8%99%9B%E6%93%AC%E6%A9%9F%E8%BB%9F%E9%AB%94%E7%89%88%E6%9C%AC-1)
### 4. [ELK stack 安裝](https://github.com/yotzom/Document/blob/master/ELKonCloud.md#4-elk-stack-%E5%AE%89%E8%A3%9D-1)
### 5. [ELK stack 使用情境範例]()
### 6. [Troubleshooting]()
--- 
## 1. ELK stack介紹

## 2. 架構圖

![ELK架構圖](https://github.com/yotzom/Document/blob/master/ELKonCloud_img/ELKstucture.png "ELK架構圖")

### 2.1 在這裡簡單介紹下整個架構的連接架構:
> Elasticsearch作為整個架構中的資料庫，所以只要掛掉或連線不通Kibana就會跟著掛掉!<BR>
> Kibana為架構中的唯一網頁介面，在裡面可以視覺化Elasticsearch中的資料。<BR>
> Logstash可以直接當資料的收集器或者也可以當filebeat的中繼器。<BR>
> Filebeat做為輕量化的收集器可取代在每台客機上安裝Logstash，可以讓客機的額外負載降到最低(安裝包大小約25M，Logstash:159M)。<BR>
  
## 3. 虛擬機&軟體版本
### 3.1 雲端服務虛擬機
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

這裡選擇的VM幾乎都是倒數第一或第二的規格， <BR>
其實這樣的規格在加上SWAP就可以穩定同時Run 2~3個程式了，<BR>
再上去就會有多餘的性能浪費了 (其實是維護費用變高會付不起T_T <BR>
3個平台相比起來GCP大方地給了別的平台兩倍的RAM，RAM大小在VM終至關重要啊!!!
  
### 3.2 作業系統版本
+ GCP : Ubuntu 20.04 LTS
+ AWS : Ubuntu 20.04 LTS
+ AZURE : Ubuntu 18.04.4 LTS

### 3.3 軟體版本
+ Elastixsearch : 7.7.0
+ Kibana : 7.7.0
+ Logstash : 7.7.0
+ FileBeat : 7.7.0 <BR>
  
**注意:以上4個軟體版本最好保持同一版本，最多只能小版本號不一樣(Ex. 7.7.0 7.7.1)，不然極有可能導致一堆不可預期的Bug，此為官方特別提醒的!**
[官網參考連結](https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html "https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html")

## 4. ELK Stack 安裝
*Logstash需要依賴JDK，安裝logstash之前記得先安裝java環境。*
### 4.1 建立Swap
此為選擇性選項，視自己的情況選擇
這裡會使用的原因是因為線上的VM給的RAM都1G左右完全不夠使用，所以才會想到要使用SWAP來加大記憶體空間
- #### 4.1.1 確認現有記憶體空間
```
$ df -h
```
- #### 4.1.2 建立swap檔案
```
$ sudo fallocate -l 1G /swapadd
```
- #### 4.1.3 將檔案標示為swap
```
$ sudo mkswap /swapadd 
```
- #### 4.1.4 啟用此swap
```
$ sudo swapon /swapadd
```
- #### 4.1.5 調整swap和ram的選擇優先程度
1. 查看當前ram&swap使用優先程度 (0~100，0:ram最優先 100:swap最優先)
```
$ cat /proc/sys/vm/swapiness
```
2. 永久修改swapiness值(sudo gedit /etc/sysctl.conf)
```
vm.swappiness=10
```
### 4.2 Elasticsearch 安裝
- #### 4.1.1 安裝
1. 允許套件管理器可以下載https來源的套件 (如有安裝過就不用再重複安裝)<BR><BR>
`sudo apt-get install apt-transport-https`<BR><BR>
2. 下載並安裝elastic的公鑰<BR><BR>
`wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -`<BR><BR>
3. 新增repository<BR><BR>
`echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list`<BR><BR>
4. 更新套件料表並安裝elasticsearch<BR><BR>
`sudo apt-get update && sudo apt-get install elasticsearch`<BR><BR>
5. 設定開機自啟動<BR><BR>
`sudo systemctl enable elasticsearch`<BR>

- #### 4.2.2 設定
1. 修改elasticsearch設定檔<BR><BR>
`sudo vi /etc/elasticsearch/elasticsearch.yml`<BR><BR>
2. 在elasticsearch.yml中找到以下兩個設定值並修改如下:<BR>
+ 監聽的IP (不修改的話僅能允許localhost訪問)<BR>
`network.host: 0.0.0.0`<BR>
+ cluster IP or Hostname (沒有設定cluster的話，必須拿掉註解且[]中保持空白)<BR>
`discovery.seed_hosts:[]`<BR><BR>
  
以下可選擇是否要改:
預設監聽的PORT
`http.port:9200`<BR><BR>
  
- #### 4.2.3 測試

<p align="center">
  <img width="70%" height="70%" src="https://github.com/yotzom/Document/blob/master/ELKonCloud_img/elasticsearch_test.png">
  <BR>測試成功畫面
</p>


### 4.3 Kibana 安裝
- #### 4.3.1 安裝
1. 允許套件管理器可以下載https來源的套件 (如有安裝過就不用再重複安裝)<BR><BR>
`sudo apt-get install apt-transport-https`<BR><BR>
2. 下載並安裝elastic的公鑰<BR><BR>
`wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -`<BR><BR>
3. 新增repository<BR><BR>
`echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list`<BR><BR>
4. 更新套件列表並安裝kibana<BR><BR>
`sudo apt-get update && sudo apt-get install kibana`<BR><BR>
5. 設定開機自啟動<BR><BR>
`sudo systemctl enable kibana`<BR>

- #### 4.3.2 設定
1. 修改kibana設定檔<BR><BR>
`sudo vi /etc/kibana/kibana.yml`<BR><BR>
2. 在kibana.yml中找到以下三個設定值並修改如下:<BR><BR>
+ 監聽的PORT<BR><BR>
`server.port:5601`<BR><BR>
+ 監聽的IP<BR><BR>
`server.host: 內部IP`<BR><BR>
+ elasticsearch的host和port<BR><BR>
`elasticsearch.hosts: ["http://'elasticsearch ip:elasticsearch port'"]`
  
- #### 4.3.3 測試
在瀏覽器輸入Kibana伺服器的外部IP或者Hostname連線
<p align="center">
  <img width="70%" height="70%" src="https://github.com/yotzom/Document/blob/master/ELKonCloud_img/Kibana_test.png">
  <BR>測試成功畫面
</p>
  
### 4.4 Logstash 安裝
- #### 4.4.1 安裝
1. 允許套件管理器可以下載https來源的套件 (如有安裝過就不用再重複安裝)<BR><BR>
`sudo apt-get install apt-transport-https`<BR><BR>
2. 下載並安裝elastic的公鑰<BR><BR>
`wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -`<BR><BR>
3. 新增repository<BR><BR>
`echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list`<BR><BR>
4. 更新套件列表並安裝logstash<BR><BR>
`sudo apt-get update && sudo apt-get install logstash`<BR><BR>
5. 設定開機自啟動<BR><BR>
`sudo systemctl enable logstash`<BR>

- #### 4.4.2 設定
1. 修改Logstash設定檔<BR><BR>
`sudo vi /etc/logstash/conf.d/logstash.yml
> (如果con.d底下沒有logstash.yml就新增一個)
2. 在kibana.yml裡面新增以下內容:<BR>
```
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://'elasticsearch ip:port'"]
    index => "test20200610"
    #user => "elastic" #if you have account verification
    #password => "changeme" #if you have account verification
  }
}
```

- #### 4.4.3 測試
暫不做測試，之後在使用情境範例中將會介紹如何確認

### 4.5 Filebeat 安裝
解釋下為甚麼這裡不選擇用跟前幾個相同的安裝方式，
考慮到使用的情境會是在既有的AP server中使用，
在不破壞原有的架構上增加一個輕量化的程式來達成收集log的目的，
以壓縮檔的方式使用filebeat更符合這程式的本意，
在之後需要大型自動化時也可以更方便的部署。

- #### 4.5.1 安裝
1. 下載filebeat壓縮檔<BR><BR>
`curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.7.0-linux-x86_64.tar.gz`<BR><BR>
2. 解壓縮filebeat壓縮檔<BR><BR>
`tar xzvf filebeat-7.7.0-linux-x86_64.tar.gz`<BR><BR>

- #### 4.5.2 設定
1. 修改Filebeat設定檔<BR>
`sudo vi /path/to/your/filebeat.yml`<BR>
2. 在filebeat.yml裡面找到以下內容並修改:<BR>
```
#----------------------------- Logstash output --------------------------------
output.logstash:
  # The Logstash hosts
  hosts: ["'logstash ip':5044"]

  # Optional SSL. By default is off.
  # List of root certificates for HTTPS server verifications
  #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]

  # Certificate for SSL client authentication
  #ssl.certificate: "/etc/pki/client/cert.pem"

  # Client Certificate Key
  #ssl.key: "/etc/pki/client/cert.key"
```
- #### 4.5.3 測試
暫不做測試，之後在使用情境範例中將會介紹如何確認

## 5. ELK Stack 使用情境範例
*Logstash需要依賴JDK，安裝logstash之前記得先安裝java環境。*
### 5.1 Troubleshooting
