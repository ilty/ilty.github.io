---
layout: post
title:  "mysql 主从复制配置"
date:   2014-05-20
desc: "mysql 主从复制配置"
keywords: "架构,mysql,主从复制"
categories: [Framework]
tags: [小记]
icon: icon-centos
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 作为互联网公司的信息安全从业人员经常要处理撞库扫号事件，产生撞库扫号的根本原因是一些企业发生了信息泄露事件，且这些泄露数据未加密或者加密方式比较弱，导致黑客可以还原出原始的用户密码。目前已经曝光的信息泄露事件至少上百起，其中包括多家一线互联网公司，泄露总数据超过10亿条。本文作者就职于携程技术中心信息安全部，文中他将分享用户密码的加密方式以及主要的破解方法。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 要完全防止信息泄露是非常困难的事情，除了防止黑客外，还要防止内部人员泄密。但如果采用合适的算法去加密用户密码，即使信息泄露出去，黑客也无法还原出原始的密码（或者还原的代价非常大）。也就是说我们可以将工作重点从防止泄露转换到防止黑客还原出数据。下面我们将分别介绍用户密码的加密方式以及主要的破解方法。

<h2>用户密码加密：</h2>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 用户密码保存到数据库时，常见的加密方式有哪些，我们该采用什么方式来保护用户的密码呢？以下几种方式是常见的密码保存方式：

1.直接明文保存，比如用户设置的密码是“123456”，直接将“123456”保存在数据库中，这种是最简单的保存方式，也是最不安全的方式。但实际上不少互联网公司，都可能采取的是这种方式。

2.使用对称加密算法来保存，比如3DES、AES等算法，使用这种方式加密是可以通过解密来还原出原始密码的，当然前提条件是需要获取到密钥。不过既然大量的用户信息已经泄露了，密钥很可能也会泄露，当然可以将一般数据和密钥分开存储、分开管理，但要完全保护好密钥也是一件非常复杂的事情，所以这种方式并不是很好的方式。

![User1]({{ site.img_path }}/framework/user1.jpg)

3.使用MD5、SHA1等单向HASH算法保护密码，使用这些算法后，无法通过计算还原出原始密码，而且实现比较简单，因此很多互联网公司都采用这种方式保存用户密码，曾经这种方式也是比较安全的方式，但随着彩虹表技术的兴起，可以建立彩虹表进行查表破解，目前这种方式已经很不安全了。

![User1]({{ site.img_path }}/framework/user2.jpg)

4.特殊的单向HASH算法，由于单向HASH算法在保护密码方面不再安全，于是有些公司在单向HASH算法基础上进行了加盐、多次HASH等扩展，这些方式可以在一定程度上增加破解难度，对于加了“固定盐”的HASH算法，需要保护“盐”不能泄露，这就会遇到“保护对称密钥”一样的问题，一旦“盐”泄露，根据“盐”重新建立彩虹表可以进行破解，对于多次HASH，也只是增加了破解的时间，并没有本质上的提升。

![User1]({{ site.img_path }}/framework/user3.jpg)

5.PBKDF2算法，该算法原理大致相当于在HASH算法基础上增加随机盐，并进行多次HASH运算，随机盐使得彩虹表的建表难度大幅增加，而多次HASH也使得建表和破解的难度都大幅增加。使用PBKDF2算法时，HASH算法一般选用sha1或者sha256，随机盐的长度一般不能少于8字节，HASH次数至少也要1000次，这样安全性才足够高。一次密码验证过程进行1000次HASH运算，对服务器来说可能只需要1ms，但对于破解者来说计算成本增加了1000倍，而至少8字节随机盐，更是把建表难度提升了N个数量级，使得大批量的破解密码几乎不可行，该算法也是美国国家标准与技术研究院推荐使用的算法。

![User1]({{ site.img_path }}/framework/user4.jpg)

6.bcrypt、scrypt等算法，这两种算法也可以有效抵御彩虹表，使用这两种算法时也需要指定相应的参数，使破解难度增加。

下表对比了各个算法的特性：

<table cellpadding="0" cellspacing="0" width="623" interlaced="enabled"><colgroup><col width="116" style=";width:116px"  /><col width="132" style=";width:132px"  /><col width="121" style=";width:121px"  /><col width="81" style=";width:81px"  /><col width="172" style=";width:172px"  /></colgroup><tbody><tr height="20" style="height:20px"><td height="20" width="116">算法</td><td width="132" style="">特点</td><td width="121" style="">有效破解方式</td><td width="81" style="">破解难度</td><td width="172" style="">其它</td></tr><tr height="20" style="height:20px" ><td height="20">明文保存</td><td>实现简单</td><td>无需破解</td><td>简单</td><td><br  /></td></tr><tr height="20" style="height:20px" ><td height="20">对称加密</td><td>可以解密出明文</td><td>获取密钥</td><td>中</td><td>需要确保密钥不泄露</td></tr><tr height="20" style="height:20px" ><td height="20">单向HASH</td><td>不可解密</td><td>碰撞、彩虹表</td><td>中</td><td><br  /></td></tr><tr height="20" style="height:20px" ><td height="20">特殊HASH</td><td>不可解密</td><td>碰撞、彩虹表</td><td>中</td><td>需要确保“盐”不泄露</td></tr><tr height="20" style="height:20px" ><td height="20">Pbkdf2</td><td>不可解密</td><td>无</td><td>难</td><td>需要设定合理的参数</td></tr></tbody></table>
<tr>
## 用户密码破解

用户密码破解需要针对具体的加密方式来实施，如果使用对称加密，并且算法足够安全（比如AES），必须获取到密钥才能解密，没有其它可行的破解方式。

如果采用HASH算法（包括特殊HASH），一般使用彩虹表的方式来破解，彩虹表的原理是什么呢？我们先来了解下如何进行HASH碰撞。单向HASH算法由于不能进行解密运算，只能通过建表、查表的方式进行碰撞，即将常用的密码及其对应的HASH值全计算出来并存储，当获取到HASH值是，直接查表获取原始密码，假设用MD5算法来保护6位数字密码，可以建如下表：

<table cellpadding="0" cellspacing="0" width="717" interlaced="enabled"><colgroup><col width="168" style=";width:168px"  /><col width="549" style=";width:549px"  /></colgroup><tbody><tr height="24" style="height:24px" ><td height="24" width="168" style="background-color: rgb(187, 187, 187);">原始密码</td><td width="549" style="background-color: rgb(187, 187, 187);">MD5值</td></tr><tr height="93" style="height:93px"><td height="93" width="168">0</td><td width="549" style="">670B14728AD9902AECBA32E22FA4F6BD</td></tr><tr height="92" style="height:92px" ><td height="92" width="168" style="background-color: #FAFBFB">1</td><td width="549" style="background-color: #FAFBFB">04FC711301F3C784D66955D98D399AFB</td></tr><tr height="24" style="height:24px"><td height="24" width="168">…</td><td width="549" style="">…</td></tr><tr height="92" style="height:92px" ><td height="92" width="168" style="background-color: #FAFBFB">999999</td><td width="549" style="background-color: #FAFBFB">52C69E3A57331081823331C4E69D3F2E</td></tr></tbody></table>

全表共100W条记录，因为数据量不大，这种情况建表、查表都非常容易。但是当密码并不是6位纯数字密码，而是数字、大小写字母结合的10位密码时，建立一个这样的表需要（26+26+10）^ 10 ≈ 83亿亿（条记录），存储在硬盘上至少要占用2000W TB的空间，这么大的存储空间，成本太大，几乎不可行。有什么办法可以减少存储空间？一种方法是“预计算哈希链”，“预计算哈希链”可以大幅减少HASH表的存储空间，但相应的增加了查表时的计算量，其原理大致如下：


建表过程：

![User1]({{ site.img_path }}/framework/user5.jpg)

先对原始数据“000000”进行一次HASH运算得到“670B1E”，再对HASH值进行一次R运算，R是一个定制的算法可以将HASH值映射到明文空间上（这里我们的明文空间是000000~999999），R运算后得到“283651”，再对“283651”进行hash运算得到“1A99CD”，然后在进行R运算得到“819287”，如此重复多次，得到一条哈希链。然后再选用其它原始数据建立多条哈希链。最终仅将链头和链尾保存下来，中间节点全都去掉。

查表过程：假设拿到了一条HASH值“670B1E”，首先进行一次R运算，得到了“283651”，查询所有链尾是否有命中，如果没有，则再进行一次HASH、一次R，得到了“819287”，再次所有链尾，可以得到看出已经命中。这样我们就可以基本确认“670B1E”对应的明文就在这条链上，然后我们把这条链的生成过程进行重新计算，计算过程中可以发现“000000”的HASH值就是“670B1E”，这样就完成了整个查表过程。这种表就是“预计算哈希链”。这种方式存在一个问题，多条链之间可能存在大量的重复数据，如下图所示：

![User1]({{ site.img_path }}/framework/user6.jpg)

为了解决这个问题，我们将R算法进行扩展，一条链上的多次R运算采用不同的算法，如下图：

![User1]({{ site.img_path }}/framework/user7.jpg)

一条链上的每个R算法都不一样，就像彩虹的每层颜色一样，因此取名的为彩虹表。

当然彩虹表除了可以用户破解HASH算法外，理论上还可以用于破解对称加密算法，比如DES算法，由于DES算法密钥比较短，建立彩虹表破解是完全可行的；但对于AES算法，由于密钥比较长，建表几乎不可行（需要耗时N亿年）。

## 小结

采用PBKDF2、bcrypt、scrypt等算法可以有效抵御彩虹表攻击，即使数据泄露，最关键的“用户密码”仍然可以得到有效的保护，黑客无法大批量破解用户密码，从而切断撞库扫号的根源。当然，对于已经泄露的密码，还是需要用户尽快修改密码，不要再使用已泄露的密码。