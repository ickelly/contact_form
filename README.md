contact_form
============

contact form from tutorial by Matt West (original post is here: http://blog.teamtreehouse.com/create-ajax-contact-form)


CONTACT.HTML

<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>AJAX Contact Form Demo</title>
	<link rel="stylesheet" href="style.css">
</head>
<body>
	<div id="page-wrapper">
	  <h1>AJAX Contact Form Demo</h1>
	  <div id="form-messages"></div>
		<form id="ajax-contact" method="post" action="mailer.php">
			<div class="field">
				<label for="name">Name:</label>
					<input type="text" id="name" name="name" required>
			</div>
			<div class="field">
				<label for="email">Email:</label>
				<input type="email" id="email" name="email" required>
			</div>
			<div class="field">
				<label for="message">Message:</label>
				<textarea id="message" name="message" required></textarea>
			</div>
			<div class="field">
				<button type="submit">Send</button>
			</div>
		</form>
	</div>
	<script src="jquery-2.1.0.min.js"></script>
	<script src="app.js"></script>
</body>
</html>

STYLE.CSS

*, *:before, *:after {
  -moz-box-sizing: border-box;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
}

html {
  font-family: Helvetica, Arial, sans-serif;
  font-size: 100%;
  background: #333;
  -webkit-font-smoothing: antialiased;
}

#page-wrapper {
  width: 640px;
  background: #FFFFFF;
  padding: 1em;
  margin: 1em auto;
  border-top: 5px solid #69c773;
  box-shadow: 0 2px 10px rgba(0,0,0,0.8);
}

h1 {
  margin-top: 0;
}

.field {
  margin: 1em 0;
}

label {
  display: block;
  margin-top: 2em;
  margin-bottom: 0.5em;
  color: #999999;
}

input {
  width: 100%;
  padding: 0.5em 0.5em;
  font-size: 1.2em;
  border-radius: 3px;
  border: 1px solid #D9D9D9;
}

textarea {
  width: 100%;
  height: 200px;
  padding: 0.5em 0.5em;
  font-size: 1.2em;
  border-radius: 3px;
  border: 1px solid #D9D9D9;
}

button {
  display: inline-block;
  border-radius: 3px;
  border: none;
  font-size: 0.9rem;
  padding: 0.5rem 0.8em;
  background: #69c773;
  border-bottom: 1px solid #498b50;
  color: white;
  -webkit-font-smoothing: antialiased;
  font-weight: bold;
  margin: 0;
  width: 27%;
  text-align: center;
}

button+button {
  margin-left: 1em;
}

button:hover, button:focus {
  opacity: 0.75;
  cursor: pointer;
}

button:active {
  opacity: 1;
  box-shadow: 0 -3px 10px rgba(0, 0, 0, 0.1) inset;
}

.success {
  padding: 1em;
  margin-bottom: 0.75rem;
  text-shadow: 0 1px 0 rgba(255, 255, 255, 0.5);
  color: #468847;
  background-color: #dff0d8;
  border: 1px solid #d6e9c6;
  -webkit-border-radius: 4px;
     -moz-border-radius: 4px;
          border-radius: 4px;
}

.error {
  padding: 1em;
  margin-bottom: 0.75rem;
  text-shadow: 0 1px 0 rgba(255, 255, 255, 0.5);
  color: #b94a48;
  background-color: #f2dede;
  border: 1px solid rgba(185, 74, 72, 0.3);
  -webkit-border-radius: 4px;
     -moz-border-radius: 4px;
          border-radius: 4px;
}


JQUERY-2.1.0.MIN.JS

APP.JS

$(function() {

	// Get the form.
	var form = $('#ajax-contact');

	// Get the messages div.
	var formMessages = $('#form-messages');

	// Set up an event listener for the contact form.
	$(form).submit(function(e) {
		// Stop the browser from submitting the form.
		e.preventDefault();

		// Serialize the form data.
		var formData = $(form).serialize();

		// Submit the form using AJAX.
		$.ajax({
			type: 'POST',
			url: $(form).attr('action'),
			data: formData
		})
		.done(function(response) {
			// Make sure that the formMessages div has the 'success' class.
			$(formMessages).removeClass('error');
			$(formMessages).addClass('success');

			// Set the message text.
			$(formMessages).text(response);

			// Clear the form.
			$('#name').val('');
			$('#email').val('');
			$('#message').val('');
		})
		.fail(function(data) {
			// Make sure that the formMessages div has the 'error' class.
			$(formMessages).removeClass('success');
			$(formMessages).addClass('error');

			// Set the message text.
			if (data.responseText !== '') {
				$(formMessages).text(data.responseText);
			} else {
				$(formMessages).text('Oops! An error occured and your message could not be sent.');
			}
		});

	});

});

MAILER.PHP

<?php
    // My modifications to mailer script from:
    // http://blog.teamtreehouse.com/create-ajax-contact-form
    // Added input sanitizing to prevent injection

    // Only process POST reqeusts.
    if ($_SERVER["REQUEST_METHOD"] == "POST") {
        // Get the form fields and remove whitespace.
        $name = strip_tags(trim($_POST["name"]));
				$name = str_replace(array("\r","\n"),array(" "," "),$name);
        $email = filter_var(trim($_POST["email"]), FILTER_SANITIZE_EMAIL);
        $message = trim($_POST["message"]);

        // Check that data was sent to the mailer.
        if ( empty($name) OR empty($message) OR !filter_var($email, FILTER_VALIDATE_EMAIL)) {
            // Set a 400 (bad request) response code and exit.
            http_response_code(400);
            echo "Oops! There was a problem with your submission. Please complete the form and try again.";
            exit;
        }

        // Set the recipient email address.
        // FIXME: Update this to your desired email address.
        $recipient = "recipient@email.com";

        // Set the email subject.
        $subject = "New contact from $name";

        // Build the email content.
        $email_content = "Name: $name\n";
        $email_content .= "Email: $email\n\n";
        $email_content .= "Message:\n$message\n";

        // Build the email headers.
        $email_headers = "From: $name <$email>";

        // Send the email.
        if (mail($recipient, $subject, $email_content, $email_headers)) {
            // Set a 200 (okay) response code.
            http_response_code(200);
            echo "Thank You! Your message has been sent.";
        } else {
            // Set a 500 (internal server error) response code.
            http_response_code(500);
            echo "Oops! Something went wrong and we couldn't send your message.";
        }

    } else {
        // Not a POST request, set a 403 (forbidden) response code.
        http_response_code(403);
        echo "There was a problem with your submission, please try again.";
    }

?>

