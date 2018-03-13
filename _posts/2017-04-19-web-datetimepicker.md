---
title: Datetime picker 사용하기
categories:
- web
tag: 
- datetime
- ruby
- js
- gem
---

### 한국 시간대로 설정하기
rails는 기본 시간대가 `utc`로 되어있다. timezone을 한국으로 바꾸기 위해서는 `config/application.rb`파일을 아래와 같이 수정한다.
```ruby
require_relative 'boot'

require 'rails/all'

# Require the gems listed in Gemfile, including any gems
# you've limited to :test, :development, or :production.
Bundler.require(*Rails.groups)

module Datetime
  class Application < Rails::Application
    # Settings in config/environments/* take precedence over those specified here.
    # Application configuration should go into files in config/initializers
    # -- all .rb files in that directory are automatically loaded.
    config.time_zone = 'Seoul'
  end
end
```

#### 시간 선택 jQuery

`gem bootstrap3-datetimepicker`을 사용해서 시간을 선택하고, 날짜를 고를 수 있도록 할것이다.
이때! css가 제대로 적용되지 않았는데 그런 경우에는 `gem bootstrap-sass`을 설치해 주면된다.

```erb
<div style="overflow:visible!important;">
    <div class="form-group">
        <div class="row">
            <div class="col-md-8">
                <div id="datetimepicker12">
                <div class="field">
                  <%= f.label :booking %>
                  <%= f.text_field :booking ,id: 'datetimepicker12'%>
                </div>
          
                </div>
            </div>
        </div>
    </div>
    <script type="text/javascript">
		$(function () {
            $('#datetimepicker12').datetimepicker({
                inline: true,
                sideBySide: true,
                 format: 'LL HH',
                 minDate: 'now',
                 maxDate: moment().add(7,'days')
            });
        });
    </script>
```
moment()를 써서 maxDate를 제한 할 수 있다는 것을 알았다. 지정된 포멧을 지키면 rails console로도 시간을 쉽게 timestamp로 바꿀수 있다. `.to_time.to_i`

위의 스트립트를 이제 요일별로 제한일자를 다른 js를 만들었다.

```js
var a=moment()._d.getDay();

if (a===0) {
	restric_sun();
}
else{
	restric_date();
}

function restric_date(){
    $(function () {
	    $('#datetimepicker12').datetimepicker({
	        inline: true,
	        sideBySide: true,
	        keepOpen: true,
	    	format: 'MM / DD , ddd , HH',
	        minDate: 'now',
	        maxDate: moment().endOf('week').add(1,'day'),
	        disabledHours: [0,1,2,3,4,5,6,7,21,22,23]
	    });
	});
}
function restric_sun(){
    $(function () {
	    $('#datetimepicker12').datetimepicker({
	        inline: true,
	        sideBySide: true,
	        keepOpen: true,
	    	format: 'MM / DD , ddd , HH',
	        minDate: moment().startOf('day'),
	        maxDate: moment().endOf('day'),
	        disabledHours: [0,1,2,3,4,5,6,7,21,22,23]
	    });
	});
}
```

#### radio_button으로 선택 후 (1시간, 2시간을 선택하면) 추가저장

```ruby
#controller
def create
    @post = Post.new(post_params)

    respond_to do |format|
      if @post.save
        if @post.choose.to_i==2
          Post.create!(name: @post.name ,date: @post.date, time: @post.time.to_time+@post.choose.to_i.hours)  
        end
        format.html { redirect_to @post, notice: 'Post was successfully created.' }
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end
```

#### datetimepicker css

```erb
<style type="text/css">
.ui-datepicker {
   background: #333;
   border: 1px solid #555;
   color: #EEE;
 }
</style>
```

이런식으로 직접 수정할 수 있다.

#### 한글변환
`datetimepicker`에서 한글변환이 필요하다. 이때 `moment.js`가 필요하다.
그래서 `gem momentjs-rails`을 이용해서 환경 셋팅을 할 것이다.

```ruby
# Gemfile
gem 'momentjs-rails', '~> 2.17', '>= 2.17.1'
```

```js
#application.js
//= require moment
//= require moment/ko.js
```

여기서 중요한 것은 local을 하고 싶은 곳을 **`moment/ko.js`**로 하는 것이다.
그다음에 datepicker함수에`locale: 'ko'`를 추가해주면된다.