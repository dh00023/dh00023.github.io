---
layout: post
title: Chartkick
tags: [ruby-on-rails, gem, js]
---

- [Github page](https://github.com/ankane/chartkick)

```
#Gemfile
gem 'chartkick', '~> 2.2', '>= 2.2.4'
```
```
$ bundle install
```

차트를 그릴때 평균이나 날짜별로 그려야 한다면 [Groupdate](https://github.com/ankane/groupdate), [ActiveMedian](https://github.com/ankane/active_median)을 같이 사용하면된다.

```erb
# application.html.erb
<%= javascript_include_tag "https://www.gstatic.com/charts/loader.js" %>
```
```js
//applicatoin.js
//= require chartkick
```
여기까지가 `chartkick`사용 준비가 다된것이다.

```ruby
class HomesController < ApplicationController
  def show
  	@posts = Post.all
  end
end
```

```erb
  <!-- Line Chart - Single Series -->
  <div>
    <h3>성감대선택수</h3>
    <%= line_chart @posts.group('body').count, discrete: true , width: "800px", height: "500px"%>
    <%= pie_chart @posts.group(:body).count ,colors: ["#b00", "#666"]%>
    <%= column_chart @posts.group(:body).count, stacked: true %>
    <%= bar_chart @posts.group(:body).count %>
  </div>
```

필요한 차트의 구성에 따라 쉽게 변경할 수 있다.
