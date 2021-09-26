---
title: Getting my Yoga stats from Yogaglo
author: admin
type: post
date: 2015-06-17T19:16:33+00:00
url: /?p=1499
publicize_twitter_user:
  - snaveen
categories:
  - yoga
tags:
  - yoga

---
<div>
  <h1>
  </h1>
  
  <p>
    I am Yogi and have practiced some sort physical workout for a while now. IMHO physical strength/ movement have always attributed to better clarity in my life! This post shows how I managed to get my <a title="Yogaglo" href="https://www.yogaglo.com/mypractice" target="_blank">yogaglo</a> stats to track and measure my practice.
  </p>
  
  <p>
    <a href="https://naveensrinivasan.files.wordpress.com/2015/06/naveensrinivasan-trx.jpg"><img class=" wp-image-1503 size-large alignnone" src="https://naveensrinivasan.files.wordpress.com/2015/06/naveensrinivasan-trx.jpg?w=343" alt="NaveenSrinivasan-TRX" width="343" height="1024" /></a>
  </p>
  
  <p>
    I am strong believer in habit loops and always have found that has worked a lot for me. One of the good books on this which I recommend to other is<a title="Power Of Habit" href="http://www.amazon.com/The-Power-Habit-What-Business/dp/081298160X" target="_blank">http://www.amazon.com/The-Power-Habit-What-Business/dp/081298160X</a> and also another good resource is <a href="http://getupandcode.com/" target="_blank">http://getupandcode.com/</a> which is a audio podcast fitness and technology.
  </p>
  
  <p>
    I have been practicing Yoga for a while now and I would like to track my practice. I usually go to studio twice a week to be part of the <a title="Sangha" href="https://en.wikipedia.org/?title=Sangha" target="_blank">Sangha</a> and the rest 4-5 days I practice twice a day.
  </p>
  
  <p>
    I knew yogaglo had my stats information stored in their site because when I logged into the site it did provide me with history. But I wanted the API to query based on the raw data. I wanted to track how often I worked and what kind of classes have I done. My goal was to work on the strengthening my core and I usually like to track that and API would help with this kind of information.
  </p>
  
  <p>
    Thanks to tools like <a title="fiddler" href="http://www.telerik.com/fiddler" target="_blank">fiddler</a> or <a href="http://mitmproxy.org/" target="_blank">http://mitmproxy.org/</a> I could look at the http traffic that was sent with the headers. The headers are important because it contained the authentication token information. FYI I have set yogaglo to remember my login information which meant I have cookies that it could send across part of the http request.
  </p>
  
  <p>
    Here is the code to download the yogaglo stats
  </p>
  
  <p>
    [gist https://gist.github.com/naveensrinivasan/2e40f409bf6c386766c6]
  </p>
  
  <p>
    You could take the json and dump into excel and get some amazing stats using <a href="https://support.office.com/en-us/article/Introduction-to-Microsoft-Power-Query-for-Excel-6E92E2F4-2079-4E1F-BAD5-89F6269CD605" target="_blank">powerquery</a>.
  </p>
  
  <p>
    I am not a excel whiz to do it. I used the json to convert it to C# objects using<a href="http://json2csharp.com/" target="_blank">http://json2csharp.com/</a> and here is the code it generated.
  </p>
  
  <p>
    [gist https://gist.github.com/naveensrinivasan/0cf3cecede742c3587dc]
  </p>
  
  <p>
    With that here is a simple query to get total duration by date.
  </p>
  
  <p>
    [gist https://gist.github.com/naveensrinivasan/213c2092babc23d7c772]
  </p>
</div>