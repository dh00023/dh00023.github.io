---
layout:     post
title:      Gem searchkick
date:       2017-05-24 15:25:00
summary:    searchkick gem 적용하기
tags: RoR gem
---

```gem
gem 'searchkick'
```

```vim
$ sudo apt-get update
$ java -version
$ wget https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.7.2.deb
$ sudo dpkg -i elasticsearch-1.7.2.deb
$ sudo update-rc.d elasticsearch defaults
$ sudo service elasticsearch start
$ bundle exec rake searchkick:reindex:all
```

```ruby
# routes.rb
resources :posts do
  collection do
    get 'search'
  end
end
```

```ruby
# posts_controller.rb
def search
  if params[:search].present?
    @posts = Post.search(params[:search])
  else
    @posts = Post.all
  end
end
```

```erb
#_form.html.erb
<%= form_tag search_posts_path, method: :get, role: "search" do %>
      <p>
        <%= text_field_tag :search, params[:search], class: "form-control" %>
        <%= submit_tag "Search", name: nil, class: "btn btn-default"%>
      </p>
<% end %>
```

```erb
# search.html.erb
<div class="row">
  <% @posts.each do |p| %>
    <div class="col-sm-6 col-md-4">
      <div class="thumbnails">
        <%= p.name %>
      </div>
    </div>
  <% end %>
</div>
```

[ubuntu elasticsearch 설치 참조](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-elasticsearch-on-ubuntu-14-04)