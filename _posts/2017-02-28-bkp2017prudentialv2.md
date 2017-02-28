---
title: Boston Key Party CTF 2017 - Prudentialv2
layout: post
post_author: pickacard and lou
excerpt_separator: "<!--more-->"
---

Short version: make this script print the flag.

```
<?php
require 'flag.php';

if (isset($_GET['name']) and isset($_GET['password'])) {
    $name = (string)$_GET['name'];
    $password = (string)$_GET['password'];

    if ($name == $password) {
        print 'Your password can not be your name.';
    } else if (sha1($name) === sha1($password)) {
      die('Flag: '.$flag);
    } else {
        print '<p class="alert">Invalid password.</p>';
    }
}
?>
```

There are two possible ways to solve this: either one finds a collision on SHA-1, or one manages to convince PHP to enter the `else if` without doing so.<!--more-->
We were aware of the [latest news](http://shattered.io) on the topic, so we tried right away to pass the colliding PDFs as parameters.
Since the PDFs were too big, we tried some reasoning to see if we could feed only some parts of them to the script.
Reading other writeups, it is clear that this was the right approach, but we failed, missing the correct way of thinking.

So we started a long journey into PHP internals too see if we could somehow cheat. We learned a lot, but we didn't find a way.

We returned on the PDF path, and we managed to solve the challenge with this overcomplicated approach:
* we produced the smallest colliding PDFs we could think of with [this nice tool](https://alf.nu/SHA1), using the [smallest possible JPEG we could find](https://github.com/mathiasbynens/small/blob/master/jpeg.jpg) (and a copy of it with one byte manually changed). We got two nice colliding PDFs, 1742 bytes each.
* we tried feeding the URL-encoded PDFs as parameters... nope, still too big: every byte sent as %XX is 3 characters.  3 x 1742 x 2 is greater than 8000, roughly the limit for the "URI too long" error.
* we URL-encoded only the non-alphanumeric characters in the PDFs, so for example {0x00,0x41,0x42,0x01} would be sent as "%00AB%01". This time we were under the character limit, so it worked.

Not the most efficient approach, but we had fun!