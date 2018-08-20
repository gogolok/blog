---
title: "How to Use Activesupport Within Sinatra"
date: 2014-08-15T23:36:32+02:00
---

{{< highlight ruby >}}
# Gemfile
source 'https://rubygems.org'

gem 'sinatra', '1.4.5'
gem 'activesupport', '4.1.4'
{{< / highlight >}}

{{< highlight ruby >}}
# app.rb
require 'active_support/all'
require 'sinatra'

Time::DATE_FORMATS[:my_datetime] = "%Y-%m-%d Hour: %H Minute: %M Second: %S"

get '/' do
  Time.now.to_s(:my_datetime)
end
{{< / highlight >}}

[https://gist.github.com/gogolok/5877734](https://gist.github.com/gogolok/5877734)
