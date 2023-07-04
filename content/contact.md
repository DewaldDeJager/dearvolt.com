---
title: "Contact"
date: 2018-09-21T22:07:06+02:00
draft: false
---

You can contact me via the form below:

{{< raw_html >}}
<form name="contact" netlify>
    <label>
        Name:<br>
        <input type="text" name="name" required>
    </label>
    <br>
    <label>
        Email Address:<br>
        <input type="email" name="email">
    </label>
    <br>
    <label>
        Message:<br>
        <textarea name="message" required></textarea>
    </label>
    <br>
    <button type="submit">Send</button>
</form>
{{< /raw_html >}}