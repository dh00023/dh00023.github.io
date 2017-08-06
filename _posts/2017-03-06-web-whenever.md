---
title: "Whenever Gem"
tag: web
---

`$gem whenever` 을 이용해서 DB의 records를 자동으로 삭제되게 설정할 것이다.

```
#Gemfile
gem 'whenever', '~> 0.9.7'
```
```
$ bundle install
```

```
$ wheneverize .
```
를 하면 자동으로 `config/schedule.rb`가 생성된다.

```
whenever --update-crontab
```
로 환경을 일단 자동 설정해둔다.

자동으로 실행될 tasks파일을 생성한다.
```
$ rails g task reservations delete_every_monday
```
```ruby
# vi lib/tasks/reservations.rake
namespace :reservations do
  desc "TODO"
  task delete_every_monday: :environment do
        Reservation.delete_all
  end
end
```
제대로 실행이되는지 확인해본다.
```
$ rake reservations:delete_every_monday
```
```ruby
#config/schedule.rb
every :monday, :at => '12:00am' do
  rake "reservations:delete_every_monday", environment: 'production'
end
```
```
$ whenever -w
# [write] crontab file written
$ whenever -i
[write] crontab file updated
```
하면 자동으로 실행되는 것을 확인 할 수 있다!