---
title: "Gem nokogiri"
tag: web
---

#### 참고페이지
- [http://www.nokogiri.org/](http://www.nokogiri.org/tutorials/searching_a_xml_html_document.html)
- [상황에 따른 적절한 gem 선택하기](https://stackoverflow.com/questions/4981379/what-are-some-good-ruby-based-web-crawlers)

### Gem 설치

```
$ gem install nokogiri
```

### Parsing an HTML/XML Document(From the Internets)
```ruby
require 'nokogiri'
require 'open-uri'

doc = Nokogiri::HTML(open("http://www.threescompany.com/"))
667530.downto(667525) do |p|
	# `#{}`를 쓰면 루비 문법을 넣을 수 있다.
	doc = Nokogiri::HTML(open("http://www.slrclub.com/bbs/zboard.php?id=free&page=#{p}"))
	# css형태로 파싱 .은 class #은 id //a는 a태그만 뽑고싶다
	doc.css(".sbj//a").each do |x|
		#a태그 내의 텍스트만 뽑고싶다. include?를 사용해서 필터링 할 수 있다.
		puts x.inner_text if x.inner_text.include?("류")
	end
end
```

크롬개발자도구를 이용해서 내가 파싱하고 싶은 데이터의 `class`를 긁어와야한다.

`gem whenever`을 이용하면 정해놓은 시간대로 자동으로 해준다. 파싱할때 자주사용된다.
레일즈 프로젝트에서는 일반적으로 모델파일에 저장한다.


