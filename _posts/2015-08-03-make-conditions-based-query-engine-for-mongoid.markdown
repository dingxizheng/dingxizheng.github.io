---
title:  "How to Make General Condition-Based Query Engine For Mongoid Models"
date:   2015-08-03 10:18:00
description: Here, in this artical, i introduce how to make a simpale condition-based query engine which could be used to turn url-encoded query parameters to mongoid query statements
tags: ["Ruby", "Mongoid", "Rails"]
---

Building websites and json-based apis with rails is great. Rails provides wonderful tools to help build a fully functional websites by runing a few lines of commands. 

In this artical, we are going to make a simple query engine which helps get the model list returned with url query parameters. For example, by giving url like`example.com/users?age=>18&gender=male`, we expect the results should be male users who are older than 18. 

---
Before making it, we define some basic url query rules base on this schema `[field]=[vlaue]`:

* greater than: `field=>value`
* greater than or equal to: `field=>=value`
* less than: `field=<value`
* less than or equal to: `field=<=value`
* equal to: `field=value`
* not equal to: `field=!=value`
* not null: `field=!=null`
* value not in: `field=!=value1,,value2,,value3...`
* value in: `field=value1,,value2...`
* sortBy fields: `sortBy=fieldName1,,-fieldName` 

---
<h5>Implementation Example</h5>
{% highlight ruby %}
  # build mongoid query
  # 1. id=123,,345,,678  
  #       ===> id in [123, 345, 678]
  # 2. created=<=1023456 
  #       ===> created <= 1023456  
  # 3. filtering by fields' existence 
  #       ===> filed=!=null or filed=null
  # 4. to query item within a distance to a certain point 
  #       ===>  within=5 (within 5 kms)
  def query_by_conditions(scope, query_parameters)
    tempResult = scope
    sortBy = query_parameters[:sortBy]
    page = query_parameters[:page]
    per_page = query_parameters[:per_page]

    query_parameters.except!(*([:sortBy] + params_to_skip)).each do |key, value|
      field = key.to_sym

      logger.tagged('QUERY') { logger.info "key: #{key} , value: #{value}"}
      
      if value.nil?
        logger.tagged('QUERY') { logger.info "query value is empty!"}
      elsif value.start_with? '<='
        tempResult = tempResult.lte(field => value[2..-1])
      elsif value.start_with? '<'
        tempResult = tempResult.lt(field => value[1..-1])
      elsif value.start_with? '>='
        tempResult = tempResult.gte(field => value[2..-1])
      elsif value.start_with? '>'
        tempResult = tempResult.gt(field => value[1..-1])
      elsif value.start_with? '!='
        if value == "!=null"
          tempResult = tempResult.where(field.exists => false)
        else
          tempResult = tempResult.nin(field => value[2..-1].split(',,'))
        end
      else
        if value == "null"
          tempResult = tempResult.where(field.exists => true)
        else
          tempResult = tempResult.in(field => value.split(',,'))
        end
      end

    end

    # if 'sortBy' is contained in the query parameters
    if sortBy.present? 
      # multiple sortBy parameters must be seperated by ',,'
      # for instance: 'sortBy=time,,name' ==> means sortBy 'time' and 'name'
      order_by_params = sortBy.split(',,').map do |item|    
        logger.tagged('SORTBY') { logger.info item }  
        if item.start_with? '-'
          [item[1..-1].to_sym, -1]
        else
          [item.to_sym, 1]
        end
      end
      tempResult = tempResult.order_by(order_by_params)
    end

    # if pagenation is required, then return required page
    if page.present? and per_page.present?
      logger.tagged('PAGE') { logger.info "page: #{page} , number per page: #{per_page}" }
      return tempResult.page(page).per(per_page)
    else
      return tempResult.all
    end
  end
{% endhighlight %}