---
layout: post
title: Gem mailgun-ruby
tags: [ruby-on-rails, gem, mailgun]
---

#### Mailgun 메일 보내기

CRUD로 기본 이메일 게시판 완성이 됐다는 가정하에 `mailgun`적용 해보겠습니다.

1. [Mailgun](https://www.mailgun.com/)가입
2. `mailgun-ruby` gem 설치 [[github  mailgun-ruby]](https://github.com/mailgun/mailgun-ruby)

```ruby
# post controller

require 'mailgun'
class PostsController < ApplicationController
  before_action :set_post, only: [:show, :edit, :update, :destroy]

  def index
    @posts = Post.all
  end

  def show
  end

  def new
    @post = Post.new  
  end

  def edit
  end

  def create
    @post = Post.new(post_params)

    respond_to do |format|
      if @post.save
        format.html { redirect_to @post, notice: 'Post was successfully created.' }
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
    # First, instantiate the Mailgun Client with your API key
      mg_client = Mailgun::Client.new ENV["api_key"]

      # Define your message parameters
      message_params =  { from: @post.from,
                        to:   @post.to,
                        subject: '연습용입니다.',
                        text:    @post.description
                      }

      # Send your message through the client
	  mg_client.send_message ENV["domain"], message_params
  end

  def update
    respond_to do |format|
      if @post.update(post_params)
        format.html { redirect_to @post, notice: 'Post was successfully updated.' }
        format.json { render :show, status: :ok, location: @post }
      else
        format.html { render :edit }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  def destroy
    @post.destroy
    respond_to do |format|
      format.html { redirect_to posts_url, notice: 'Post was successfully destroyed.' }
      format.json { head :no_content }
    end
  end

  private
    def set_post
      @post = Post.find(params[:id])
    end

    def post_params
      params.require(:post).permit(:to, :from, :description)
    end
end
```

이때 `Figaro`gem을 이용해서 api-key와 도메인을 입력한다.

나중에는 Mailgun페이지에서 도메인을 설정 후 이용하면된다.

<b style= "color: red;">Sandbox subdomains are for test purposes only. Please add your own domain or add the address to authorized recipients in Account Settings.</b> 즉, 서브도메인은 오직 테스트만 가능하다.
