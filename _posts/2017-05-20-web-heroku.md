---
layout: post
title: Heroku 배포하기
tags: [ruby-on-rails, heroku, deploy]
---

### 참조페이지
- [What is heroku?](https://www.heroku.com/what)
- [공식문서](https://devcenter.heroku.com/articles/getting-started-with-rails4)

#### Gemfile 수정하기

현재 Gemfile(가독성을 위해서 주석을 제거했습니다.)

```
source 'https://rubygems.org'

git_source(:github) do |repo_name|
  repo_name = "#{repo_name}/#{repo_name}" unless repo_name.include?("/")
  "https://github.com/#{repo_name}.git"
end

gem 'rails', '~> 5.0.2'
gem 'sqlite3'
gem 'puma', '~> 3.0'
gem 'sass-rails', '~> 5.0'
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.2'

gem 'jquery-rails'
gem 'turbolinks', '~> 5'
gem 'jbuilder', '~> 2.5'

group :development, :test do
  gem 'byebug', platform: :mri
end

group :development do
  gem 'web-console', '>= 3.3.0'
  gem 'listen', '~> 3.0.5'
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'
end

gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
gem 'bootstrap', '~> 4.0.0.alpha6'
```

- `gem 'sqlite3'`를 `group :development, :test`로 옮긴다.
- `group :production`생성
- `group :production`에 `gem 'pg'`와 gem `rails_12factor`를 추가

```
source 'https://rubygems.org'

git_source(:github) do |repo_name|
  repo_name = "#{repo_name}/#{repo_name}" unless repo_name.include?("/")
  "https://github.com/#{repo_name}.git"
end

gem 'rails', '~> 5.0.2'
gem 'puma', '~> 3.0'
gem 'sass-rails', '~> 5.0'
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.2'

gem 'jquery-rails'
gem 'turbolinks', '~> 5'
gem 'jbuilder', '~> 2.5'

group :development, :test do
  gem 'byebug', platform: :mri
  gem 'sqlite3'
end

group :development do
  gem 'web-console', '>= 3.3.0'
  gem 'listen', '~> 3.0.5'
  gem 'spring'
  gem 'spring-watcher-listen', '~> 2.0.0'
end

group :production do
  gem 'pg'
  gem 'rails_12factor'
end

gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
gem 'bootstrap', '~> 4.0.0.alpha6'
```

이때 Mac에서는 Error가 날 수 있다.

1. `brew update`
2. `brew uninstall postgresql`
3. `sudo chown -R $(whoami) /usr/local`
4. `brew install postgresql`
5. `bundle update`

#### Heroku 사용하기

1. [https://www.heroku.com/](https://www.heroku.com/) 가입
2. [https://dashboard.heroku.com/account/billing](https://dashboard.heroku.com/account/billing)에서 카드 등록((등록하면 1개의 app을 1달 내내 무료로 사용 가능하다.)

### Heroku 설치

#### Mac OS

1. `brew install heroku-toolbelt`
2. `heroku update`

#### C9

1. `sudo apt-get install heroku`

#### 공통
1. `$ heroku login` (가입한 e-mail 과 password 입력)
2. `heroku create [app_name]` ( [app_name].herokuapp.com 이 [root_url] 이 된다. )

#### Heroku 에 배포하기.

1. `$ git push heroku master` ( 수정사항이 있다면 git commit 이후에)
2. `$ heroku run rake db:migrate` (DB 를 사용한다면. 현재 단계에서는 필요 없음)


### 이미지 업로드

헤로쿠에 배포할때 이미지가 제대로 안뜨는 경우가 있다. 이때, 이미지 경로를 제대로 설정해줘야한다.

```ruby
# config/environments/production.rb
config.serve_static_assets = true
config.assets.compile = true
```
를 추가해준다.

이미지 경로는 `img_path`로 수정해준다.

```xml
<img class="card-img" src="<%= image_path(@chooseimg) %>">
```

`$ rake assets:precompile` 을 해준다.
