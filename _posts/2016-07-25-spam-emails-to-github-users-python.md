---
layout: post
title: "Sending spam emails to github users [python script]"
date: 2016-07-25
type: tech
excerpt: "simple python script to send spam emails to github users using github api"
tags: [python, api, github]
comments: true
---

Yesterday I received this spam e-mail. When I first saw the subject, I thought yeah finally my ultimately powerful github profile attracted an employer attention :'D . Then when I realized that it was just an advertise for a website, I thought what is going on? is github selling our info in such a cheap way? 
finally I remembered that I set this email as my public email for my github account. Simply they might use github api to retrieve github users info and send this email advertising their website.

<figure>
	<img src="../assets/img/gihub_spam_email.png">
</figure>

Then I was thinking: what about writing a simple scrapper imitating them, So I wrote this python script and soon I will do the same using node.js.


### GitHub API:

Github provides a nice API that I was hoping to play with it one day. What we need here is just the users API [developer.github.com/v3/users/](https://developer.github.com/v3/users/). We just need to send a HTTP GET request with to this url:

```
	GET http://api.gihub.com/users
```

Github will send back a JSON formated response that is a list of objects with this format:

```
	[
	  {
	    "login": "octocat",
	    "id": 1,
	    "avatar_url": "https://github.com/images/error/octocat_happy.gif",
	    "gravatar_id": "",
	    "url": "https://api.github.com/users/octocat",
	    "html_url": "https://github.com/octocat",
	    "followers_url": "https://api.github.com/users/octocat/followers",
	    "following_url": "https://api.github.com/users/octocat/following{/other_user}",
	    "gists_url": "https://api.github.com/users/octocat/gists{/gist_id}",
	    "starred_url": "https://api.github.com/users/octocat/starred{/owner}{/repo}",
	    "subscriptions_url": "https://api.github.com/users/octocat/subscriptions",
	    "organizations_url": "https://api.github.com/users/octocat/orgs",
	    "repos_url": "https://api.github.com/users/octocat/repos",
	    "events_url": "https://api.github.com/users/octocat/events{/privacy}",
	    "received_events_url": "https://api.github.com/users/octocat/received_events",
	    "type": "User",
	    "site_admin": false
	  }
	]
```

**Firstly,** this is not a full user object like the one we get when using the single user request described at the beginning of the the API documentation. It does not have the information we need, more specifically the public email address of the user.

So unfortunately we will need to make another request for each user in this list using `login` value with the single user request so that we can get his name and his public email.

**Secondly,** we notice that github will not send us all the users at once. Which is reasonable, this will be a huge json file which will consume both their resources and ours. So they will send us the response in patches and we can retrieve these patches using the `since` parameter which take as value the last user id you want to get its next patch.

**Thirdly,** github allow max 60 requests per hour for unauthenticated requests. Unfortunately, it will take a really long long long time to spam all github users using the same machine (requests with unauthenticated requests associated with ip address). However we can increase this rate by adding a github account to the request. But come on! I am a spammer now, so I want to be more anonymous. Let's look at the shiny part: we are not in hurry, we don't need to optimize the performance or launch threads, our script is still simple and small.

Let's write the code:

### GET request for retrieving the API:

I will use urllib2 library as it provides an ultimately easy interface for that (just one line).

So simply function that take last id we got to send it as a parameter, parse the JSON response and return it.

``` python
def get_all_users_api(last_id=0):
	response = urllib2.urlopen("https://api.github.com/users?since=" + str(last_id))
	if response.getcode() == 200: # success
		users_list = json.load(response)
		return users_list
```
And a similar function that makes the single user request and return the user object.

``` python
def get_single_user(username):
	if username: # not empty
		response = urllib2.urlopen("https://api.github.com/users/" + username)
		user = {}
		if response.getcode() == 200: # success
			user = json.load(response)
		return user
```

Done with API part, let's Iterate over the list and send our lovely spammy mails.

In our main loop, we will use `try and except` to catch the exception occurs when we reach our limit of requests per hour, so we will halt for 30 minutes then try again. Obliviously the script will work for a long time so we may stop the script and restart it many times, so we don't want to start every time from the very beginning id, so I will store and update a text file beside the script with the last id we retrieved and read from it when ever the script starts, it it does not exist or badly formated, we will assume that we will begin from the beginning.

Then let's use a gmail account for sending the email as explained [here](http://stackabuse.com/how-to-send-emails-with-gmail-using-python/).

So that is it, check the full code here: [github.com/HazemSamir/github_scrapper](https://github.com/HazemSamir/github_scrapper), simple and easy for a fun hour of coding. Feel free to try or modify it and notify me if some thing went wrong.

#### what to do next:

- Try to rewrite the code uisng node.js.
- Try to use the code on a network of machines and try to sync them.