---
title: Email Verification
weight: 1
---

# Email Verification

![](https://res.cloudinary.com/dl4murstw/image/upload/v1645158360/ff88b03d-5f21-45df-9639-ea12aebc5b16_uaglrz.jpg)

The 'email verification' functionality is essentially a part of the onboarding process, where a new user gets a email verification upon sign up. The way this is constructed is in two parts.

1. A `POST` request sent to api/email route upon clicking the sign up button, where the controller creates a user and sends an email to the user with the verification link.

2. A `GET` request that is sent when the user clicks on the verification link to 'confirm/:id' of route of the email router. This alters the 'confirmed' field of the database record of said user to be `true`, and upon confirmation, redirects the user to the 'verified' page on frontend.

## Part 1: Post request + email sending

The main files in which the frontend logic is contained in is `signUp.js`. Within it, we see the data being sent within the handleSubmit() function. There is a state within the component called `signUpSuccess`. If the server responds with a `200` after we make an axios post request to `api/email`, `signUpSuccess` is set to true, and JSX renders a page asking the user to click on verification link.

The backend router responsible for creating the 'api/email' route is 'email.js'. The responsibility of this route is to hash the password, create the User record in the database, and send the email.

The sendEmail feature is built using the sendEmail function, which takes two arguments: User email, and an email template. The sendEmail function is imported from `email.send.js`. This file requires nodemailer, and dotenv. The sender email configuration is defined within this file, pulling the email user and pass from .env file. It uses the transporter object created from nodemailer to send email, by invoking the .sendEmail() function available to the object. Check out email.send.js for more details.

That's about it for this part!.

# Part 2: Confirmation link
The user then clicks on the confirmation link sent to their email. The confirmation link directs the user basically to `api/email/confirm/:id`. The route, stored in 'email.js' on the backend as well, changes the confirmed field for the user to true in the database, and redirects the user to the 'verified' page on the frontend.

Simple.

There we have an email verification feature! What's perhaps worth pointing out is that this system design leverages on routes in the backend to create user, send email, and subsequently verify. The front-end client is responsible for displaying pages only.
