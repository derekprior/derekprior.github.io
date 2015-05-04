---
layout: post
title: "Re-enabling Password Pasting"
date: 2012-08-08 10:40
summary: Some misguided web developers have decided pasting passwords should
  be prohibited. If you want to paste your secure password into one of these
  annoying web forms, then you've come to the right place.
---

As a [1password][1p] user, I usually have very long, unique, and random
passwords for every site that requires one. I use the 1Password browser
extensions to fill password forms for me, but on iOS I copy and paste the
values manually. Some misguided web developers have decided pasting
passwords should be prohibited and have added `onpaste` events to prevent
it. The latest culprit I have encountered is [Apple][apple].

While this is an infrequent annoyance it can be crippling. To combat it, I
whipped up a bookmarklet that will run some JavaScript on the page to remove
`onpaste` events from all password fields.

To use the bookmarklet, drag the
<a href="javascript:(function(){var inputs=document.getElementsByTagName('input');for(var i=0;i<inputs.length;i++){if(inputs[i].getAttribute('type').toLowerCase()==='password'){inputs[i].setAttribute('onpaste','');}}})();">allow pwpaste</a>
bookmarklet link to your bookmark toolbar. When you encounter one of these
annoying forms, simply click the bookmark to re-enable pasting. The bookmarklet
source is available in a more readable format below.

Adding bookmarklets to mobile safari is a [giant pain][msb], so I would suggest
adding the bookmarklet in desktop Safari and using iCloud to sync it.

```javascript
var inputs = document.getElementsByTagName('input');
for (var i=0; i < inputs.length; i++) {
  if (inputs[i].getAttribute('type').toLowerCase() === 'password') {
    inputs[i].setAttribute('onpaste', '');
  }
}
```

[1p]: https://agilebits.com/onepassword
[apple]: https://appleid.apple.com/cgi-bin/WebObjects/MyAppleId.woa/372/wa/directToSignIn
[msb]: http://chrisbray.com/a-not-so-easy-way-to-add-bookmarklets-to-safa
