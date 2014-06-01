.. ==================================================
.. FOR YOUR INFORMATION
.. --------------------------------------------------
.. -*- coding: utf-8 -*- with BOM.

.. include:: ../Includes.txt


.. _introduction:

Introduction
============


.. _what-it-does:

What does it do?
----------------

This extension allows to use the spam protection provided by the wt_spamshield_ extension to forms handled by the formhandler_ extension.
Most specifically, it provides the following spam checks:

#. Link check: You can configure how many links are allowed within a message.
#. Unique check: Specified fields are checked for duplicate entries.
#. Honeypot check: A non-visible input field is added to your form. If the field is filled the message is handled as spam.
#. Akismet check: Akismet is a reliable and powerful online check to determine if a message is spam.
#. Blacklist: You can set up IP or email blacklists.

The spam check is implemented as a formhandler Interceptor. Each spam check can be enabled or disabled, globally or on each individual form.


.. _wt_spamshield: http://typo3.org/extensions/repository/view/wt_spamshield
.. _formhandler: http://www.typo3-formhandler.com/
