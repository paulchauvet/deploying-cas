# Setting up CAS client pages

To start with - we'll be creating two php pages.  These will be supplemented later as we add other functionality like attribute release or MFA into the CAS serverbuild.

These two example php pages should be placed in your cas-client's templates directory.

### Create example content

The following should be called 'main-index.php' and placed in the templates directory.  When deployed by the template, it will be copied to /var/www/html/index.php.  It's just a collection of links that we will be adding to as we add functionality to the CAS server.

**roles/cas-client/templates/main-index.php:**

``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>CAS test page</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
  </head>
  <body>
    <div class="container">
      <h1>CAS test page</h1>
        <p><big>Click <a href="secured-by-cas/index.php">here</a> for some secure content.</big></p>
    </div>
  </body>
</html>
```

The following should be called 'basic-cas-check-index.php' and should also be placed in the templates directory.  When deployed by the template, it will be copied to /var/www/html/secured-by-cas/index.php.  It will display information on the logged in user for test purposes.

**roles/cas-client/templates/basic-cas-check-index.php:**
``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Hello, World!</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
  </head>
  <body>
    <div class="container">
      <h1>Secured Content</h1>
      <p><big>This is some secure content. You should not be able to see it
       until you have entered your username and password.</big></p>
      <h2>Attributes Returned by CAS</h2>
      <?php
        echo "<pre>";

        if (array_key_exists('REMOTE_USER', $_SERVER)) {
            echo "REMOTE_USER = " . $_SERVER['REMOTE_USER'] . "<br>";
        }

        $headers = getallheaders();
        foreach ($headers as $key => $value) {
            if (strpos($key, 'CAS_') === 0) {
                echo substr($key, 4) . " = " . $value . "<br>";
            }
        }

        echo "</pre>";
      ?>
    </div>
  </body>
</html>

```