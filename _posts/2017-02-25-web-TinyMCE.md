---
title: "TinyMCE 적용하기"
tag: web
---

TinyMCE 글을 보이는데로 수정할 수 있게 해주는 것이다.

1. `Gemfile`에 gem 추가
```
gem 'tinymce-rails'
```

2. `application.js` 수정
```js
//= require tinymce
//= require tinymce-jquery
```
둘 중 한개를 추가해준다.

3. 수정이 필요한 페이지에 추가해준다.
```erb
<%= tinymce_assets %>
```

4. text field를 수정해준다!
```erb
<%= f.text_area :content, :class => "tinymce", :rows => 40, :cols => 120 %>
```

5. `tinymce`helper
```erb
<%= tinymce %>
```

이렇게하면 작동한다! 하지만 새로고침을 해야만 뜨는 문제가 발생했는데 이건 `gem turbolinks`때문에 발생하는 것이므로 gem과 `application.js`에서 `//= require turbolinks`를 제거해준다.