---
layout: post
title: Presenter Pattern
permalink: presenter.html
categories: presenter design-pattern
---

## Presenter Pattern

Hey guys, today I wanted to talk about something I encounter during my job. The presenter pattern.

## What is a presenter ?

The presenter is a pattern allowing you to format / organize your data. This is super helpful in case where you have plenty of variables that should be displayed depending on certain conditions. Here is a basic implementation of it :

__controller.rb__
{% highlight ruby %}
  def usage
    data = model.retrieve_data(params)
    render status: :ok, json: { Presenter::AnalyticsPresenter.new(data).present.to_json }
  end
{% endhighlight %}

and here is the code of the presenter itself :

__Presenter.rb__
{% highlight ruby %}
module Presenters
  class AnalyticsPresenter
    include AnalyticsHelpers

    attr_reader :data
    attr_reader :json

    def initialize(data)
      @data = data
      @json = {}
    end

    def present
      json['usage'] = @data['usage']
      json['analytics_v2'] = @data['analytics_v2'] if analytics_v2?
      json['click_analytics'] = @data['click_analytics'] if click_analytics?
      json
    end
  end
end
{% endhighlight %}

I believe the code is pretty explicit. Notice that I included analyticsHelpers which give me access to `analytics_v2?` and `click_analytics?`. I believe this pattern should be included in all rail application rather than put the condition logic to display the data or not in the model.



