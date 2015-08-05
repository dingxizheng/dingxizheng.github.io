---
title:  "How to Refresh $resource after Update Actions in AngularJS"
date:   2015-08-04 10:18:00
description: This artical shows how to refresh / invalidate ng-resource after being updated
tags: ["Angular", "JS"]
---

In angular $resource, if we want to cache results, we set `cache` as `true` in method definitions:
{% highlight javascript %}
$resource('/api/users/:userId', {
	userId: '@id'
}, {
	get: {
		cache: true, // set as true, the result will be cached
		method: 'GET'
	}
});
{% endhighlight %}

In many cases, we also want the changs that made to a resource should be visable to the updaters immediately:
{% highlight javascript %}
$resource('/api/users/:userId', {
	userId: '@id'
}, {
	get: {
		cache: true, // set as true, the result will be cached
		method: 'GET'
	},
	update: { // we want the resource to be refreshed after update
		method: 'POST'
	}
});
{% endhighlight %}
To meet the requirement metioned above, we can just simply pust an 'uncache' flag in the request method that you want the cache to be cleaned after the update request is responsed:
{% highlight javascript %}
$resource('/api/users/:userId', {
	userId: '@id'
}, {
	get: {
		cache: true, // set as true, the result will be cached
		method: 'GET'
	},
	update: { // we want the resource to be refreshed after update
		method: 'POST',
		params: {
			uncache: true // attach a 'uncache' flag to the request body 
		}
	}
});
{% endhighlight %}
To read the `uncache` flag from the response body, we should be define a general interceptor to process the `uncache` actions after responses:
{% highlight javascript %}
.factory('uncacheInterceptor', function($cacheFactory) {
	var cache = $cacheFactory.get('$http');

	return {
		response: function(response) {
			// if 'uncache' is set as true
			if (response.config.params && response.config.params.uncache) {
				// remove the 'cached' resource
				cache.remove(response.config.url);
			}
			return response;
		}
	};
})	
{% endhighlight %}	