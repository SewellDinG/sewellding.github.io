---
layout: post
title: Dynamic debugging with PHPStorm+MAMP on Mac
description: ""
keywords: "Sword"
---

# 1. Setting

**Enable Xdebug** in PHP setting, MAMP PRO.

![xdebug](/assets/images/2019-06-27/xdebug.png)

Settings have taken effect. In PHP.ini

![php.ini](/assets/images/2019-06-27/php.ini.png)

Open the project in PHPStorm. **In the upper right corner, Add Configuration, click + symbol, select PHP Web Page, set Name, select Server, click Apply and OK.**

![phpstorm](/assets/images/2019-06-27/phpstorm.png)

# 2. Debugging

Demo: QWB, Web, UPLOAD

Knowledge: code audit, PHP unserialize

Build an environment according to [qwb-2019-upload](https://github.com/glzjin/qwb_2019_upload) in MAMP PRO.

Looked and found that it has register, login, upload image, etc.

And leak source code at url/www.tar.gz. It is based on ThinkPHP v5.

First, look at the routing configuration and understand the general structure.

Then, look for unserialize function. It appears in the login_check() of index.php.

**Click at the beginning of the current line and appear a red dot.**

![reddot](/assets/images/2019-06-27/reddot.png)

**In the upper right corner, select the Server Name, click on the red bug to start debugging.**

![debug](/assets/images/2019-06-27/debug.png)

It will open this server using a browser. URL: http://qwb/?XDEBUG_SESSION_START=11956

**`?XDEBUG_SESSION_START=11956` is the bridge to PHPStorm and PHP Xdebug connections.**

NOTE: All urls that use it will pass PHPStorm!

Finally, implement the attack chain.

# 3. Attack chain

![demo](/assets/images/2019-06-27/demo.gif)

# Ref

[https://tttang.com/archive/1301](https://tttang.com/archive/1301)

[https://www.zhaoj.in/read-5873.html](https://www.zhaoj.in/read-5873.html)