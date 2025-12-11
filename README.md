**Project Title — LAB10_PHP_OOP**
### *Rafli Anugrah Ramadhan*

<p align="left">
  <!-- Python -->
  <img src="https://img.shields.io/badge/Python-000000?style=for-the-badge&logo=python&logoColor=yellow" />

  <!-- XAMPP -->
  <img src="https://img.shields.io/badge/XAMPP-000000?style=for-the-badge&logo=xampp&logoColor=orange" />

  <!-- GitHub -->
  <img src="https://img.shields.io/badge/GitHub-000000?style=for-the-badge&logo=github&logoColor=white" />
</p>

<img width="423" height="122" alt="image" src="https://github.com/user-attachments/assets/4b32de8d-2cc0-4f81-a3dc-51e33f38c227" />

1. Database

~~~
<?php

class Database
{
    protected $host;
    protected $user;
    protected $password;
    protected $db_name;
    protected $conn;

    public function __construct()
    {
        $this->getConfig();
        $this->conn = new mysqli(
            $this->host,
            $this->user,
            $this->password,
            $this->db_name
        );

        if ($this->conn->connect_error) {
            die("Connection failed: " . $this->conn->connect_error);
        }
    }

    private function getConfig()
    {
        include("config.php");
        $this->host = $config['host'];
        $this->user = $config['username'];
        $this->password = $config['password'];
        $this->db_name = $config['db_name'];
    }

    public function query($sql)
    {
        return $this->conn->query($sql);
    }

    public function get($table, $where = null)
    {
        if ($where) {
            $where = " WHERE " . $where;
        }
        $sql = "SELECT * FROM " . $table . $where;
        $sql = $this->conn->query($sql);
        return $sql->fetch_assoc();
    }

    public function insert($table, $data)
    {
        foreach ($data as $key => $val) {
            $column[] = $key;
            $value[] = "'{$val}'";
        }

        $columns = implode(",", $column);
        $values = implode(",", $value);

        $sql = "INSERT INTO " . $table . " (" . $columns . ") VALUES (" . $values . ")";
        return $this->conn->query($sql);
    }

    public function update($table, $data, $where)
    {
        foreach ($data as $key => $val) {
            $update_value[] = "$key='{$val}'";
        }

        $update_value = implode(",", $update_value);

        $sql = "UPDATE " . $table . " SET " . $update_value . " WHERE " . $where;
        return $this->conn->query($sql);
    }
}
?>
~~~

2. Form

~~~
<?php

/**
 * Nama Class: Form
 * Deskripsi: Class untuk membuat form inputan dinamis
 */

class Form
{
    private $fields = array();
    private $action;
    private $submit = "Submit Form";
    private $jumField = 0;

    public function __construct($action, $submit)
    {
        $this->action = $action;
        $this->submit = $submit;
    }

    public function displayForm()
    {
        echo "<form action='" . $this->action . "' method='POST'>";
        echo '<table width="100%" border="0">';

        foreach ($this->fields as $field) {
            echo "<tr><td align='right' valign='top'>" . $field['label'] . "</td>";
            echo "<td>";

            switch ($field['type']) {

                case 'textarea':
                    echo "<textarea name='" . $field['name'] . "' cols='30' rows='4'></textarea>";
                    break;

                case 'select':
                    echo "<select name='" . $field['name'] . "'>";
                    foreach ($field['options'] as $value => $label) {
                        echo "<option value='" . $value . "'>" . $label . "</option>";
                    }
                    echo "</select>";
                    break;

                case 'radio':
                    foreach ($field['options'] as $value => $label) {
                        echo "<label><input type='radio' name='" . $field['name'] . "' value='" . $value . "'> " . $label . "</label> ";
                    }
                    break;

                case 'checkbox':
                    foreach ($field['options'] as $value => $label) {
                        echo "<label><input type='checkbox' name='" . $field['name'] . "[]' value='" . $value . "'> " . $label . "</label> ";
                    }
                    break;

                case 'password':
                    echo "<input type='password' name='" . $field['name'] . "'>";
                    break;

                default:
                    echo "<input type='text' name='" . $field['name'] . "'>";
                    break;
            }

            echo "</td></tr>";
        }

        echo "<tr><td colspan='2'>";
        echo "<input type='submit' value='" . $this->submit . "'></td></tr>";
        echo "</table>";
        echo "</form>";
    }

    public function addField($name, $label, $type = "text", $options = array())
    {
        $this->fields[$this->jumField]['name'] = $name;
        $this->fields[$this->jumField]['label'] = $label;
        $this->fields[$this->jumField]['type'] = $type;
        $this->fields[$this->jumField]['options'] = $options;
        $this->jumField++;
    }
}
?>
~~~

3. Index 1

~~~
<?php
$db = new Database();
$data = $db->query("SELECT * FROM artikel");
?>

<h2>Daftar Artikel</h2>

<table border="1" width="100%" cellpadding="6">
    <tr>
        <th>ID</th>
        <th>Judul</th>
        <th>Aksi</th>
    </tr>

    <?php while ($row = $data->fetch_assoc()) { ?>
    <tr>
        <td><?= $row['id'] ?></td>
        <td><?= $row['judul'] ?></td>
        <td>
            <a href="/lab11_php_oop/artikel/ubah?id=<?= $row['id'] ?>">Ubah</a>
        </td>
    </tr>
    <?php } ?>
</table>
~~~

4. Tambah

~~~
<?php
$db = new Database();
$form = new Form("/Lab11Web/artikel/tambah", "Simpan");

if ($_POST) {
    $data = [
        'judul' => $_POST['judul'],
        'isi' => $_POST['isi']
    ];

    $db->insert("artikel", $data);
    echo "<p style='color:green'>Artikel berhasil ditambahkan!</p>";
}
?>

<h2>Tambah Artikel</h2>

<?php
$form->addField("judul", "Judul Artikel");
$form->addField("isi", "Isi Artikel", "textarea");
$form->displayForm();
?>
~~~

5. Ubah

~~~
<?php
$db = new Database();

$id = $_GET['id'];
$old = $db->get("artikel", "id=$id");

$form = new Form("/lab11_php_oop/artikel/ubah?id=$id", "Update");

if ($_POST) {
    $data = [
        'judul' => $_POST['judul'],
        'isi' => $_POST['isi']
    ];

    $db->update("artikel", $data, "id=$id");
    echo "<p style='color:green'>Artikel berhasil diperbarui!</p>";
}
?>

<h2>Ubah Artikel</h2>

<?php
$form->addField("judul", "Judul", "text");
$form->addField("isi", "Isi", "textarea");
$form->displayForm();
?>
~~~

6. footer

~~~
<hr>
<footer>
    <p>Modular Routing — Praktikum 11</p>
</footer>
</body>
</html>
~~~

7. Header

~~~
<!DOCTYPE html>
<html>
<head>
    <title>Lab 11 PHP OOP</title>
    <style>
        body { font-family: Arial; padding: 20px; }
        .menu a { margin-right: 20px; }
    </style>
</head>
<body>

<div class="menu">
    <a href="/Lab11Web/home/index">Home</a>
    <a href="/Lab11Web/artikel/index">Artikel</a>
    <a href="/Lab11Web/artikel/tambah">Tambah Artikel</a>
</div>
<hr>
~~~

8. .htaccess

~~~
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /Lab11Web/

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f

    RewriteRule ^(.*)$ index.php/$1 [L]
</IfModule>
~~~

9. Config

~~~
<?php
$config = [
    'host' => 'localhost',
    'username' => 'root',
    'password' => '',
    'db_name' => 'latihan_oop'
];
?>
~~~

10. Index 2

~~~
<?php
include "config.php";
include "class/Database.php";
include "class/Form.php";

session_start();

// Ambil PATH_INFO → /artikel/tambah
$path = isset($_SERVER['PATH_INFO']) ? $_SERVER['PATH_INFO'] : '/home/index';

$segments = explode('/', trim($path, '/'));

$mod  = $segments[0] ?? 'home';
$page = $segments[1] ?? 'index';

$file = "module/{$mod}/{$page}.php";

include "template/header.php";

if (file_exists($file)) {
    include $file;
} else {
    echo "<h3>Modul tidak ditemukan: $mod / $page</h3>";
}

include "template/footer.php";
~~~
















