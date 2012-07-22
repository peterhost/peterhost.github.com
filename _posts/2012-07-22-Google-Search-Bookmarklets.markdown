---
layout: post
title: Custom Google Search Bookmarklets - past year, past month, custom range
categories: bookmarklet
tags: bookmarklet, google, search
intro: "Google search is great, but the defaults are not. If like myself you happen to google every two minutes for some answer to a Tech problem, you most certainly happen to spend like 10 friggin seconds huntig down for the `show search tools` ➔ `past year` option.<br /><br /><strong>Use Case:</strong> use bookmarklets to fix that ! Don't let google candy drive you nuts, dont let google tell you that '1 month' is a better time-based search option than '3 months' !!<br/><br/>Logo stolen from awesome graphic designer James Cook from ToonRefugee : http://toonrefugee.com/toonblog/google-cartoons/google-is-evil"
intro-img: "2012-07-22-Google-Search-Bookmarklets.jpg"

---


# Bookmarkleting tools

Today eventually, i got pissed.

Pissed enough about having to hunt down for
google's `show search tools` ➔ `past year` options *every single frigging time* 
I need to filter that particular set of posts which have been
closed for a decade and still show up at the first position when I search for, say,
`Ubuntu 12.04 openvpn bridged`...

Pissed enough about my procrastination, that I decided it was high time I wrote me somme bookmarklets.

I guess you know what bookmarklets are (basically, a local anonymous
javascript function launched by clicking/touching/licking a bookmark wich
sratrs with `javascript:` and which executes itself in the current
environment of the loaded page/DOM).

Here are some tools you can use to ease the process :

1. you've grabbed a bookmarklet and wish to edit the URI-encoded code in
your editor : [http://urldecode.org/](http://urldecode.org/) it already
!
2. thing is minified, beautify it : [http://jsbeautifier.org/](http://jsbeautifier.org/)
3. work on the code
4. minify it : [http://jscompress.com/](http://jscompress.com/)
5. Bookmarklet it : [http://chris.zarate.org/bookmarkleter](http://chris.zarate.org/bookmarkleter)
6. Copy/paste the resulting URI-encoded javascript anonymous function
call with `javascript:` prefix to a dummy bookmark, and you're done

# I'm on a mobile platform !

I feel your pain. Adding bookmarklets in mobile safari is as fun as
giving Hercules a hand cleaning Augeas' stables. So there is one mobile
bookmarklet to rule them all, the first one to get in your bookmarks,
and that's [the one from CSS Ninja](http://www.thecssninja.com/javascript/iphone-bookmarklet).

On a mobile phone/tablet, only thing you can do with a bookmarklet is
"touch" it and let it execute. This special bookmarklet renders all
bookmarklet links within a page **mobile** bookmark-able.

<a class='bookmarklet' href='javascript:(function(){var%20i=document.links.length;while(i--){if(/^javascript:/.test(document.links[i].href)){var%20linkStyle=document.links[i].style;document.links[i].href='#removeme_'+document.links[i].href;linkStyle.backgroundColor='#1E528C';linkStyle.color='#fff';linkStyle.fontWeight='bold';}}})();'>Mobile Bookmarklet Maker</a>

<br/><br/><br/>

# The Google Search Bookmarklets :

If, like I was you're in a hurry, just drag the following links (yeah,
the little blueish boxes) to your
bookmark bar

Show me Google's results for current search in the past :


<a class='bookmarklet' href='javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=y1%26q=%22+escape(p)}})();'>Past Year</a>
<a class='bookmarklet' href='javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=y2%26q=%22+escape(p)}})();'>Past 2 Years</a>
<a class='bookmarklet' href='javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=y3%26q=%22+escape(p)}})();'>Past 3 Years</a>


<a class='bookmarklet' href='javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=m9%26q=%22+escape(p)}})();'>Past 9 Months</a>
<a class='bookmarklet' href='javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=m6%26q=%22+escape(p)}})();'>Past 6 Months</a>
<a class='bookmarklet' href='javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=m2%26q=%22+escape(p)}})();'>Past 2 Months</a>

<a class='bookmarklet' href='javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=w2%26q=%22+escape(p)}})();'>Past 2 Weeks</a>
<a class='bookmarklet' href='javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=d4%26q=%22+escape(p)}})();'>Past 4 Days</a>
<a class='bookmarklet' href='javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=h2%26q=%22+escape(p)}})();'>Past 2 Hours</a>
<br/><br/><br/><br/><br/>
And as a bonus (see below, it's so simple to implement you could even
die) :
<br/>
<a class='bookmarklet' href='javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=y1%26as_filetype=pdf%26q=%22+escape(p)}})();'>PDF, 1 year max</a>
<br/><br/><br/><br/><br/>

Then head on to `google.com`, do some reseach, and hit the bookmarklets
to see the `search results` adapt.

<br/><br/><br/>

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

## the key points here are :

1. `the getQueryString()` function (thanks to [CMS on stackoverflow](http://stackoverflow.com/a/647272/474526)) which plainly extracts parameters to the `query string` (if any) of the page you're currently on. In our case, `google.com`.
2. the modification of `document.location.href` : the parameter to
re-inject is : `_qdr=xxx` where `xxx` can be :
    * `y1` : one year
    * `m1` : one month
    * `d7` : 7 days
    * `h2` : 2 hours
    * and so on...

###Minified

    function getQueryString(){var a={},b=location.search.substring(1),c=/([^&=]+)=([^&]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return a}p=getQueryString()["q"];if(p){document.location.href="http://www.google.com/search?as_qdr=y1&q="+escape(p)}

###Bookmarkletified

    javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=y1%26q=%22+escape(p)}})();

That's all there is to it.

## Do whaterver you like !

As you can see, you can configure your bookmarklet to do som advanced
google search provided you know the `query string`'s parameter's name.

>Example : only display pdfs, less thant 1 year old.
>
>Google's query string parameter for that is : `as_filetype=pdf`
>
>Code would be :


    function getQueryString() {
      var result = {}, queryString = location.search.substring(1),
      re = /([^&=]+)=([^&]*)/g, m;

      while (m = re.exec(queryString)) {
        result[decodeURIComponent(m[1])] = decodeURIComponent(m[2]);
      }

      return result;
    }

    p=getQueryString()["q"]
    p=getQueryString()["q"]
    if(p){
      document.location.href='http://www.google.com/search?as_qdr=y1&as_filetype=pdf&q='+escape(p)
    }



<a class='bookmarklet' href='javascript:(function(){function%20getQueryString(){var%20a={},b=location.search.substring(1),c=/([^%26=]+)=([^%26]*)/g,d;while(d=c.exec(b)){a[decodeURIComponent(d[1])]=decodeURIComponent(d[2])}return%20a}p=getQueryString()[%22q%22];if(p){document.location.href=%22http://www.google.com/search%3Fas_qdr=y1%26as_filetype=pdf%26q=%22+escape(p)}})();'>PDF, 1 year max</a>
<br/><br/><br/><br/><br/>

# Are you a true Bookmarklet NERD ?



![A true nerdy-geek's bookmark bar has to look something like this](img/covers/2007-07-22-bookmarkbar.png)

As a matter of fact, when I said earlier that the bookmarklet from
 [CSS Ninja](http://www.thecssninja.com/javascript/iphone-bookmarklet)
 was the **ruler** of all living bookmarklet, I was being unfair. The
 first bookmarklet in *any of my bookmarks bar*, the one with the
 enclosed letter "Q" in the above screenshot, **The One Ring of bookmarklet** [is called QUIX](http://www.quixapp.com/).
 
 Once you've seen the potential of bookmarklets and begin to dig the
 stuff, you'll never regret the 5 minutes you spent watching Quix's
 introductory screencast, *my preciousss*.

