1. Basic Setup

1. Create a new PHP project named `school_demo`.

2. Set up your project directory structure. It might look something like this:

   ```

   school_demo/

   ├── create.php

   ├── delete.php

   ├── edit.php

   ├── index.php

   ├── view.php

   ├── classes.php

   ├── uploads/

   ├── css/

   └── db.php

   ```



### 2. Database

1. Create a MySQL database named `school_db`.



2. Create the `student` table:

   ```sql

   CREATE TABLE student (

       id INT PRIMARY KEY AUTO_INCREMENT,

       name VARCHAR(255) NOT NULL,

       email VARCHAR(255),

       address TEXT,

       class_id INT,

       image VARCHAR(255),

       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

       FOREIGN KEY (class_id) REFERENCES classes(class_id)

   );

   ```



3. Create the `classes` table:

   ```sql

   CREATE TABLE classes (

       class_id INT PRIMARY KEY AUTO_INCREMENT,

       name VARCHAR(255) NOT NULL,

       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP

   );

   ```



### 3. Functionality



#### Home Page (index.php)

```php

<?php

include 'db.php';



$query = "SELECT student.*, classes.name AS class_name 

          FROM student 

          JOIN classes ON student.class_id = classes.class_id";

$result = mysqli_query($conn, $query);

?>

<!DOCTYPE html>

<html>

<head>

    <link rel="stylesheet" href="css/styles.css">

</head>

<body>

    <h1>Students</h1>

    <a href="create.php">Create Student</a>

    <table>

        <tr>

            <th>Name</th>

            <th>Email</th>

            <th>Created At</th>

            <th>Class</th>

            <th>Image</th>

            <th>Actions</th>

        </tr>

        <?php while($row = mysqli_fetch_assoc($result)): ?>

        <tr>

            <td><?php echo $row['name']; ?></td>

            <td><?php echo $row['email']; ?></td>

            <td><?php echo $row['created_at']; ?></td>

            <td><?php echo $row['class_name']; ?></td>

            <td><img src="uploads/<?php echo $row['image']; ?>" alt="Image" width="50"></td>

            <td>

                <a href="view.php?id=<?php echo $row['id']; ?>">View</a>

                <a href="edit.php?id=<?php echo $row['id']; ?>">Edit</a>

                <a href="delete.php?id=<?php echo $row['id']; ?>">Delete</a>

            </td>

        </tr>

        <?php endwhile; ?>

    </table>

</body>

</html>

```



#### Create Student (create.php)

```php

<?php

include 'db.php';



if ($_SERVER['REQUEST_METHOD'] == 'POST') {

    $name = $_POST['name'];

    $email = $_POST['email'];

    $address = $_POST['address'];

    $class_id = $_POST['class_id'];

    $image = $_FILES['image']['name'];

    $target = "uploads/" . basename($image);



    // Validate image

    $allowed_types = ['image/jpeg', 'image/png'];

    $image_type = mime_content_type($_FILES['image']['tmp_name']);



    if (!empty($name) && in_array($image_type, $allowed_types)) {

        if (move_uploaded_file($_FILES['image']['tmp_name'], $target)) {

            $query = "INSERT INTO student (name, email, address, class_id, image) 

                      VALUES ('$name', '$email', '$address', '$class_id', '$image')";

            mysqli_query($conn, $query);

            header('Location: index.php');

        }

    }

}



$classes_result = mysqli_query($conn, "SELECT * FROM classes");

?>

<!DOCTYPE html>

<html>

<head>

    <link rel="stylesheet" href="css/styles.css">

</head>

<body>

    <h1>Create Student</h1>

    <form method="post" enctype="multipart/form-data">

        <label>Name:</label><input type="text" name="name" required><br>

        <label>Email:</label><input type="email" name="email"><br>

        <label>Address:</label><textarea name="address"></textarea><br>

        <label>Class:</label>

        <select name="class_id">

            <?php while($class = mysqli_fetch_assoc($classes_result)): ?>

                <option value="<?php echo $class['class_id']; ?>"><?php echo $class['name']; ?></option>

            <?php endwhile; ?>

        </select><br>

        <label>Image:</label><input type="file" name="image" required><br>

        <button type="submit">Create</button>

    </form>

</body>

</html>

```



#### View Student (view.php)

```php

<?php

include 'db.php';

$id = $_GET['id'];

$query = "SELECT student.*, classes.name AS class_name 

          FROM student 

          JOIN classes ON student.class_id = classes.class_id 

          WHERE student.id = $id";

$result = mysqli_query($conn, $query);

$student = mysqli_fetch_assoc($result);

?>

<!DOCTYPE html>

<html>

<head>

    <link rel="stylesheet" href="css/styles.css">

</head>

<body>

    <h1>View Student</h1>

    <p>Name: <?php echo $student['name']; ?></p>

    <p>Email: <?php echo $student['email']; ?></p>

    <p>Address: <?php echo $student['address']; ?></p>

    <p>Class: <?php echo $student['class_name']; ?></p>

    <p>Created At: <?php echo $student['created_at']; ?></p>

    <p><img src="uploads/<?php echo $student['image']; ?>" alt="Image" width="100"></p>

    <a href="index.php">Back to Home</a>

</body>

</html>

```



#### Edit Student (edit.php)

```php

<?php

include 'db.php';

$id = $_GET['id'];



if ($_SERVER['REQUEST_METHOD'] == 'POST') {

    $name = $_POST['name'];

    $email = $_POST['email'];

    $address = $_POST['address'];

    $class_id = $_POST['class_id'];

    $image = $_FILES['image']['name'];

    $target = "uploads/" . basename($image);



    $query = "UPDATE student SET 

              name='$name', email='$email', address='$address', class_id='$class_id'";



    if (!empty($image)) {

        $image_type = mime_content_type($_FILES['image']['tmp_name']);

        $allowed_types = ['image/jpeg', 'image/png'];

        if (in_array($image_type, $allowed_types)) {

            if (move_uploaded_file($_FILES['image']['tmp_name'], $target)) {

                $query .= ", image='$image'";

            }

        }

    }

    

    $query .= " WHERE id=$id";

    mysqli_query($conn, $query);

    header('Location: index.php');

}



$query = "SELECT * FROM student WHERE id=$id";

$result = mysqli_query($conn, $query);

$student = mysqli_fetch_assoc($result);

$classes_result = mysqli_query($conn, "SELECT * FROM classes");

?>

<!DOCTYPE html>

<html>

<head>

    <link rel="stylesheet" href="css/styles.css">

</head>

<body>

    <h1>Edit Student</h1>

    <form method="post" enctype="multipart/form-data">

        <label>Name:</label><input type="text" name="name" value="<?php echo $student['name']; ?>" required><br>

        <label>Email:</label><input type="email" name="email" value="<?php echo $student['email']; ?>"><br>

        <label>Address:</label><textarea name="address"><?php echo $student['address']; ?></textarea><br>

        <label>Class:</label>

        <select name="class_id">

            <?php while($class = mysqli_fetch_assoc($classes_result)): ?>

                <option value="<?php echo $class['class_id']; ?>" <?php if ($student['class_id'] == $class['class_id']) echo 'selected'; ?>>

                    <?php echo $class['name']; ?>

                </option>

            <?php endwhile; ?>

        </select><br>

        <label>Image:</label><input type="file" name="image"><br>

        <button type="submit">Update</button>

    </form>

</body>

</html>

```



#### Delete Student (delete.php)

```php

<?php

include 'db.php';

$id = $_GET['id'];



if ($_SERVER['REQUEST_METHOD'] == 'POST') {

    $query = "SELECT image FROM student WHERE id=$id";

    $result = mysqli_query($conn, $query);

    $student = mysqli_fetch_assoc($result);



    if ($student['image']) {

        unlink("uploads/" . $student['image']);

    }



    $query = "DELETE FROM student WHERE id=$id";

    mysqli_query($conn, $query);

    header('Location: index.php');

}



$query = "SELECT * FROM



 student WHERE id=$id";

$result = mysqli_query($conn, $query);

$student = mysqli_fetch_assoc($result);

?>

<!DOCTYPE html>

<html>

<head>

    <link rel="stylesheet" href="css/styles.css">

</head>

<body>

    <h1>Delete Student</h1>

    <p>Are you sure you want to delete <?php echo $student['name']; ?>?</p>

    <form method="post">

        <button type="submit">Yes</button>

        <a href="index.php">No</a>

    </form>

</body>

</html>

```



#### Manage Classes (classes.php)

```php

<?php

include 'db.php';



if ($_SERVER['REQUEST_METHOD'] == 'POST') {

    if (isset($_POST['class_name'])) {

        $class_name = $_POST['class_name'];

        $query = "INSERT INTO classes (name) VALUES ('$class_name')";

        mysqli_query($conn, $query);

    } elseif (isset($_POST['edit_class_id'])) {

        $class_id = $_POST['edit_class_id'];

        $class_name = $_POST['edit_class_name'];

        $query = "UPDATE classes SET name='$class_name' WHERE class_id=$class_id";

        mysqli_query($conn, $query);

    } elseif (isset($_POST['delete_class_id'])) {

        $class_id = $_POST['delete_class_id'];

        $query = "DELETE FROM classes WHERE class_id=$class_id";

        mysqli_query($conn, $query);

    }

    header('Location: classes.php');

}



$classes_result = mysqli_query($conn, "SELECT * FROM classes");

?>

<!DOCTYPE html>

<html>

<head>

    <link rel="stylesheet" href="css/styles.css">

</head>

<body>

    <h1>Manage Classes</h1>

    <form method="post">

        <label>New Class Name:</label><input type="text" name="class_name" required><br>

        <button type="submit">Add Class</button>

    </form>

    <table>

        <tr>

            <th>Class Name</th>

            <th>Actions</th>

        </tr>

        <?php while($class = mysqli_fetch_assoc($classes_result)): ?>

        <tr>

            <td><?php echo $class['name']; ?></td>

            <td>

                <form method="post" style="display:inline;">

                    <input type="hidden" name="delete_class_id" value="<?php echo $class['class_id']; ?>">

                    <button type="submit">Delete</button>

                </form>

                <form method="post" style="display:inline;">

                    <input type="hidden" name="edit_class_id" value="<?php echo $class['class_id']; ?>">

                    <input type="text" name="edit_class_name" value="<?php echo $class['name']; ?>" required>

                    <button type="submit">Edit</button>

                </form>

            </td>

        </tr>

        <?php endwhile; ?>

    </table>

</body>

</html>

```



### Image Upload Handling

- Make sure the `uploads` directory is writable by the server.

- Add validation checks for the image format and unique filename generation (e.g., prepend a timestamp or a unique identifier).



### Styling

- Add a `css/styles.css` file to style your application.

- You can use Bootstrap for better styling. Include Bootstrap CSS in your `<head>` tags:

  ```html

  <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">

  ```



### Database Connection (db.php)

```php

<?php

$conn = new mysqli('localhost', 'username', 'password', 'school_db');



if ($conn->connect_error) {

    die("Connection failed: " . $conn->connect_error);

}

?>

```



