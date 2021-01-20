# db_placehold

Simple php db placeholder for query debug

#### Code

```
<?php
function placehold()
{
    $args = func_get_args();
    if (!$args) return "Empty query";
    $q = $args[0];
    if (func_num_args() - 1 != substr_count($q, '?')) return "Mismatch on Variables to Placeholders (?)";
    $r = "";
    $i = 1;
    foreach (str_split($q) as $v) {
        if ($v === '?') {
            $arg = $args[$i++];
            $r .= is_int($arg) ? $arg : "'$arg'";
        } else $r .= $v;
    }
    return $r;
}
```

#### Usage

```
<?php
$name = 'Jonh';
$id = 1;
echo placehold('SELECT * FROM users WHERE name=? AND id=? ', $name, $id);
```

#### Result

```SELECT * FROM users WHERE name='Jonh' AND id=1```

## Or you can add it as a public function to any your class

#### Code
[Link to original db class!](https://codeshack.io/super-fast-php-mysql-database-class/)
```
<?php
class db
{

	protected $connection;
	protected $query;
	protected $show_errors = TRUE;
	protected $query_closed = TRUE;
	public $query_count = 0;

	public function __construct($dbhost = 'localhost', $dbuser = 'root', $dbpass = '', $dbname = '', $dbport = '3306', $charset = 'utf8')
	{
		$this->connection = new mysqli($dbhost, $dbuser, $dbpass, $dbname, $dbport);
		if ($this->connection->connect_error) {
			$this->error('Failed to connect to MySQL - ' . $this->connection->connect_error);
		}
		$this->connection->set_charset($charset);
	}

	public function query($query)
	{
		if (!$this->query_closed) {
			$this->query->close();
		}
		if ($this->query = $this->connection->prepare($query)) {
			if (func_num_args() > 1) {
				$x = func_get_args();
				$args = array_slice($x, 1);
				$types = '';
				$args_ref = array();
				foreach ($args as $k => &$arg) {
					if (is_array($args[$k])) {
						foreach ($args[$k] as $j => &$a) {
							$types .= $this->_gettype($args[$k][$j]);
							$args_ref[] = &$a;
						}
					} else {
						$types .= $this->_gettype($args[$k]);
						$args_ref[] = &$arg;
					}
				}
				array_unshift($args_ref, $types);
				call_user_func_array(array($this->query, 'bind_param'), $args_ref);
			}
			$this->query->execute();
			if ($this->query->errno) {
				$this->error('Unable to process MySQL query (check your params) - ' . $this->query->error);
			}
			$this->query_closed = FALSE;
			$this->query_count++;
		} else {
			$this->error('Unable to prepare MySQL statement (check your syntax) - ' . $this->connection->error);
		}
		return $this;
	}


	public function fetchAll($callback = null)
	{
		$params = array();
		$row = array();
		$meta = $this->query->result_metadata();
		while ($field = $meta->fetch_field()) {
			$params[] = &$row[$field->name];
		}
		call_user_func_array(array($this->query, 'bind_result'), $params);
		$result = array();
		while ($this->query->fetch()) {
			$r = array();
			foreach ($row as $key => $val) {
				$r[$key] = $val;
			}
			if ($callback != null && is_callable($callback)) {
				$value = call_user_func($callback, $r);
				if ($value == 'break') break;
			} else {
				$result[] = $r;
			}
		}
		$this->query->close();
		$this->query_closed = TRUE;
		return $result;
	}

	public function fetchArray()
	{
		$params = array();
		$row = array();
		$meta = $this->query->result_metadata();
		while ($field = $meta->fetch_field()) {
			$params[] = &$row[$field->name];
		}
		call_user_func_array(array($this->query, 'bind_result'), $params);
		$result = array();
		while ($this->query->fetch()) {
			foreach ($row as $key => $val) {
				$result[$key] = $val;
			}
		}
		$this->query->close();
		$this->query_closed = TRUE;
		return $result;
	}

	public function close()
	{
		return $this->connection->close();
	}

	public function numRows()
	{
		$this->query->store_result();
		return $this->query->num_rows;
	}

	public function affectedRows()
	{
		return $this->query->affected_rows;
	}

	public function lastInsertID()
	{
		return $this->connection->insert_id;
	}

	public function error($error)
	{
		if ($this->show_errors) {
			exit($error);
		}
	}

	private function _gettype($var)
	{
		if (is_string($var)) return 's';
		if (is_float($var)) return 'd';
		if (is_int($var)) return 'i';
		return 'b';
	}
    
    	/** Simple placehold */
	public function placehold()
	{
		$args = func_get_args();
		if (!$args) return "Empty query";
		$q = $args[0];
		if (func_num_args() - 1 != substr_count($q, '?')) return "Mismatch on Variables to Placeholders (?)";
		$r = "";
		$i = 1;
		foreach (str_split($q) as $v) {
			if ($v === '?') {
				$arg = $args[$i++];
				$r .= is_int($arg) ? $arg : "'$arg'";
			} else $r .= $v;
		}
		return $r;
	}
}
```

#### Usage

```
<?php
$c = include('config.php');
$db = new db($c->server, $c->user, $c->password, $c->name);
$name = 'Jonh';
$id = 1;
echo $db->placehold('SELECT * FROM users WHERE name=? AND id=? ', $name, $id);
```

#### Result

```SELECT * FROM users WHERE name='Jonh' AND id=1```
