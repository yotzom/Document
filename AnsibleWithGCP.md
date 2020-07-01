# Ansible add multi google cloud platform compute engine
### 1. [Ansible介紹]()
### 2. [Ansible安裝]()
### 3. [Ansible連接GCP架構與說明]()
### 4. [各細部模組講解]()
### 5. [DEMO : Ansible新增多台GCP VM]()

--- 
## 1. Ansible介紹
ELK是由三個工具組合而成的，Elasticsearch + Logstash + Kibana，這三個工具組合形成了一套監控架構，許多公司用此架構來建立視覺化的log分析系統。

1. ElasticSearch
ElasticSearch是一個基於Lucene的搜尋伺服器。它提供了一個分散式多用戶的全文搜尋引擎，並提供RESTfulAPI接口。Elasticsearch是用Java開發的，並作為Apache許可條款下的開放源碼發布，是當前流行的企業級搜尋引擎。設計用於雲處理，能夠達到實時搜尋，穩定，可靠，快速，安裝使用方便。

2. Logstash
Logstash是一個用於管理log和事件的工具，你可以用它去收集log、轉換log、解析log並將他們作為數據提供給其它功能使用，例如搜尋、儲存等。

3. Kibana
Kibana是一個前端log視覺化網頁，它可以非常詳細的將日誌轉化為各種圖表，為用戶提供強大的數據視覺化。

1.1 ELK優點
1. 快速的搜尋功能，elasticsearch可以以分散式搜尋的方式快速搜尋，而且支持DSL的語法來進行搜尋，簡單的說，就是通過類似配置的語言，快速篩選數據。

2. 強大的展示功能，可以展示非常詳細的圖表信息，而且可以定製展示內容，將數據可視化發揮的淋漓盡致。

3. 分散式功能，能夠解決大型群集維運工作很多問題，包括監控、預警、log收集解析等。

1.2 ELK可以用來做什麼?
- 分散式log數據集中式查詢和管理
- 系統監控，包含系統硬體和應用各個組件的監控
- 故障排查
- 安全信息和事件管理
- 報表功能
- log查詢，問題排查，上線檢查
- 伺服器監控，應用監控，錯誤報警，Bug管理
- 性能分析，用戶行為分析，安全漏洞分析，時間管理

--- 
