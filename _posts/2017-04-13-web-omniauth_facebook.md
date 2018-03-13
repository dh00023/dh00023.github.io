---
title: OmniAuth Facebook Gem
categories:
- web
tag:
- gem
- facebook
- omniauth
- ruby
---

```
gem 'omniauth', '~> 1.4', '>= 1.4.1'
gem 'omniauth-facebook', '~> 4.0'
```

```
rails g migration AddUsernameToUser username
```

```
rails g migration AddConfirmableToDevise
```
```ruby
class AddConfirmableToDevise < ActiveRecord::Migration[5.0]
  def up
    add_column :users, :confirmation_token, :string
    add_column :users, :confirmed_at, :datetime
    add_column :users, :confirmation_sent_at, :datetime
    # add_column :users, :unconfirmed_email, :string # Only if using reconfirmable
    add_index :users, :confirmation_token, unique: true
    # User.reset_column_information # Need for some types of updates, but not for update_all.
    # To avoid a short time window between running the migration and updating all existing
    # users as confirmed, do the following
  end
  def down
    remove_columns :users, :confirmation_token, :confirmed_at, :confirmation_sent_at
    # remove_columns :users, :unconfirmed_email # Only if using reconfirmable
  end
end
```

```
rails g migration AddFieldsToUser provider uid image
```

- `username` : 페이스북 사용자명
- `provider` : 인증서비스 제공업체명(예, facebook, twitter 등)
- `uid` : 사용자 아이디
- `image` : 사용자 프로필 사진

[페이스북 개발자 페이지](https://developers.facebook.com)에서 `APP_ID`와 `APP_SECRET`발급을 받는다.

보안을 위해서 `figaro` gem 사용
```ruby
bundle exec figaro install
# => config/application.yml
```

```ruby
#config/initailize/devise.rb
config.omniauth :facebook, ENV["FACEBOOK_APP_ID"], ENV["FACEBOOK_SECRET_KEY"], scope: 'email', info_fields:'email'
```

```yml
#application.yml
FACEBOOK_APP_ID: ""
FACEBOOK_SECRET_KEY: ""
```

User모델에 다음과 같이 추가해준다.

```ruby
# user.rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable,
         :confirmable, :omniauthable, :omniauth_providers => [:facebook]

  validates :username, presence: true, length: {maximum: 50}

  has_many :rooms
  has_many :reservations
  has_many :reviews

  def self.from_omniauth(auth)
    user = User.where(email: auth.info.email).first

    if user
      return user
    else
      where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
        user.provider = auth.provider
        user.username = auth.info.email
        user.uid = auth.uid
        user.email = auth.info.email
        user.image = auth.info.image
        user.password = Devise.friendly_token[0,20]
        user.skip_confirmation!
      end
    end
  end
end
```

`omniauth_callbacks_controller.rb`생성한다.

```ruby
class OmniauthCallbacksController < Devise::OmniauthCallbacksController
  
  def facebook
    # You need to implement the method below in your model (e.g. app/models/user.rb)
    @user = User.from_omniauth(request.env["omniauth.auth"])

    if @user.persisted?
      sign_in_and_redirect @user, :event => :authentication #this will throw if @user is not activated
      set_flash_message(:notice, :success, :kind => "Facebook") if is_navigational_format?
    else
      session["devise.facebook_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end

  def failure
    redirect_to root_path
  end
end
```

라우팅 설정을 수정해준다.
```ruby
#routes.rb
devise_for :users, controllers: {:omniauth_callbacks => 'omniauth_callbacks', :registrations => 'registrations'}
```

마지막으로 페이스북 로그인할 수 있는 링크를 추가해준다.

```erb
#index.html.erb
<% if user_signed_in? %>
  <p><%= link_to "로그아웃", destroy_user_session_path, method: :delete, data: {confirm: "Are you sure?" }, class: 'btn btn-default'%></p>
<% else %>
  <p><%= link_to "로그인", new_user_session_path, class: 'btn btn-default' %></p>
  <p><%= link_to "페이스북으로 로그인", user_facebook_omniauth_authorize_path %></p>
<% end %>
```