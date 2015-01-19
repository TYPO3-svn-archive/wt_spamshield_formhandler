.. ==================================================
.. FOR YOUR INFORMATION
.. --------------------------------------------------
.. -*- coding: utf-8 -*- with BOM.

.. include:: ../Includes.txt


.. _admin-manual:

Administrator Manual
====================

.. _admin-installation:

Installation
------------

This extension depends on both wt_spamshield and formhandler extensions. Thus you need to install them before installing `wt_spamshield_formhandler`.

You need to install static templates from both `wt_spamshield` and `wt_spamshield_formhandler`. 
`wt_spamshield` static template should be loaded before `wt_spamshield_formhandler` static template.


.. _admin-configuration:

Configuration
-------------

All the configuration is performed using TypoScript. There are global options you can configure for all forms,
and there are options you can configure for specific form. Global options are accessible through setup and constants
of `wt_spamshield`, while local options are configurable on the TypoScript of the form. Local options always
override global options.

Because of the versatility of `formhandler`, `wt_spamshield` protection is not automatically enabled,
but need to be configured on each form.


Global options
^^^^^^^^^^^^^^

Configuration options described here are specific to `wt_spamshield_formhandler`. 
Please referer to `wt_spamshield manual`_ for other configuration options.

.. _wt_spamshield manual: http://docs.typo3.org/typo3cms/extensions/wt_spamshield/

TypoScript constants
""""""""""""""""""""

All the constants are configurable using constant editor, under `WT_SPAMSHIELD MAIN` category.

**plugin.wt_spamshield.formhandler**
  If *false*, wt_spamshield protection is disabled for all forms (default: *true*)

**plugin.wt_spamshield.validators.formhandler.enable**
  Comma separated list of enabled validators. Can be: blacklistCheck, httpCheck, uniqueCheck, honeypotCheck, akismetCheck. By default, all validators are enabled.

**plugin.wt_spamshield.formhandler.how_many_validators_can_fail**
  Number of tests that can fail before reporting the form as spam. For example, if it is 1, there must be at least 2 checks that fail to report the form as spam. Default: 0.

**plugin.wt_spamshield.honeypot.inputname.formhandler**
  Name of the field used for the honeypot. Default: uid987651.

TypoScript setup
""""""""""""""""

**plugin.wt_spamshield.fields.formhandler**
	Default names of the form fields used for Akismet check. See Akismet configuration details below for more information.



Formhandler Interceptor configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In order to enable `wt_spamshield` protection on a `formhandler` form, you need to configure a Save Interceptor in the form TypoScript configuration.
Here is an example:

.. code-block:: typoscript

    plugin.Tx_Formhandler.settings {
        ...
        saveInterceptors {
            1.class = Tx_WtSpamshieldFormhandler_Interceptor_WtSpamshield
            1.disable = 0
            1.config {
                redirectPage = 100
                loggers {
                    1.class = Logger_DB
                }
                akismetCheck.fields {
                    author = name
                    email = email
                    homepage = 
                    body = message
                    permalink = 
                }
                httpCheck.maximumLinkAmount = 1
                uniqueCheck.fields = name,email
            }
        }
    }

The only mandatory configuration option is the **class**, since it tells `formhandler` to use the WtSpamshield Interceptor.
All the configuration options for the interceptor go inside the **config** sections. There are two kinds of options:
those who configure the formhandler interceptor, and those who configure the spamshield protection.

Interceptor configuration options
"""""""""""""""""""""""""""""""""

You should at least configure either **redirectPage** or **templateFile**.

**redirectPage**
  Page id or url where to redirect user, if spam is detected. If configured, the user will be redirected to this page if the spam check fail.

**correctRedirectUrl**
  Replaces  '&amp;' with '&' in URL

**additionalParams**
  Adds additional parameters to the redirect URL. Each parameter can be a string or a TypoScript object.

**headerStatusCode**
  Set a custom header code set when redirecting to another page.

**templateFile**
  Path to template file or template code to display if spam check fail.

**loggers**
  You can configure formhandler loggers here, to log data when spam is detected. Otherwise, form data is discarded.
  See `formhandler Logger documentation`_ for more details. If you enable Loggers, you should consider disabling
  the logging already performed by `wt_spamshield`; otherwise you will log spam twice.

**validators.enable**
  Comma separated list of validators to use. Override the global option. Use this if you want to enable or disable some specific validators for this form.
  Possible validators are: blacklistCheck, httpCheck, uniqueCheck, honeypotCheck, akismetCheck.

**validators.how_many_validators_can_fail**
  Use this to override the default number of validators that can fail, for this specific form.

.. _formhandler Logger documentation: http://www.typo3-formhandler.com/documentation/loggers/


Blacklist configuration
"""""""""""""""""""""""

To blacklist email of IP addresses, the only thing you need to do is to add some spamshield **blacklist** records somewhere. 
Off course, **blacklistCheck** must be in the list of enabled validators.


HTTP configuration
""""""""""""""""""

This validator counts the total number of links present in all fields of the form. If this number is greater than **maximumLinkAmount** (default is 3),
the form is reported as spam. The definition of links is everything than contains http://, https://, or ftp.

**httpCheck.maximumLinkAmount**
  Number of links allowed in the form.


Unique configuration
""""""""""""""""""""

This validator ensure that given fields does not contain the same value. You can specify different groups of fields to check for uniqueness.

**uniqueCheck.fields**
  Each field are separated by a comma. Each independent groups of fields are separated by a semi-column. For instance,
  if uniqueCheck.fields = firstname,lastname;field1,field2,field3 , firstname and lastname must be different, as well as any
  value in field1, field2, and field3, but firstname and field1 could be the same value.


Honeypot configuration
""""""""""""""""""""""

The principle of the honeypot is to add a field that is not visible to the user, but is visible to bots. If there is any value
filled into the honeypot field, the form is reported as spam.

**honeypotCheck.inputname**
  The name of the input used for the honeypot check. You must ensure there is a field with this name somewhere in the form.

To include the honeypot field into the form, you can simply add the ###HONEYPOT### marker at the beginning of you form template.
This marker is defined in **plugin.Tx_Formhandler.settings.markers.HONEYPOT** TypoScript object.
Most of the parameters of this field can be configured using TypoScript constants of wt_spamshield.
If you locally change the value of **honeypotCheck.inputname** and you use the ###HONEYPOT### marker, you must 
modify it to reflect the change. It can be done by putting something like

.. code-block:: typoscript

    markers.HONEYPOT < plugin.Tx_Formhandler.settings.markers.HONEYPOT
    markers.HONEYPOT.value < plugin.Tx_Formhandler.settings.saveInterceptors.1.config.honeypotCheck.inputname

in you form TypoScript configuration. If you defined a custom **formValuePrefix** in you form, you should also
modify the HONEYPOT marker like:

.. code-block:: typoscript

    markers.HONEYPOT.value.prepend.value < .formValuesPrefix

Instead of using the HONEYPOT marker, it is also possible to manually add the honeypot field to your form. In that
case, be sure to use the same name for the input as you defined in **honeypotCheck.inputname**, and be sure
to use the right styles to hide the honeypot field in the form.


Akismet configuration
"""""""""""""""""""""

The first thing you need is to configure your Akismet key. You must register on akismet_ to get a key.
Then, you must enter you key in the **plugin.wt_spamshield.akismetCheck.akismetKey** TypoScript constant
of wt_spamshield. 

The most fields you use, the most precise the check will be. Unused fields can be left blank.

**akismetCheck.fields.author**
    Field to use as author name

**akismetCheck.fields.email**
    Field to use as author email

**akismetCheck.fields.homepage**
    Field to use as author web page

**akismetCheck.fields.body**
    Field to use as the body of the comment or the message

**akismetCheck.fields.permalink**
    Field to use as the permalink for the entered comment



.. _akismet: http://akismet.com/




.. _admin-faq:

.. FAQ
.. ---

.. Possible subsection: FAQ

.. Subsection
.. ^^^^^^^^^^

.. Some subsection

.. Sub-subsection
.. """"""""""""""

.. Deeper into the structure...
