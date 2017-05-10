# snooper [![Build Status](https://travis-ci.org/JuicyPasta/snooper.svg?branch=master)](https://travis-ci.org/JuicyPasta/Snooper)

[![Join the chat at https://gitter.im/reddit-snooper-nodjs/Lobby](https://badges.gitter.im/reddit-snooper-nodjs/Lobby.svg)](https://gitter.im/reddit-snooper-nodjs/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Reddit bot framework that allows you to easily create high performance Reddit bots with nodejs. 

## Installing snooper
``` bash
npm install reddit-snooper --save
```

## Library usage
``` js
var Snooper = require('reddit-snooper')
    snooper = new Snooper(
        {
            // credential information is not needed for snooper.watcher
            username: 'reddit_username',
            password: 'reddit password',
            app_id: 'reddit api app id',
            api_secret: 'reddit api secret',
            user_agent: 'OPTIONAL user agent for your bot',

            automatic_retries: true, // automatically handles condition when reddit says 'you are doing this too much'
            api_requests_per_minuite: 60 // api requests will be spread out in order to play nicely with Reddit
        })
```

## Reddit Watcher

#### unexpected factorial bot
``` js
snooper.watcher.getCommentWatcher("all")
.on("comment", function (comment) {

    // only reply if the comment contains a factorial
    let match = comment.data.body.match("[0-9]+!")
    if (match) {

        // post a reply, if you have automatic_retries set to true this comment will make
        // it through even if it takes 10+ minutes ('you are doing this too much!') otherwise
        // an error will be thrown
        snooper.api.post("/api/comment", {
            api_type: "json",
            text:     match[0] + " = " + factorial(+match[0].substring(0, match[0].length -1)),
            thing_id: comment.data.name
        }, function (err, statusCode, data) {
            if (!err) console.log('just replied to comment: ' + comment.data.name)
        })
    }
})
.on("error", console.error)

// when you are done 
commentWatcher.close()
```

#### download all pics that are posted to r/pics
``` js
// start watching for new posts to r/pics
snooper.watcher.getPostWatcher('pics')
.on('post', function(post) { // post will be a json object containing all post information

    // check if its a link post (t3, definitions found on reddit api page)
    // also make sure there is an image link
    let urlmatch = post.data.url.match('\.([a-zA-Z]+$)')
    if (post.kind === 't3' && urlmatch) {
        request(post.data.url).pipe(fs.createWriteStream("gifs/"+post.data.title+urlmatch[0]))
    }

})
.on('error', console.error)
```


## Reddit API

Snoopers api component at the moment is an agnostic wrapper around reddit's rest API that handles retries, rate limit throttling and Reddit's different response codes.

Keep in mind that new accounts get little to no posting privileges (1 comment or post per 5 minutes or more) if you dont have any karma. If you just want to play around with the api I recommend using an active account. 

[Reddit API Documentation](https://www.reddit.com/dev/api/)

#### basic api usage
``` js
// check how much karma your bot has
snooper.api.get('api/v1/me/karma', {}, function(err, statusCode, data) {
    console.log("I have " + data.karma + " karma")
})

// I highly doubt your bot is over 18 years of age
snooper.api.patch('/api/v1/me/prefs/', {
    over_18: false
}, function(err, statusCode, data) {

})

// activate turbo mode doubling performance       (kidding)
snooper.api.post('api/v1/gold/give', {
    months: 1,
    username: 'juicypasta'
}, function(err, statusCode, data) {

})

```

## API setup 
All you need to do to get up and running is obtain an api_id and an api_secret. Both can be obtained from [Reddit app console](https://reddit.com/prefs/apps)
1. Create (or log into) a reddit account
2. Navigate to the [authorized applications console](https://reddit.com/prefs/apps)
3. Select 'create another app...' at the bottom
4. Fill in the name, description and click on 'script', put in anything for the redirect uri, its not needed and you can change it later if you want to
5. Copy down the 'secret' that is your api_secret, the 14 character string by the name of your app is your app_id
6. Use these values and your credentials to configure the library


## Coming Soon
- Support for multiple accounts
- Convenience methods for commonly used api requests
- Object model for api items (User, Subreddit, Post, Comment, etc.)
- Watchers for **everything**
    - private messages
    - replies
    - new post reaching the front page (configurable to different pages/ # of pages)

