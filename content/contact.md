---
title: "Contact"
date: 2018-09-21T22:07:06+02:00
draft: false
---

You can contact me via the form below:

{{< raw_html >}}
<form class="contact-form" name="contact" method="POST" data-netlify="true" data-netlify-recaptcha="true">
    <div class="contact-form-field">
        <label for="contact-name">Name</label>
        <input type="text" id="contact-name" name="name" autocomplete="name" required>
    </div>
    <div class="contact-form-field">
        <label for="contact-email">Email address</label>
        <input type="email" id="contact-email" name="email" autocomplete="email">
    </div>
    <div class="contact-form-field">
        <label for="contact-message">Message</label>
        <textarea id="contact-message" name="message" rows="6" required></textarea>
    </div>
    <div class="contact-form-field" data-netlify-recaptcha="true"></div>
    <button type="submit">Send message</button>
</form>
{{< /raw_html >}}
