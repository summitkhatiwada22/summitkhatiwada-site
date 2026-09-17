---
layout: page
title: contact
permalink: /contact/
nav: true
nav_order: 5
description: Get in touch.
_styles: |
  .post-title { display: none; }
  .post-description { font-size: 1rem; }
  #contact-form input, #contact-form textarea {
    width: 100%;
    max-width: 480px;
    padding: 0.6rem 0.8rem;
    margin-bottom: 1rem;
    background: var(--global-bg-color);
    color: var(--global-text-color);
    border: 1px solid var(--global-text-color-light);
    border-radius: 6px;
    font-family: inherit;
    font-size: 1rem;
    display: block;
  }
  #contact-form textarea { min-height: 140px; resize: vertical; }
  #contact-form button {
    padding: 0.6rem 1.4rem;
    background: var(--global-theme-color);
    color: var(--global-bg-color);
    border: none;
    border-radius: 6px;
    font-size: 1rem;
    cursor: pointer;
  }
  #contact-form button:hover { opacity: 0.85; }
---

The best way to reach me is here, or directly at [summitkhatiwada22@gmail.com](mailto:summitkhatiwada22@gmail.com).


<form id="contact-form" action="https://formsubmit.co/summitkhatiwada22@gmail.com" method="POST">
  <input type="hidden" name="_subject" value="New message from summitkhatiwada.com">
  <input type="hidden" name="_template" value="table">
  <input type="hidden" name="_captcha" value="false">
  <input type="text" name="_honey" style="display:none">

  <input type="text" name="name" placeholder="Your name" required>
  <input type="email" name="email" placeholder="Your email" required>
  <textarea name="message" placeholder="Message" required></textarea>
  <button type="submit">Send</button>
</form>