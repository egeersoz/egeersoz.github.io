---
layout: post
title:  "Auto-Synchronizing Meetups and Google Calendar"
date:   2015-05-16
categories: general
comments: True
---

## Introduction

Last week I wrote and deployed a Ruby script that synchronizes my Meetups with my Google Calendar. The code is [published on Github]((https://github.com/egeersoz/omni-cal)). It was a fun and informative experience, so I decided to write about it.

Meetup.com is a website where people create groups based on shared interests and then host events. You first join groups and then RSVP to events. Those you RSVP "yes" to appear on your Meetup calendar. I've been using the site a lot recently, and have been looking for a way to easily synchronize my meetups with my Google Calendar.

Meetup.com actually publishes an iCal feed of each user's events, and Google Calendar can subscribe to this feed. The iCal protocol has been around for a long time, and is excellent especially for non-technical users who need to automatically sync multiple calendars. The setup is very simple: you grab the link from your Meetup profile, tell Google Calendar to subscribe to it, and you are done.

Except you aren't. You see, I found that Google Calendar's support of iCal feeds to be spotty and unreliable. It takes a long time for events to be added to, updated on or removed from the calendar. I did some research and found out that this is a well-known problem. Some people say it can take anywhere from 8 to 24 hours for the synchronization to happen. Google is apparently aware of it, but so far they haven't done anything about it, probably because it's a free service. 

In any case, this wasn't going to cut it for me. I need the synchronizations to be much more frequent. Often times I RSVP for meetups on the same day and I need them to appear on my calendar right away so that I don't accidentally double-book. And sometimes an event's details (location, time, etc.) may change, or it may get cancelled altogether (for instance, a hike got cancelled today due to weather), and if I miss the email notifications then I at least need to see it update my calendar right away.

## Background and Disclaimers

Most of my Ruby experience is actually within the context of Rails. I've been meaning to get my hands dirty with a pure Ruby project, and this gave me the opportunity to do so. The project is still web-related, as in it sends GET requests and processes JSON responses, but it's not encumbered by a full-blown web framework. After all, it just needs to do one thing.

Also a disclaimer: this is basically version 0.1. The code works, but it is pretty messy, as I was primarily focusing on getting it to work. And there are no test cases... yet. I normally follow TDD et al, but only when working on things I'm familiar with. There were a lot of unknowns in this case, so it wasn't possible to write test cases describing how things should work. This will probably change moving forward, though.

## Tools and Strategy

At a high level, the code follows these steps:

1. Log into Google using OmniAuth and grab a list of calendar events for the next two months
2. Send a GET request to Meetup and grab a list of meetups for the next two months
3. Compare and contrast the lists and compile a batch of changes that need to be made
4. Send the batch to Google Calendar

We're using the following tools:

[Google API Ruby Client](https://github.com/google/google-api-ruby-client): This acts as a wrapper around the Google API. It also includes the Signet gem, which contains methods for OAuth2 (the second version of OmniAuth) that Google requires for most of their services.

[Meetup API](http://www.meetup.com/meetup_api/): This is a RESTful HTTP interface that allows us to send requests to various end-points at Meetup.com and receive data in response.

Without further delay, let's examine the code step-by-step!

## The Code

At this point I'll assume that you are viewing the code on Github and will follow along. You don't have to of course, but if you're trying to create something similar, looking at the whole thing would be better than just reading the snippets I'm going to post below.

As I mentioned previously, this is just version 0.1, so everything is contained inside one long function called `sysc`. Here's how it starts:

{% highlight ruby %}
client = Google::APIClient.new(application_name: "Calendar Sync", application_version: "0.0.1")
batch = Google::APIClient::BatchRequest.new
private_key = ENV['GOOGLE_CALENDAR_KEY']
key = OpenSSL::PKey::RSA.new private_key, 'notasecret'
client.authorization = Signet::OAuth2::Client.new(
	                    :token_credential_uri => 'https://accounts.google.com/o/oauth2/token',
	                    :audience             => 'https://accounts.google.com/o/oauth2/token',
	                    :scope                => 'https://www.googleapis.com/auth/calendar',
	                    :issuer               => ENV['GOOGLE_SERVICE_ACCOUNT'],
	                    :signing_key          => key )
client.authorization.fetch_access_token!
{% endhighlight %}

We start by initializing a new APIClient object. This takes two paremeters, the name of our application and its version. As far as I could tell, these are completely arbitrary and they don't have any impact on the actual operation of our script.

On the second line, we create a new BatchRequest object. The reason we're doing this is because we'll need to tell Google Calendar to perform a bunch of actions (add this event, update this event, delete this other event, etc.), and instead of sending these as individual HTTP requests, we'll send them as one request, which is more efficient.

The rest of the code snippet deals with authentication for the Google API. I'm not going to go into too much detail here because it could actually be an article of its own, but essentially, we're telling Google who we are (the `:issuer` option) and then using a private key to sign the request, which tells Google we are who we say we are.

Let's examine the next step:

{% highlight ruby %}
api = client.discovered_api('calendar', 'v3')
google_events = client.execute(:api_method => api.events.list,
                               :parameters => { 'calendarId'   => ENV['GOOGLE_CALENDAR_ID'],
                                                'timeMin'      => DateTime.now,
                                                'timeMax'      => 2.months.from_now.to_datetime } ).data['items'].to_a
time_range = "#{Time.now.to_datetime.to_i * 1000},#{2.months.from_now.to_datetime.to_i * 1000}"
meetup_url = "https://api.meetup.com/2/events?&sign=true&key=#{ENV['MEETUP_API_KEY']}&photo-host=public&rsvp=yes&member_id=#{ENV['MEETUP_MEMBER_ID']}&time=#{time_range}"
meetup_events = JSON.parse(URI.parse(meetup_url).read)['results']
{% endhighlight %}

We start this snippet by telling the API client which part of the Google API we'll be using. Remember that Google has a ton of services (Gmail, Google Drive, Adwords, etc.) so we simply specify that we'll use version 3 of the Calendar API. This is often referred to as "API discovery": the available methods are discovered during execution.

Out of those discovered api methods, we're calling `events.list` on the next line. As you probably guessed, this method returns a list of calendar events. We also supply the `timeMin` and `timeMax` parameters, with the former being the current time (in datetime format) and the latter being 2 months from now. Essentially, a date range. That's just my own preference: I rarely RSVP to events that are further out, and even if I do, I figure there would be little value in having them appear on my calendar right away.

The events are hashes nested inside the `items` key of the JSON-formatted response, so we take them and convert them to an array (that's what `client.execute(stuff).data['items'].to_a` is doing). If that sounds confusing, don't worry. What you need to know is that we'll end up with an array of hashes, like this:

{% highlight bash %}
[{"name" => "Hike at the Park", "location" => "2341 Zilker Ave"}, 
{"name" => "Zoe's Birthday", "location" => "12893 S Congress Ave"}, 
{"name" => "Hanging out with Angell", "location" => "7329 Sasquatch Dr"}]
{% endhighlight %}

This is a simplified version of course (the actual event data contains a ton more detail) but you get the idea.

Next, we need to compare and contrast the lists. If an event exists on the meetup list but not on the calendar, that means I recently RSVP'd to it, so it needs to get added to the calendar. In the opposite scenario, it needs to get removed. And if it exists in both places, the details need to get updated (in case the event organizer changed the time, place, or other details). Since this is the first version, I didn't want to deal with actually checking whether these details have changed. I simply assume that they did and update them using `events.update`.

Here's the fun part about the comparison though: in order to check if an event on one list exists in the other, there needs to be a way to uniquely identify them. I considered several options. Names obviously can't be used, since two events can have the same name (especially if they are recurring, e.g. "Monthly River Walk"). Can the name and the time be used in conjunction? After all, a "Monthly River Walk" that happens on June 5th, 2015 at 9am is going to be pretty unique. The problem with that approach is that both the name and the time are prone to change, so they can't be used to uniquely identify the event either.

After doing some research, I noticed that Google Calendar events have an option called **ExtendedProperties**, which allow setting custom properties as key-value pairs. And since each meetup event has a unique id number, I'm simply assigning them to the Google Event when I add it:

{% highlight ruby %}
body = { 'summary'            => meetup_event["name"],
         'description'        => meetup_event["description"],
         'location'           => meetup_event["venue"]["address_1"],
         'start'              => { 'dateTime' => Time.at(meetup_event["time"].to_i / 1000).to_datetime.to_s },
         'end'                => { 'dateTime' => Time.at((meetup_event["time"].to_i / 1000) + (meetup_event["duration"].to_i / 1000)).to_datetime.to_s},
         'extendedProperties' => { 'private'  => { 'unique_id' => meetup_event["id"] } }
       }
{% endhighlight %}

Note the last key-value pair in the body hash: it is used to set the unique id, which is then used in subsequent executions of the script to check if an event is currently on Google Calendar:

{% highlight ruby %}
if !google_events.any? {|g| g.extended_properties.private["unique_id"] == meetup_event["id"] }
  # add it
end
{% endhighlight %}

There's two lists we're dealing with: `meetup_events` and `google_events`. The script iterates through both, first to add the events in `meetup_events` to the calendar, and then to check if there are any events on the calendar that need to be removed (i.e. if they no exist in `meetup_events` because I changed my RSVP to "no"). In the end, the batch gets executed and the `response.status` is printed to the console.

I ended up deploying this script to Heroku along with the Heroku Scheduler add-on, which is configured to run it every 10 minutes. Much better to sync calendars frequently like that, rather than every 8-24 hours like Google does with iCal, wouldn't you say? :)