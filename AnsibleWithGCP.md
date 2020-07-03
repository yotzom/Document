# Ansible add multi google cloud platform compute engine
### 1. [Infrastructure as code介紹](https://github.com/yotzom/Document/blob/master/AnsibleWithGCP.md#1-infrastructure-as-code%E4%BB%8B%E7%B4%B9-1)
### 2. [Ansible介紹](https://github.com/yotzom/Document/blob/master/AnsibleWithGCP.md#2-ansible%E4%BB%8B%E7%B4%B9-1)
### 3. [Ansible安裝](https://github.com/yotzom/Document/blob/master/AnsibleWithGCP.md#3-ansible%E5%AE%89%E8%A3%9D-1)
### 4. [DEMO : Ansible新增多台GCP VM]()

--- 
## 1. Infrastructure as code介紹

在開始介紹Ansible之前先來段前情提要，在IaC(Infrastructure as code)的概念還未出現時，<BR>
 
要開設一個testing/staging/production的環境時，會先建立三個環境都共用的系統和設定，<BR>
 
然後再根據三台不同的特性用CLI去分別設定，這時如果要去修改一個共用的設定時，<BR>
 
必須在三台機器重複做同樣的事，那如果這個修改需要很多步驟呢?<BR>
 
不只需要花費大量時間做同樣的事，還有可能因為一個疏忽設定錯誤而造成三個環境開始產生差異；<BR><BR>
 
在IaC的概念出現後，原本要用人來操作CLI不能自動化的思維被打破了，<BR>
 
取而代之的以寫 code 的方式去部署、管理、維護 Infrastructure，由「人」來擬定好機器要做的腳本，<BR>
 
讓「機器」透過你寫好的腳本來自動化部署、管理、維護，這帶來了以下幾點好處：<BR>

- 速度與可靠性，任何的設定、修改都是全自動化的，這比透過 UI/CLI 操作來的更加快速與可靠。
 
- 可以將編寫好的Code上到Git之類的板控軟體做管理，這意味著可以Code review以及多人合作。
 
- 可以將Code模組化的編寫，之後只要根據不同平台的設定檔修改，就可以輕易的部署到不同的虛擬化平台、雲端平台。
 
- 有順序地執行你所寫的腳本，只要腳本撰寫人在寫時頭腦清楚邏輯清晰就可以解決程式間相依性的問題，之後也可以再修改重複使用。

- testing/staging/production 環境會長得幾乎一模一樣，這對開發和維運人員來講，是作夢也不敢想的事情，也減少了非常多環境相依性的問題。

--- 
## 2. Ansible介紹

Ansible官網的標題是"Ansible is Simple IT Automation"——簡單的自動化IT工具。

Ansible 是基於Python開發的自動化管理工具，即是IaC的一種實作，實現了以下等功能:
- 批次虛擬機器部屬
- 批次系統設定
- 批次程式設定
- 批次執行命令

Ansible 基於 Python paramiko 開發的，分布式、無需客戶端、輕量級，配置語法使用 YMAL 及 Jinja2模板語言；
是基於module在工作的，本身沒有批量部署的能力。真正具有批量部署的是 Ansible 所運行的模塊，Ansible 只是提供一種框架。進而能減少我們的重複操作，提高工作效率。

--- 
## 3. Ansible安裝

### 3-1.更新apt套件清單
```
sudo apt update
```
### 3-2.安裝Ansible
```
sudo apt install ansible
```
### 3-3.確認是否安裝成功
```
ansible --version
```
### 3-4.ansible的安裝就是這麼簡單，跟其他linux程式安裝方式一樣，好的那麼就...結束 收工回家(X
```
當然沒那麼快，這裡先測試連線到GCP的VM測試是否真的可以用
```
### 3-5.確認GCP VM的SSH是否有開啟
```
sudo service sshd status
```
### 3-6.在家目錄下新增一個檔案檔名為hosts，裡面加入要host的IP or HOSTNAME
```
格式如下
35.185.129.150 gcp
```
### 3-7.測試連線，成功畫面如下圖
```
ansible all -i ./hosts -u host_username --private-key ~/gcp_ssh_key/id_rsa -m ping
這裡因為是要連接的是GCP的VM，所以才會用到ssh_key沒有的話可以使用--ask-pass會在需要密碼時詢問你，
不過還是會鼓勵使用SSH KEY，畢竟之後在做自動化時你不會想要將明碼的密碼存在設定檔中吧。
```
<p align="center">
  <img width="70%" height="70%" src="https://github.com/yotzom/Document/blob/master/AnsibleWithGCP_img/ansible_test_result.png">
  <BR>測試成功畫面
</p>

---
## 4. Ansible連接GCP架構與說明
本階段會以GCP官方的範例文件實作一遍，並在實作後介紹playbook中所有用到的模組
### 4-1. DEMO
#### 4-1-1. PULL GCP官方DEMO的playbook
```
clone https://github.com/GoogleCloudPlatform/compute-video-demo-ansible.git
```
#### 4-1-2. 根據readme修改相關設定

<p align="center">
  <img width="70%" height="70%" src="https://github.com/yotzom/Document/blob/master/AnsibleWithGCP_img/preparation.png">
  <BR>前置作業
</p>
 
 ##### 1.在GCP創建一個帳戶並新增一個專案
 
 ##### 2.點選頁面左上方的切換專案，複製下專案ID <BR>
 
 ![參考畫面](https://github.com/yotzom/Document/blob/master/AnsibleWithGCP_img/project_number.png)

 ##### 3.必須在你的專案中新增一個付款方式，後續才能使用GCE(使用試用帳戶也是可以的)
 
 ##### 4.建立一個此專案的服務帳戶，名子推薦為"demo-ansible"，也可以自訂但不推薦使用GCP預設服務帳戶名稱， 並且以此帳號創建一個JSON格式的SSH私鑰。 
 
 [參考連結](https://cloud.google.com/compute/docs/access/service-accounts#serviceaccount)

 ##### 5.在安裝Ansible的主機安裝Cloud SDK，並成功驗證身分 
 
 [參考連結](https://cloud.google.com/sdk/)

 ##### 6.確保在GCP已經新增好你的Ansible主機SSH公鑰，並已測試過可以使用```gcloud compute ssh```連線現有的GCE Instance 
 
 [參考連結](https://medium.com/@pk60905/google-compute-engine-%E5%A6%82%E4%BD%95%E9%80%A3%E7%B7%9A%E8%87%B3linux%E5%9F%B7%E8%A1%8C%E5%80%8B%E9%AB%94-b048fdfbaff3)

 ##### 7.預設你的SSH私鑰位置是$HOME/.ssh/google_compute_engine
 
#### 4-1-3. 安裝依賴套件
```
sudo apt install -y build-essential git python-dev python-pip
```

#### 4-1-4. 安裝Google認證和requests套件
```
pip install googleauth requests
```

#### 4-1-5. For the purposes of the demo, you can set a couple of environment variables to simplify your commands and SSH interactions.
```
export ANSIBLE_HOST_KEY_CHECKING=False
```

#### 4-1-6. 修改gce_vars/auth中的project&credentials_file
```
# Google Compute Engine required authentication global variables
# (Replace 'YOUR_PROJECT_ID' with the Project ID used in creating your GCP project.)
project: i-monolith-XXXXXXX #<-修改此項project_id
credentials_file: ~/i-monolith-XXXXXXX-XXXXXXXXXXXX.json #<-修改此項 服務帳戶認證json檔存放位置
auth_kind: serviceaccount
```

#### 4-1-7. 修改group_vars/all中的ansible_ssh_user&ansible_ssh_private_key_file
```
# group_vars/all
# --------------------
#
# Variables to be set for dynamic hosts.
---
ansible_ssh_user: user
ansible_ssh_private_key_file: ~/.ssh/google_compute_engine
```

#### 4-1-8. 修改gce_vars/auth中的zone&region
```
---
# Variables defined for us-central1-a
name_zonea: myinstance1
name_zoneb: myinstance2
zone: asia-east1-a
region: asia-east1
```

#### 4-1-9. 執行playbook
```
ansible-playbook -i ansible_hosts site.yml
```

#### 4-1-10. 成功結果
在GCP上可以看到新增的兩台新機器，在網頁上輸入這兩台機器的外部IP如果有順利開啟網頁就代表成功了!

### 4-2. 各模組介紹
      - gcp_compute_address
      - gcp_compute_instance
      - wait_for
      - add_host
      - command
      - apt
      - template
      - file
      - copy
      - service
### 4-3. 連結架構介紹
這裡會特別來說明Ansible和GCP如何連接的原因是，
我在剛碰到Ansible時，看到的介紹和實作教學都只有實際操作並沒有說背後是怎麼運作的，
可是就會很好奇Ansible到底是怎麼告訴GCP我要新增機器的，是用RESTfulAPI嗎?還是GCP有寫好的程式呢?
針對上述的問題，最好的解答就是...去看新增GCE的Module怎麼寫的!

1.搜尋*gcp_compute_instance*.py檔案
```
sudo find / -name *gcp_compute_instance*.py
```
