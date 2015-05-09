---
layout: post
title: Automatic snake_case <=> lowerCamelCase with ActiveModel Serializer
class: automatic-casing-activemodel-serializer
description: So you can stick to the conventions in Ruby and JavaScript and have it just work.
---

## Setup

Add `active_model_serializers` to your `Gemfile`, run `bundle`, of course.

### Serializing

In `ApplicationController`:

{% highlight ruby %}
# Disable the root node, eg: {projects: [{..}, {..}]}
def default_serializer_options
  {root: false}
end
{% endhighlight %}

Then in `config/initializers/active_model_serializer.rb`:

{% highlight ruby %}
# Convert attributes from snake_case to lowerCamelCase
ActiveModel::Serializer.setup do |config|
  config.key_format = :lower_camel
end
{% endhighlight %}


### Deserializing

Again in `ApplicationController`:

{% highlight ruby %}
# Convert lowerCamelCase params to snake_case automatically
before_filter :deep_snake_case_params!
def deep_snake_case_params!(val = params)
  case val
  when Array
    val.map {|v| deep_snake_case_params! v }
  when Hash
    val.keys.each do |k, v = val[k]|
      val.delete k
      val[k.underscore] = deep_snake_case_params!(v)
    end
    val
  else
    val
  end
end
{% endhighlight %}

### And you're done!

Go ahead and write your serializers and notice that they will output
JSON like `{fullName: 'Brent Vatne'}`, and if you submit that same data
back to Rails, it will automatically convert it to `{full_name: 'Brent Vatne'}`.
