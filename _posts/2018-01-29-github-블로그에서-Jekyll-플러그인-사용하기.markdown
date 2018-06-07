---
layout: post
title:  "GitHub 블로그에서 Jekyll 플러그인 사용하기"
date:   2018-01-29 21:08:55 +0900
author: 방구석엔지니어
categories: jekyll
tags:	jekyll github 블로그
cover:  "/assets/blog.jpg"
---

GitHub Pages 내부적으로 Jekyll을 사용하고 있고, 정적 페이지 생성에 있어서 Jekyll의 간편함때문에 많은 개발자들이 자신의 기술블로그로 GitHub Pages를 사용하고 있다. 하지만 GitHub에서는 보안상의 이유로 GitHub에서 기본으로 제공하고 있는 플러그인 이외의 사용자 플러그인을 지원하지 않고 있다. 그래도 외부 플러그인을 사용하는 방법이 다 있다. 그런데 ruby를 잘 모른다면 적용에 있어서 애를 먹을지도 모른다. 이 블로그에서도 [jekyll-archives][jekyll-archive] 라는 플러그인을 적용시키는데에 애를 먹었기 때문에, 한 번 정리해 두려고 한다.

# 기본세팅 #
[jekyllthemes.org][jekyllthemes]를 통해 원하는 테마를 선택하고, Fork까지 하는 과정은 생략하려 한다. 이와 관련된 내용은 다른 블로그를 참고해도 되고, 아주 간단하다. 내 GitHub 계정에 Fork하고, Setting에서 repository 이름을 `<내계정>.github.io `로 설정하면 자동으로 `https://<내계정>.github.io`로 호스팅 된다. 하지만 만약 Fork한 테마에 외부 플러그인이 적용된 부분이 있다면, 그 부분은 작동하지 않는다.

기본적인 Jekyll에 대한 지식이 부족한 상태에서는 [Jekyll 공식 페이지][jekyll]를 참고하자.


# 외부플러그인 적용하기 #
외부 플러그인을 적용하기 위해서는 로컬에서 페이지들을 build하고, build된 페이지가 생성된 폴더를 root로하여 repository의 `master` 브랜치에 업로드 해야 한다. 아래와 같은 절차를 따르자.


## Repository에 source 브랜치 생성 ##
Repo에 `source`라는 브랜치를 생성하자. `master` 브랜치에는 빌드 된 리소스들이 들어가야 하기 때문에, 현재 `master`에 존재하는 소스들이 복사될 브랜치이다.

<pre>
$ git checkout -b source master
$ git push -u origin source
</pre>

그리고나서 GitHub Setting 페이지에서 `source`브랜치를 default 브랜치로 바꿔주자. `Settings > Branches > Default branch`에 있다.


## Publishig 자동화 하기 ##
수동으로 빌드 된 페이지를 GitHub에 publishing하는 방법은 [여기][here]에 잘 설명 되어있다. 허나 너무 번거롭다. 자동화 하자.

로컬 Repo 루트에 `rakefile` 이라는 파일을 하나 생성한다. 그리고 파일 내용은 아래와 같이 구성한다.

{% highlight ruby %}
require "rubygems"
require "tmpdir"

require "bundler/setup"
require "jekyll"


# Change your GitHub reponame
GITHUB_REPONAME = "eunvanz/eunvanz.github.io"


desc "Generate blog files"
task :generate do
  Jekyll::Site.new(Jekyll.configuration({
    "source"      => ".",
    "destination" => "_site"
  })).process
end


desc "Generate and publish blog to gh-pages"
task :publish => [:generate] do
  Dir.mktmpdir do |tmp|
    cp_r "_site/.", tmp

    pwd = Dir.pwd
    Dir.chdir tmp

    system "git init"
    system "git add ."
    message = "Site updated at #{Time.now.utc}"
    system "git commit -m #{message.inspect}"
    system "git remote add origin https://github.com/#{GITHUB_REPONAME}.git"
    system "git push origin master --force"

    Dir.chdir pwd
  end
end
{% endhighlight %}

그리고 로컬 Repo Root 커맨드라인에서 `jekyll serve`로 로컬의`_site` 폴더에 빌드를 하고, `rake publish`를 입력하기만 하면 자동으로 GitHub Pages에 배포된다.

여러 방법을 찾아봤으나 이 방법이 가장 간단한 방법이다. 이후에는 로컬에서 소스를 관리하고, 빌드된 리소스만 커맨드입력을 통해 배포하면 된다. Ruby를 사용할 줄 몰라도 비교적 쉽게 따라할 수 있으니 헤매고 계신다면 이렇게 해보세요. 설명이 부족한 부분이 있다면 지적바랍니다.

[jekyll]: http://jekyllrb-ko.github.io/docs/home/
[jekyll-archive]: https://github.com/jekyll/jekyll-archives
[jekyllthemes]: http://jekyllthemes.org
[here]: http://gumpcha.github.io/blog/github-pages-with-jekyll-custom-plugin/