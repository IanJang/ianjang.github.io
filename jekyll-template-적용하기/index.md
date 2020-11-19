# Jekyll Template 적용하기


## 들어가며
  
  윈도우 환경에서 Jekyll을 사용해 보고 그 내용을 정리해 보았습니다.
  
---

## 마음에 드는 템플릿 찾기

우선 아래 사이트 중, 혹은 구글링을 통해 마음에 드는 jekyll 템플릿을 찾습니다.
- [http://jekyllthemes.org/](http://jekyllthemes.org/)
- [https://jekyllthemes.io/](https://jekyllthemes.io/)
- [https://github.com/jekyll/jekyll/wiki/Themes](https://github.com/jekyll/jekyll/wiki/Themes)

  
저는 lanyon-plus을 선택했습니다. 군더더기 없이 심플한 UI가 무척 맘에 들었습니다.

- lanyon-plus github repo: [https://github.com/dyndna/lanyon-plus](https://github.com/dyndna/lanyon-plus)

--- 

## 템플릿 다운로드

  아마도 대부분의 jekyll 템플릿이 github에 공유되어 있을 것입니다. 아래 명령으로 내려 받습니다.

```bash
$ git clone https://github.com/dyndna/lanyon-plus.git
```

## Package 설치

  템플릿을 다운받은 경로에서 jekyll을 구동해봅시다. 

```bash
$ jekyll serve
```

아마도 잘 되지 않을것이며, 아래와 같은 에러메시지를 마주할 것입니다.

```bash
C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/bundler-1.14.6/lib/bundler/spec_set.rb:87:in `block in materialize': Could not find i18n-0.7.0 in any of the sources (Bundler::GemNotFound)
        from C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/bundler-1.14.6/lib/bundler/spec_set.rb:80:in `map!'
        from C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/bundler-1.14.6/lib/bundler/spec_set.rb:80:in `materialize'
        from C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/bundler-1.14.6/lib/bundler/definition.rb:176:in `specs'
        from C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/bundler-1.14.6/lib/bundler/definition.rb:235:in `specs_for'
        from C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/bundler-1.14.6/lib/bundler/definition.rb:224:in `requested_specs'
        from C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/bundler-1.14.6/lib/bundler/runtime.rb:118:in `block in definition_method'
        from C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/bundler-1.14.6/lib/bundler/runtime.rb:19:in `setup'
        from C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/bundler-1.14.6/lib/bundler.rb:100:in `setup'
        from C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/jekyll-3.3.1/lib/jekyll/plugin_manager.rb:36:in `require_from_bundler'
        from C:/Ruby23-x64/lib/ruby/gems/2.3.0/gems/jekyll-3.3.1/exe/jekyll:9:in `<top (required)>'
        from C:/Ruby23-x64/bin/jekyll:22:in `load'
        from C:/Ruby23-x64/bin/jekyll:22:in `<main>'
```

jekyll은 ruby 기반으로 만들어졌고, 그렇기에 jekyll 템플릿들은 저마다 다양한 ruby package들을 활용하고 있습니다. 그 때문에 개발환경이 서로 다르게 되면 jekyll이 제대로 동작하지 않을 수 있습니다. ruby에는 bundler라는 것이 존재합니다. 이를 이용하여 문제를 해결할 수 있습니다.

---

## Bundler
  
  jekyll 템플릿이 설치된 폴더를 살펴보면 Gemfile, Gemfile.lock 찾을 수 있습니다. 이곳에는 해당 템플릿이 참고한 package 그 version의 spec 상세히 정리되어있고 한 번의 명령어 수행만으로 모두 설치할 수 있습니다.
bundler가 설치되어있지 않다면 이부터 설치합시다.

```bash
$ gem install bundler
```

  bundler가 설치되었다면 Gemfile이 존재하는 경로에서 아래 명령을 수행합시다.

```bash
$ bundle install
```

---

## jekyll 구동하기

  준비가 완료되었습니다. 아래 명령어로 jekyll을 구동해봅시다.

```bash
$ jekyll serve
```

---

# Reference
- [https://github.com/dyndna/lanyon-plus](https://github.com/dyndna/lanyon-plus)
- [http://guides.rubygems.org/rubygems-basics/](http://guides.rubygems.org/rubygems-basics/)
- [http://ruby-korea.github.io/bundler-site/](http://ruby-korea.github.io/bundler-site/)
- [http://bundler.io/v1.6/git.html](http://bundler.io/v1.6/git.html)

