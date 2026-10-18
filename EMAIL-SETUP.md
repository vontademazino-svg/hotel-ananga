# Booking emails: setup guide (about 10 minutes)

When a guest books, the website sends two emails:

1. **To the guest:** booking reference, stay details, total, and how to pay (in the language the guest used on the site).
2. **To the hotel:** a "new booking" alert with the guest's contact details and a button to open the staff page.

A website on GitHub Pages can't send email by itself, so we use **EmailJS** (free plan: 200 emails per month). EmailJS sends the emails **from the hotel's own Gmail account**, so guests see them coming from the hotel.

> Menu names on the EmailJS website may differ slightly from this guide.

## 1. Create the account
1. Go to **emailjs.com** and click **Sign Up** (free).
2. Confirm your email address.

## 2. Connect the hotel's Gmail
1. In EmailJS, open **Email Services** → **Add New Service** → choose **Gmail**.
2. Click **Connect Account** and sign in with the hotel's Gmail. Allow EmailJS to send email.
3. Click **Create Service**.
4. Write down the **Service ID** (looks like `service_abc1234`).

## 3. Create the email template (one template for both emails)
1. Open **Email Templates** → **Create New Template**.
2. **Subject:** type exactly:
   ```
   {{subject}}
   ```
3. **Content:** switch the editor to **code / HTML** mode (the `<>` button), delete everything, and type exactly:
   ```
   {{{message_html}}}
   ```
   It must have **three** curly brackets on each side, so the email's layout shows correctly.
4. In the settings on the right:
   - **To Email:** `{{to_email}}`
   - **From Name:** `Hotel Ananga`
   - **Reply To:** `{{reply_to}}`
5. Click **Save**, then write down the **Template ID** (looks like `template_xyz7890`).

## 4. Get the public key
1. Open **Account** (top right) → **General**.
2. Write down the **Public Key**.

## 5. Put the details in the website
Open `booking.html`, find `EMAIL_SETTINGS`, and fill it in:

```js
const EMAIL_SETTINGS = {
  publicKey:  'your-public-key',
  serviceId:  'service_abc1234',
  templateId: 'template_xyz7890',
  hotelEmail: 'hotel@gmail.com',   // where new-booking alerts go
  hotelLang:  'pt'                 // language of the hotel's alert: 'pt' or 'en'
};
```

Or send these values to Claude and they will be filled in for you. They are safe to share: the public key is meant to be public.

Upload `booking.html` to GitHub, wait a minute, and make a test booking with your own email address. You should receive the guest email, and the hotel address should receive the alert.

## 6. Protect against misuse (recommended)
In EmailJS, look under **Account → Security** for a setting that limits sending to your own website (often called allowed origins or domains). If it's available, add:
```
https://teddy5590.github.io
```
That way, only your website can use your EmailJS account.

## Troubleshooting
- **No email arrives:** check the guest's spam folder. In EmailJS, open **Email History** to see whether it was sent or why it failed.
- **The email shows code instead of a nice layout:** the template content must be `{{{message_html}}}` with three brackets, in code/HTML mode.
- **The booking itself always works,** even if emails fail. Emails are sent after the booking is saved.
