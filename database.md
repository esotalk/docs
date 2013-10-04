# Database

- [Constructing Queries](#queries)
- [Schema](#schema)

The **[ETDatabase](/api/class-ETDatabase.html)** instance is stored in the static $database property of the ET class. This object handles the database connection and runs queries.

**Executing A MySQL Query**

	ET::$database->query("SELECT 1");
	
**Getting The Last Insert ID**

	$id = ET::$database->lastInsertId();

<a name="queries"></a>
## Constructing Queries

esoTalk provides a class called **[ETSQLQuery](/api/class-ETSQLQuery.html)** which allows MySQL queries to be constructed easily and dynamically. This brings many benefits — most significantly, giving plugins the power to alter queries before they are executed.

An instance of the ETSQLQuery class can be created using the `ET::SQL` method. [Many methods](/api/class-ETSQLQuery.html) are available on the ETSQLQuery class that allow the different parts of the MySQL query to be altered.

**Constructing A Simple SELECT Query**

	$sql = ET::SQL();
	$sql
		->select("username")
		->from("member")
		->where("memberId", 1);
	
	echo $sql->get(); // Outputs: SELECT `username` FROM `et_member` WHERE `memberId` = 1
	$result = $sql->exec(); // Executes the query

Placeholders can be inserted into any component of the query. Data can then be bound to those placeholders using the `bind` method and esoTalk will handle all escaping to make sure the query is safe.

**Binding Data To A Query And Getting The Result**

	$result = ET::SQL()
		->select("memberId")
		->from("member")
		->where("username = :username OR countPosts = :posts")
		->bind(":username", $username)
		->bind(":posts", $posts, PDO::PARAM_INT)
		->exec();
		
**Constructing A More Complex Query**

	$result = ET::SQL()
		->select("p.*")
		->select("m.username", "name")
		->from("post p")
		->from("member m", "m.memberId = p.memberId", "left")
		->where("p.postId IN (:postIds)")
		->orderBy("p.time ASC")
		->limit(20)
		->bind(":postIds", array(1, 2, 3))
		->exec();

The result returned by the `query` method of the ETDatabase object — and in turn, the `exec` method of an ETSQLQuery object — is a new instance of the **[ETSQLResult](/api/class-ETSQLResult.html)**. This is a wrapper class with various functions that can be used to read the resultset.

**Getting The Number Of Rows In The Resultset**

	$sql = ET::SQL()->select("*")->from("member");
	$result = $sql->exec();
	$rowCount = $result->numRows();
	
**Getting A Multidimensional Array Of All Rows**
	
	$rows = $result->allRows();
	
**Getting A Keyed Multidimensional Array Of All Rows**

	$rowsKeyed = $result->allRows("memberId");
	
**Getting The Value Of The First Column Of The First Row**

	$value = $result->result();

The ETSQLQuery class can also construct UPDATE, INSERT, and DELETE queries.

**Constructing And Running An UPDATE Query**

	ET::SQL()
		->update("member")
		->set("username", "Toby")
		->set(array(
			"email" => "toby@esotalk.org",
			"account" => "administrator"
		))
		->where("memberId", 1)
		->exec();
		
**Constructing And Running An INSERT Query**

	ET::SQL()
		->insert("member")
		->set("username", "Toby")
		->exec();
		
**Inserting Multiple Rows At Once**

	ET::SQL()
		->insert("member")
		->setMultiple(array("username", "email"), array(
			array("Toby", "toby@esotalk.org"),
			array("Simon", "simon@esotalk.org")
		))
		->exec();
	
<a name="schema"></a>
## Schema

The **[ETDatabaseStructure](/api/class-ETDatabaseStructure.html)** class provides a way to easily manage and maintain the database schema without having to worry about writing complex `ALTER TABLE` queries specific to each upgrade. This class will automatically compare the current structure of the database to a defined schema, and perform the necessary operations to update the current structure so that it matches.

An ETDatabaseStructure instance is accessible via the `ETDatabase::structure` method. On this object, the structure of a table can be defined by chaining method calls, starting with the `table` method, and following with `column` and `key` methods. Finally, the `exec` method is used to execute the queries necessary to get the database structure up-to-date.

**Getting The ETDatabaseStructure Instance**

	$structure = ET::$database->structure();

The `table` method takes an argument for the table name, and an optional argument for the engine to use (*InnoDB* is used by default.)

**Defining A Table Using The MyISAM Engine**

	$structure->table("member", "MyISAM");

The `column` method takes three arguments: the column name, the column type signature, and the optional default value of the column (*NULL* by default.)

**Defining A Column**

	$structure->column("username", "varchar(255)");
	
**Defining A NOT NULL Column**

	$structure->column("memberId", "int(11) unsigned", false);

The `key` method takes two arguments: the name(s) of the column(s) to apply an key to, and the type of key.

**Defining A Primary Key**

	$structure->key("memberId", "primary");
	
**Defining A Multi-Column Key**

	$structure->key(array("foo", "bar"));
	
**Defining A Fulltext Index**

	$structure->key("content", "fulltext");

Once the appropriate schema has been defined, the `exec` method can be called to apply necessary changes to the database. Note that columns will not be dropped using this method; they will only be created or altered. This method also takes an optional argument which, when true, will drop the table if one exists and recreate it from scratch.

**Creating Or Updating An Existing Table Schema**

	$structure->exec();
	
**Dropping And Creating A Table From Scratch**

	$structure->exec(true);

> The core esoTalk tables are defined in `core/models/ETUpgradeModel.class.php`.

Sometimes manual operations need to be performed in order to upgrade the database schema, such as dropping or renaming individual columns and keys. The ETDatabaseStructure class provides convenient methods to do so.

**Determining Whether Or Not A Table Exists**

	$exists = $structure->table("foo")->exists();

**Dropping A Column**

	$structure->dropColumn("bar");
	
**Renaming a Column If It Exists**

	if ($structure->columnExists("foo")) {
		$structure->renameColumn("foo", "bar");
	}
	
**Dropping A Key**
	
	$structure->dropKey("primary");