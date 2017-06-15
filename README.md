# My_store折價卷

- [系統功能](#系統功能頁面)
- [MODEL設計](#model設計)
- [技術運用(Gem)](#技術運用gem)
- [實作](#開始實作)
  - [Step.1 建立my_store專案](#step1-建立my_store專案)
  - [Step.2 加入gem套件](#step2-加入gem套件)
  - [Step.3 scaffold建立CRUD](#step3-scaffold建立product和coupon的crud)
  - [Step.4 表單驗證](#step4-表單驗證)
 
今天要做一個ruby的應用，建立一個書局，有商品、折價卷優惠等功能，主要是要讓商品輸入折價卷code後，會成功扣款，其中並加上表單的驗證處理

**以概念性驗證(POC)的概念，以宏觀的角度去實踐系統**

驗證概念所需的技術架構(gem)，建立好系統架構，系統功能完整之後，再去處理畫面性的部分
  - 好處是可以，容易整合他人意見，也可在coding過程中，避免一直計較細節，本末倒置耽誤開發進度。
  
    (自己真的常常這樣= =+ )

  
## 系統功能(頁面)
 
 
- 書本專區(product)

  書本CRUD

- 折價專區(coupon)

  折價卷CRUD


## MODEL設計

我們在建立MODEL的時候，Coupon我們有設一個`allow_multiple_use`欄位，決定該折價卷是否可重複使用，所以我們Product需要欄位`coupon_id`來紀錄折價卷，以方便後續判斷。

**(`coupon_id`欄位之後再用`add_columns`建立)**



**Product 商品資料表**

 - title: String   
 - price: integer   
 - description: text
 - coupon_id: integer

 - 關係：belongs_to :coupon

**Coupon 折價卷資料表**

 - code: String
 - discount: integer
 - begin_at: datetime
 - end_at: datetime
 - allow_multiple_use: boolean

 - 關係：has_many :products
 
**其中要注意的是，一張折價卷可用在多本不同本書上，是一對多的關係，所以 `has_many :products`和`belongs_to :coupon`建好Model記得加上去**

 
## 技術運用(Gem)
  - simple_form
  - bootstrap
> 備註：建議先安裝這兩個套件~

> 先加入這兩個gem後，再用`rails g scaffold`長出來的表單頁面，表單就會用`simple_form`自動生成喔~而且還變美美的!
## 開始實作~

### Step.1 建立my_store專案
  `rails new my_store`
  
### Step.2 加入gem套件

**[Bootstrap-sass](https://github.com/momo200e/Ruby_Rails_Notes/blob/master/Gem_Notes.md#bootstrap-sass) && [Simple_form](https://github.com/momo200e/Ruby_Rails_Notes/blob/master/Gem_Notes.md#simple_form)**

`cd my_store`進入專案目錄後
```ruby
#Gemfile
gem 'bootstrap-sass'
gem 'simple_form'
``` 
終端機中執行`bundle install`

```ruby
#application.scss
@import "bootstrap-sprockets";
@import "bootstrap";
#application.js
//= require bootstrap
``` 
因為要搭配bootstrap使用，所以使用`rails generate simple_form:install --bootstrap`



### Step.3 scaffold建立product和coupon的CRUD

```rails
rails g scaffold product title price:integer description #建立書本管理
rails g scaffold coupon code discount:integer begin_at:datetime end_at:datetime #建立折價卷管理
```


### Step.4 表單驗證

我們的Produt的title和price要必填，price要大於0
        
Coupon的discount要必填，且大於0
```ruby
#Product.rb
validates :title, presence: true
validates :price, presence: true, numericality: {greater_than: 0}
```
```ruby
#Coupon.rb
validates :discount, presence: true, numericality: {greater_than: 0}
```
### Step.5 自動產生折扣代碼
當新增、修改時，折扣代碼未輸入的話，我們希望幫使用者產生一串代碼
```ruby
#Coupon.rb
before_save :gemerate_code

private
def generate_code
if self.code.empty?
  self.code = SecureRandom.hex[1..8].upcase
else
  self.code = code.upcase
end
```
我們要定義一個`generate_code`，判斷使用者有沒有輸入折扣代碼，若沒有則系統產生，其中`.upcase`用來將字串轉大寫，

### Step.6 商品完成折扣
首先，要先在商品加入一個`input`，讓使用者可以輸入折扣代碼
```ruby
#product/show.html.erb
<%= form_for @product do |f| %>

<% end %>
```

### Step.7 練習新增欄位
