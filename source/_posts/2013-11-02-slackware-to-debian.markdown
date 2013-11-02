---
layout: post
title: "slackware to debian"
date: 2013-10-26 23:12
comments: true
categories: 
- server
---
{% img http://slackware.com/grfx/shared/slackware_traditional_website_logo.png %}
{% img http://www.debian.org/logos/openlogo-100.jpg %}

最开始接触Linux是在大一，还记得当时看的一本入门书叫[《A STUDENT'S GUIDE TO UNIX》](http://book.douban.com/subject/2143320/)。然后自己折腾过很多Linux发行版，CentOS，Debian，Slackware，Arch，Gentoo，LFS，还有FreeBSD。后来非常长一段时间都在使用[Slackware](http://slackware.com/)，我记得当时用的是Slackware 10.0。

至今，手头上还有机子是用Slackware的，包括以前的Linode VPS、现在学校的服务器等等。

我想，只要玩过Slackware的人，都知道他有明显的优点：简洁、稳定、高效。因为它里面的软件包都是原汁原味的，虽然包的数目不多，但加之[SlackBuilds](http://slackbuilds.org)的，总数都有四五千个，日常是足够用的，起码在我的使用经验来说已经足够。PkgTools包管理机制，最大的特点是无依赖，这可以说是优点，也可以说是缺点。优点当然是这样组织起来，包与包的关系很明晰，要什么包就装什么，而不会装了一些不相关的包。缺点也明显，就是对管理员有一定的要求，起码是了解Linux的，如果装一些软件，可能要手动解决包间的依赖关系。我一般都是用Full Installation的，如果用SlackBuilds，也会列出相关的依赖，手动解决也很快，现在也有[sbopkg](http://www.sbopkg.org/)这个工具。还有一点我很欣赏，就是它启动脚本管理方式是那简的简洁简单。

说了这么多，那为什么我现在下定决心转到[Debian](http://debian.org)呢？其实也有非主观因素。对于服务器的管理，可能并不是你一个人，一直打理下去，总得有时会换人吧？！对于slackware，特别是在中国，真的太少众了，资料也缺，假如换人了，那他管理起来可能比较吃力。至少Debian来说，管理起来要方便快捷得多，团队也强大，当然包的数量也是我选择这个发行版的原因之一。有人问，为什么不选CentOS，我一向不喜欢RPM系的，所以，没办法，而Debian的开放自由，刚刚好可能满足我和今后相关管理人员的最基本需要，虽然我真的不喜欢deb装一大堆依赖的东西。

每个人做出一个选择，都有自己的现实需要。还记得那句话，服务器稳不稳定，除了好的发行版外，更关键的是作为一个服务器管理人员的技术水平。所以，说到底，哪个版本都不重要，重要的是，能用好，用到点子上，并能一直稳定地工作下去。

说个题外话吧，以前Slackware给我印象，是因为求稳定，很多软件包的确比较旧，但现在用Debian了，发现，Slackware的包已经非常新了。Slackware的发展也根据现在Linux发展一样，慢慢地加快，跟上了脚步。但我也相信它可以一直坚持它KISS原则，一直良好地发展下去。
