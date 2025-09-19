## What I Had to Do

So basically Part 2 was about making the login system work. I had to:
- Make customers able to log in with their email and password
- Set up proper sessions when they log in
- Add a logout button 
- Make sure everything validates properly

## Files I Touched

### 1. New Login Action File
**File:** `actions/login_customer_action.php` - **I CREATED THIS**

This is basically the same pattern as my registration action. It gets the email and password from the form, checks if they're valid, and either logs the person in or tells them it failed.

The cool thing is when someone logs in successfully, I set a bunch of session variables so I can use their info anywhere in the app.

### 2. Updated Customer Class
**File:** `classes/customer_class.php` - **MODIFIED**

I added a new method to handle the actual login logic:

```php
public function loginCustomer($email, $password)
{
    // Get customer by email
    $customer = $this->getCustomerByEmail($email);
    
    if (!$customer) {
        return false; // Customer not found
    }
    
    // Verify password
    if (password_verify($password, $customer['customer_pass'])) {
        // Password is correct, return customer data (without password)
        unset($customer['customer_pass']); // Remove password from returned data
        return $customer;
    }
    
    return false; // Invalid password
}
```

This uses the `password_verify()` function to check if the password they typed matches what's in the database. Pretty secure stuff! I also make sure to remove the password from the data I send back because that would be bad security.

### 3. Updated Controller
**File:** `controllers/customer_controller.php` - **MODIFIED**

Just added a simple controller function that calls the class method:

```php
function login_customer_ctr($email, $password)
{
    $customer = new Customer();
    return $customer->loginCustomer($email, $password);
}
```

Same pattern as my registration controller - keeps things consistent.

### 4. New Login JavaScript
**File:** `js/login.js` - **I CREATED THIS**

This handles all the form validation and submission. I basically copied the structure from my `register.js` but simplified it since login only needs email and password:


### 5. Updated Login Page
**File:** `login/login.php` - **MODIFIED**

I just added a bit to show a success message when someone logs out:

```php
<!-- Alert Messages -->
<?php
if (isset($_GET['message'])) {
    if ($_GET['message'] == 'logged_out') {
        echo '<div class="alert alert-success text-center animate__animated animate__fadeIn">You have been logged out successfully!</div>';
    }
}
?>
```
Simple but effective!

### 6. Updated Main Page
**File:** `index.php` - **MODIFIED**

This was the fun part! I made the menu change depending on whether someone is logged in or not:

```php
<?php
session_start();
$isLoggedIn = isset($_SESSION['logged_in']) && $_SESSION['logged_in'] === true;
?>

<div class="menu-tray">
    <span class="me-2">Menu:</span>
    <?php if ($isLoggedIn): ?>
        <span class="text-muted me-2">Welcome, <?php echo htmlspecialchars($_SESSION['name']); ?>!</span>
        <a href="login/logout.php" class="btn btn-sm btn-outline-danger">Logout</a>
    <?php else: ?>
        <a href="login/register.php" class="btn btn-sm btn-outline-primary">Register</a>
        <a href="login/login.php" class="btn btn-sm btn-outline-secondary">Login</a>
    <?php endif; ?>
</div>
```

So now when someone logs in, they see "Welcome, [their name]!" and a logout button instead of the register/login buttons.

I also made the main content change to show their account info when they're logged in.

### 7. Fixed Logout
**File:** `login/logout.php` - **MODIFIED**

The original logout was pretty basic, so I made it properly destroy the session:

```php
<?php
session_start();

// Destroy session
$_SESSION = array();
session_destroy();

// Redirect to login page with a logout message
header("Location: login.php?message=logged_out");
exit();
?>
```

Now when someone logs out, it clears everything and takes them back to the login page with a nice message.


## How Everything Works Together

1. User fills out login form
2. JavaScript validates the form
3. AJAX sends data to `login_customer_action.php`
4. Action calls controller, controller calls model
5. Model checks password against database
6. If good: set session variables and return success
7. If bad: return error message
8. JavaScript handles the response and redirects or shows error
9. If logged in: index page shows personalized content and logout button

Pretty straightforward! The whole thing follows the same pattern as registration but simplified for just login/logout.

That's it! Everything works together and follows the same coding style I used in Part 1. The login system is now fully functional and secure.
