# Ansible add multi google cloud platform compute engine
### 1. [Infrastructure as code介紹]()
### 2. [Ansible介紹]()
### 3. [Ansible安裝]()
### 4. [Ansible連接GCP架構與說明]()
### 5. [各細部模組講解]()
### 6. [DEMO : Ansible新增多台GCP VM]()

--- 
## 1. Infrastructure as code介紹

**
在開始介紹Ansible之前先來段前情提要，在IaC(Infrastructure as code)的概念還未出現時，<BR>
要開設一個testing/staging/production的環境時，會先建立三個環境都共用的系統和設定，<BR>
然後再根據三台不同的特性用CLI去分別設定，這時如果要去修改一個共用的設定時，<BR>
必須在三台機器重複做同樣的事，那如果這個修改需要很多步驟呢?<BR>
不只需要花費大量時間做同樣的事，還有可能因為一個疏忽設定錯誤而造成三個環境開始產生差異；<BR><BR>
在IaC的概念出現後，原本要用人來操作CLI不能自動化的思維被打破了，<BR>
取而代之的以寫 code 的方式去部署、管理、維護 Infrastructure，由「人」來擬定好機器要做的腳本，<BR>
讓「機器」透過你寫好的腳本來自動化部署、管理、維護，這帶來了以下幾點好處：<BR>
**

- 速度與可靠性，任何的設定、修改都是全自動化的，這比透過 UI/CLI 操作來的更加快速與可靠。
 
- 可以將編寫好的Code上到Git之類的板控軟體做管理，這意味著可以Code review以及多人合作。
 
- 可以將Code模組化的編寫，之後只要根據不同平台的設定檔修改，就可以輕易的部署到不同的虛擬化平台、雲端平台。
 
- 有順序地執行你所寫的腳本，只要腳本撰寫人在寫時頭腦清楚邏輯清晰就可以解決程式間相依性的問題，之後也可以再修改重複使用。

- testing/staging/production 環境會長得幾乎一模一樣，這對開發和維運人員來講，是作夢也不敢想的事情，也減少了非常多環境相依性的問題。

## 2. Ansible介紹

**
Ansible官網的標題是"Ansible is Simple IT Automation"——簡單的自動化IT工具。

Ansible 是基於Python開發的自動化管理工具，即是IaC的一種實作，實現了以下等功能:
- 批次虛擬機器部屬
- 批次系統設定
- 批次程式設定
- 批次執行命令

Ansible 基於 Python paramiko 開發的，分布式、無需客戶端、輕量級，配置語法使用 YMAL 及 Jinja2模板語言；
是基於module在工作的，本身沒有批量部署的能力。真正具有批量部署的是 Ansible 所運行的模塊，Ansible 只是提供一種框架。進而能減少我們的重複操作，提高工作效率。
**

--- 
