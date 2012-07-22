---
layout: post
title: Custom Google Search Bookmarklets - past year, past month, custom range
categories: bookmarklet
tags: bookmarklet, google, search
intro: "Google search is great, but the defaults are not. If like myself you happen to google every two minutes for some answer to a Tech problem, you most certainly happen to spend like 10 friggin seconds huntig down for the `show search tools` ➔ `past year` option.<br /><br /><strong>Use Case:</strong> use bookmarklets to fix that ! Don't let google candy drive you nuts, dont let google tell you that '1 month' is a better time-based search option than '3 months' !!<link href='http://toonrefugee.com/toonblog/google-cartoons/google-is-evil'>Logo by James Cook from ToonRefugee</link>"
intro-img: "2012-07-22-Google-Search-Bookmarklets.jpg"

---

# Bookmarkleting tools

When I eventually got pissed enough about having to hunt down for the `show search tools` ➔ `past year` options each time I wish to refine a google search
so as **not to include** that old question on linuxforums which has been
closed for a decade and still show up in first position regarding my
very urgent, recent concern with solving a config issue on a cutting
edge version of a software i'm setting up,...

I dediced i'd make my own bookmarklets.

There are 3 tools you can use which will ease the making :

1. you've grabbed a bookmarklet and wish to edit the URI-encoded code in
your editor : [http://urldecode.org/](http://urldecode.org/) it already
!
2. thing is minified, beautify it : [http://jsbeautifier.org/](http://jsbeautifier.org/)
3. work on the code
4. minify it : [http://jscompress.com/](http://jscompress.com/)
5. Bookmarklet it : [http://chris.zarate.org/bookmarkleter](http://chris.zarate.org/bookmarkleter)
6. Copy/paste the resulting URI-encoded javascript anonymous function
call with `javascript:` prefix to a dummy bookmark, and you're done

# The Bookmarklets :

If, like I was you're in a hurry, just drag the following links to your
bookmark bar

Show me Google's results for current search in the past :

1 year : <a class='bookmarklet' href='javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=y1%26q=%22+escape(p)}})();'>Past Year</a>


3 months : <a class='bookmarklet' href='javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=m6%26q=%22+escape(p)}})();'>Past 6 Months</a>

# The Code

## unminified

    function getQueryString() {
      var result = {}, queryString = location.search.substring(1),
      re = /([^&=]+)=([^&]*)/g, m;

      while (m = re.exec(queryString)) {
        result[decodeURIComponent(m[1])] = decodeURIComponent(m[2]);
      }

      return result;
    }

    p=getQueryString()["q"]
    if(p){
      document.location.href='http://www.google.com/search?as_qdr=y1&q='+escape(p)
    }

the key points here are :

1. `the getQueryString()` function (thanks to [CMS on stackoverflow](http://stackoverflow.com/a/647272/474526)) which plainly extracts parameters to the `query string` (if any) of the page you're currently on. In our case, `google.com`.
2. the modification of `document.location.href` : the parameter to
re-inject is : `_qdr=xxx` where `xxx` can be :
    * `y1` : one year
    * `m1` : one month
    * `d7` : 7 days
    * `h2` : 2 hours
    * and so on...
