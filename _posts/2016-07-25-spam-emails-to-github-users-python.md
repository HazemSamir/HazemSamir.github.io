---
layout: post
title: "Sending spam emails to github users [python script]"
date: 2016-07-25
---

Yesterday I received this e-mail. When I first saw the subject, I thought yay! finally my ultimately powerful github profile attracted an employer attention :'D . Then, I realized that it was just an scam ad for a website. What is going on? Is github selling our info in such a cheap way? Finally I remembered that I set this email as my public email for my github account. Simply, they might use github API to retrieve github users info and send this email advertising their website to them.

<figure>
	<img src="/data/images/gihub_spam_email.png">
</figure>

Then I was thinking: what about writing a simple API scrapper like theirs? So I wrote this python script and hopefully I will do the same using node.js soon.


### GitHub API:

Github provides a nice API that I was always looking to playing with one day. We only need the users API here [developer.github.com/v3/users/](https://developer.github.com/v3/users/). We all we need to do is to send a simple HTTP GET request to this url:

```
GET http://api.gihub.com/users
```

Github will send back a JSON formated response containing a list of users of this format:

```javascript
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

**Firstly,** this is not a full user object like the one we get when using the single user request described at the beginning of the the API documentation. It does not have the information we need, specifically the public email address of the user.

So unfortunately we will need to make another request for each user in this list using `login` value with the single user request so that we can get his name and his public email.

**Secondly,** we notice that github will not send us all the users at once. Which is reasonable, all users would be a huge json file that will consume both their resources and ours. So they will send us the response in patches and we can retrieve these patches using the `since` parameter which takes the last user id you want to get its next patch.

**Thirdly,** github allows max rate of 60 requests per hour for unauthenticated requests. Unfortunately, it will take a really long long long time to spam all github users using the same machine (requests with unauthenticated requests are associated with the IP address). However we can increase this rate by adding a github account to the request. Come on! I am a spammer, I want to be anonymous. However, we are not in a hurry, we don't need to optimize the performance or launch threads. We will keep our script simple and small.

Let's write the code:

### GET request for retrieving the API:

I will use `urllib2` library as it provides an ultimately easy interface for sending and parsing HTTP requests and responses (just one function call).

Let's start with a simple function that takes last id we had, sends a request to the API, parses the JSON response and returns it.

``` python
def get_all_users_api(last_id=0):
	response = urllib2.urlopen("https://api.github.com/users?since=" + str(last_id))
	if response.getcode() == 200: # success
		users_list = json.load(response)
		return users_list
```
Then a similar function that sends a single user request and returns the user object.

``` python
def get_single_user(username):
	if username: # not empty
		response = urllib2.urlopen("https://api.github.com/users/" + username)
		user = {}
		if response.getcode() == 200: # success
			user = json.load(response)
		return user
```

Done with API part, let's iterate over the list and send our lovely spammy mails.

In our main loop, we will use `try and except` to catch the exceptions occur when we reach our limit of requests per hour, then we will halt for 30 minutes then try again. Obliviously the script will work for a long time so we may stop the script and restart it many times and we don't want to start every time from the very beginning id, so I will store and update a text file in the same directory containing the last id we retrieved and read from it when ever the script starts, if it does not exist, we will assume that we will begin from the beginning.

Then let's use a gmail account for sending the email as explained [here](http://stackabuse.com/how-to-send-emails-with-gmail-using-python/).

That's it, you can check the full code here: [github.com/HazemSamir/github_scrapper](https://github.com/HazemSamir/github_scrapper), a fun hour of coding. Feel free to play with it.

#### what to do next:

- Try to rewrite the code uisng node.js.
- Try to use the code on a network of machines and try to sync them.
