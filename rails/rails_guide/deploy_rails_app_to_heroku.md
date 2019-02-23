# 部署 Rails App 至 Heroku
部署(Deploy)就是把你的專案丟到某個 server 上去跑，讓其他人能去那個 server 看你的作品。

## 常見的雲端伺服器服務類型
* IaaS (Infrastucture as a service) = 硬體機器
* PaaS (Platform as a service)      = 硬體機器 + OS
* SaaS (Software as a service)      = 硬體機器 + OS + 軟體服務

---
## Heroku 介紹：PaaS = 硬體機器 + OS
有免費方案，也能快速升級成付費方案，很適合初期試水溫。  

使用規則與額度：https://www.heroku.com/pricing

```
# 瞭解一下行情：
Free
Ideal for experimenting with cloud applications in a limited sandbox.

CORE PLATFORM FEATURES
SLEEPS AFTER 30 MINS OF INACTIVITY
USES AN ACCOUNT-BASED POOL
OF FREE DYNO HOURS
CUSTOM DOMAINS
512 MB RAM │ 1 web/1 worker

Heroku Teams
Collaborate on multiple projects, manage team members and resources, get app-level control over permissions, and keep it all on one bill for greater visibility.

FREE 1-5 team members

$10 per month 6-25 team members
```
據瞭解後續升級服務不便宜，所以目前利用免費的居多：SLEEPS 不完全清楚是什麼意思，APP休眠模式？

有 google 到 "Keep free Heroku app awake for free using Google App Script in 2017" 這類訊息，看來是偵測使用者有沒有繼續給予刺激，如果沒有就自動睡 30 mins ？

---
部署 Heroku 常用指令：
```rb
# 顯示 10 筆 log
heroku logs --num 10
# 持續更新 最新的 log
heroku logs --tail
```

---
## 使用 Heroku 部署你的 Rails App
1. 註冊：https://www.heroku.com/。
2. 收信，點擊驗證連結。
3. 設定密碼，要求很高。
4. 回 terminal 操作：  
```bash
# MAC：
brew install heroku
```
```bash
# LINUX：
wget -qO- https://cli-assets.heroku.com/install-ubuntu.sh | sh
```
5. 登入 heroku：`heroku login`

6. 建立新專案：
```bash
heroku create rails-app-test-1
```
**這邊要注意的是名稱是否會是唯一(是否與其他名稱衝突)。**

7. 調整 Rails 環境設定  
a. 理解 Rails 執行模式：  
   development, test, production，根據不同的模式 Rails 會跑不一樣的環境。  
b. 我們需要變更 Rails 使用的資料庫，因爲 Heroku 預設是 PostgreSQL。  
   編輯 `Gemfile`：

```rb
# 移動 sqlite3 至 developmetn and test group
# Use sqlite3 as the database for Active Record
gem 'sqlite3'

# ---

# 新增
# for Heroku_test
group :production do
  gem 'pg', '~> 0.18'
end
```
**這邊要注意的是要指定版本，不然會遇到[錯誤](https://azraeil.gitbooks.io/my-note/content/rails/rails_guide/deploy_rails_app_to_heroku.html#%E6%88%91%E9%81%87%E5%88%B0%E7%9A%84-pg-version-%E5%95%8F%E9%A1%8C1)**
8. 回到 terminal：

```bash
# MAC
brew install postgresql
```

```bash
# Linux
sudo apt-get install libpq-dev
```

9. 安裝 PostgreSQL：`bundle install`

10. 編輯 rails_project/config/database.yml
```yml
# 將 production 的部分改成你指定要的
production:
  adapter: postgresql
  encoding: Unicode
```

11. 部署你的 Rails app
a. git commit 你的 Rails App
```bash
git add .
git commit
```

b. heroku git 遠端節點設定 ：  
`heroku login`  
忘紀剛才建立的新專案的話：  
`heroku list`  
設定 heroku git 遠端節點：  
`heroku git:remote -a rails-app-test-1`

c. 將 Rails app 上傳至 Heroku  
`git push heroku master`  
要等一陣子。  

d. 遷移 Heroku 上的 Rails 資料庫  
`heroku run rake db:migrate`
> 看來要在 heroku server 上執行指令就是：  
> `heroku run ` + `你要執行的命令`

e. 測試 上線的 Rails App：  
`heroku open`

值得注意的是 之後 git push 指令似乎都是要下：  
`git push heroku master`

> 因爲把 master branch 推上去，heroku 才會吃的樣子，推其他 branch 上去可以，但是 heroku 不會有其他反應。

---
### 我遇到的 pg version 問題#1  
`
Caused by:
Gem::LoadError: can't activate pg (~> 0.18), already activated pg-1.0.0. Make sure all dependencies are added to Gemfile.
`

他說我裝得 pg 版本太新了，所以我還要回去 Gmefile 再改設定降版本。

我就直接照著改了：

```rb
# for Heroku_test
group :production do
  gem 'pg', '~> 0.18'
end
```

跑完下 `bundle install`,  
`git add Gemfile`, `git commit`, `git push heroku master`

---

### 接著遇到的 git push failed 問題#2
我好奇下 `git status` 發現：  
`modified:   Gemfile.lock`  
這檔案剛才也有出現在 git push error 的 log 裡。  

下 `git diff Gemfile.lock` 觀察發生了什麼改動：
```rb
--- a/Gemfile.lock
+++ b/Gemfile.lock
@@ -98,7 +98,7 @@ GEM
     nio4r (2.2.0)
     nokogiri (1.8.1)
       mini_portile2 (~> 2.3.0)
-    pg (1.0.0)
+    pg (0.21.0)
     public_suffix (3.0.1)
     puma (3.11.0)
     rack (2.0.3)
@@ -192,7 +192,7 @@ DEPENDENCIES
   jbuilder (~> 2.5)
   jquery-rails
   listen (>= 3.0.5, < 3.2)
-  pg
+  pg (~> 0.18)
   puma (~> 3.7)
   rails (~> 5.1.4)
   sass-rails (~> 5.0)
```

原來是我改 Gemfile ，Gemfile.lock 也會有相應的改動啊，我沒 git add Gemfile.lock 導致 Heroku 上的 Gemfile 跟 Gemfile.lock 互相衝突，所以 push 失敗。

> 原來 push 階段就失敗了，我還以爲要到 migrate 呢。

OK，push 成功，而且有看到安裝 pg 0.21。

記得 migrate：`heroku run rake db:migrate`

---
### heroku open 後發現 `The page you were looking for doesn't exist.`
If you are the application owner check the logs for more information.

> 看 log 的地方在右上角 `Open app` 的右邊 `More`，按下去才有可以看 heroku log 的地方。  
> 用 command 看比較快...，`heroku logs`，`heroku logs --num 10`，`heroku logs --tail`。

簡單喵到了 route 的訊息，有沒有想到你的首頁是什麼？  
本來是 `0.0.0.0:3000/users`。  
上傳到 Heroku 就是  
 `https://rails-app-test-1.herokuapp.com/users` 囉。  

還讓我想了一下，想說沒地方錯了(吧)，確實，是我忘了我的 route 設定了。

---

補充：更新 bundler   
log 前面有提示 我的 bundler 版本有點舊。  
更新本機 bundler：  
`gem install bundler`

要更新 heroku 上的 bundler：
`heroku run gem install bundler`
