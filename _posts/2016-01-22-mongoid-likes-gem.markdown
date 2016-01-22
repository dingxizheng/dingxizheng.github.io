---
title:  "mongoid likes gem"
date:   2016-01-22 10:18:00
description: This Gem extends your Mongoid user model to have like and dislike abilities to any likeable models
tags: ["Ruby", "Rails", "Gem", "Mongoid", "Likeable"]
---

Please refer https://github.com/dingxizheng/mongoid_likes

This Gem extends your Mongoid user model to have like and dislike abilities 
to any likeable models

## How to install

Install the gem

    $ gem install mongoid-likes

or add the gem to your `Gemfile`

    gem 'mongoid-likes'


## How to use

Include the `Mongoid::Likeable` module into the models you want to like 

Include the `Mongoid::Liker` module into the user model

{% highlight ruby %}
class User
  include Mongoid::Document
  include Mongoid::Liker

end
{% endhighlight %}


{% highlight ruby %}
class Post
  include Mongoid::Document
  include Mongoid::Likeable

end
{% endhighlight %}

Now you can `like` or `dislike` your `posts`.

{% highlight ruby %}
@user.like @post # user likes a post

@user.unlike @post # user unlikes a post liked before

@user.dislike @post # user dislikes a post

@user.undislike @post # user undislike a post disliked before
{% endhighlight %}

Note that a likeable resource can not be liked and dislied by the same user at the same time

{% highlight ruby %}
@user.like @post_one #=> user likes a post

@user.dislike @post_one #=> will return false
{% endhighlight %}

{% highlight ruby %}
@user.dislike @post_one #=> user dislikes a post

@user.like @post_one #=> will return false
{% endhighlight %}

To get how many likes or dislikes a resource have 

{% highlight ruby %}
@post.likes #=> number of likes current post have
@post.dislikes #=> number of dislikes current post have
{% endhighlight %}

To check if user liked of disliked a resource

{% highlight ruby %}
@post.liked_by? @user

@user.liked? @post

@post.disliked_by? @user 

@user.disliked? @user
{% endhighlight %}

To get a resource's `likers` and `dislikers`

{% highlight ruby %}
@post.likers #=> results in Mongoid::Criteria
@post.lkiers.where(....)

@post.dislikers #=> results in Mongoid::Criteria
@post.dislkiers.where(....)
{% endhighlight %}

To get specified `type` of objects user liked or disliked

{% highlight ruby %}
@user.all_liked :post #=> results in Mongoid::Criteria

@user.all_liked :blabla #=> return nil, since model 'blabla' does not exist

@user.all_disliked :post #=> results in Mongoid::Criteria
{% endhighlight %}
