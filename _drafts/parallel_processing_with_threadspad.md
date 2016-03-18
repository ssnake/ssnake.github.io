---
layout: post
title: ThreadsPad is a tool for parallel processing
comments: true
---


Working on one commercial project I had to import and parse a xls file. Since the file had huge amount of data the parsing took time. To overcome this time-consuming process I had to split a parsing thread on a few ones. For this purpose I wrote a gem that allows to solve this task. [ThreadsPad](https://github.com/ssnake/threads_pad) allows to run a task in a few parallel threads. Once you started them you will get id of threads suite with this id you can control them and see logs. 

Check [demo app](https://tpd-demo.herokuapp.com/) that was inspired from my commercial project. 

<br>



## Installation

Add the gem to yout Gemfile:

{% highlight ruby %}
gem "threads_pad"
{% endhighlight %}

ThreadsPad uses two *threads_pad_jobs* and *threads_pad_job_logs* tables which should be generated and migrated: 

{% highlight console %}
rails generate threads_pad i
rake db:migrate
{% endhighlight %}

<br>

## Usage

 ThreadsPad works with classes descended from *ThreadsPad::Job* class. This class has a virtual *work* method that will do all work in a thread.
 
 For example, assume we need to calculate a sum. We will launch 3 threads


{% highlight ruby %}
class CalcWork < ThreadsPad::Job
	def initialize start, count
		@start = start
		@count = count
	end

	def work 
		sum= @start
		self.max = @count
		@count.times do 
			sum += 1
			self.current+=1
			debug "current #{self.current}"
			if terminated?
				debug 'terminated'
				break
			end
		end
		return sum
	end
end
{% endhighlight %}

*ThreadsPad::Job* has following attributes and methods you can control:

* \#min - minimal value of progress
* \#max - maximal value of progress
* \#current - current value of progress, between min and max
* \#terminated? - check whether job is terminated or not
* \#debug(msg) - log msg


Let's launch threads:
{% highlight ruby %}
    pad = ThreadsPad::Pad.new
    pad << CalcWork.new 1, 10000
    pad << CalcWork.new 10001, 10000
    pad << CalcWork.new 20001, 10000
    @job_id = pad.start

{% endhighlight %}


The *ThreadsPad::Pad* class has following methods:

* \#current - get a current position of the progress 
* \#done? - check if a process is finished/terminated or dead
* \#log - log a msg
* \#logs - get logs for a current job
* \#terminate - terminate a current job
* ::terminate - terminate all jobs which are in database
* \#destroy_all - remove from db all records that belongs to a current job. If a job is not finished yet, it will be marked as *destroy_on_finish*. Once it get finished it will destroy itself.
* \# start - starts jobs added to pad. It returns id number for this bundle of jobs.

<br>

##Getting a Status

Ok, we have launched threads in background, so how we can control them and get feeback. For referencing to bundle of threads we need to memorize *@job_id*, then we have to instantiate *ThreadsPad::Pad*. Here is example of coffee script:

{% highlight javascript %}
    <% @pad = ThreadsPad::Pad.new  @job_id %>
    $('#percents').html("<%= @pad.current%>")
    <% filter_job_logs(@pad.logs).each do |log|%>
      $('#logs').append("<%= "#{log[:id]}\t#{log[:msg]}\<br\>".html_safe %>")
    <%end%>

{% endhighlight %}

*filter_job_logs* is a view helper that prevents of flooding logs. It works with rails *Session* helper, so you ajax request might look as following:

{% highlight javascript %}
    $.ajax({
      url:"/main/status",
      dataType: 'script'
      settings: 
        beforeSend: (xhr)->
            xhr.setRequestHeader('X-CSRF-Token', $('meta[name="csrf-token"]').attr('content'))
    })
{% endhighlight %}

##Behind The Curtains

Every *ThreadsPad::Job* encapsulates *ThreadsPad::JobReflection* ActiveRecord class. It represents by *threads_pad_jobs* table:

* t.boolean "terminated"
* t.boolean "done"
* t.string  "result"
* t.integer "group_id"
* t.integer "integer"
* t.integer "max"
* t.integer "current"
* t.integer "min"
* t.boolean "started"
* t.boolean "destroy_on_finish"
* t.string  "thread_id"

<br>

Every time you call *ThreadsPad::Job* members (*max, min, current, terminated?*) it calls the same members in *ThreadsPad::JobReflection*. However, all calls go via "cache", so you end up with cached value. It counts how many times you call a member, by default once you called 100 times(all members share this 100 times) it would sync a cached value with db value. You can change this variable:


{% highlight ruby %}
    pad = ThreadsPad::Pad.new nil, iteration_sync: 10	
{% endhighlight %}