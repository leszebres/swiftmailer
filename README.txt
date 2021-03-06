The Swift Mailer module extends the basic e-mail sending functionality provided
by Drupal by delegating all e-mail handling to the Swift Mailer library. This
enables your site to take advantage of the many features which the Swift Mailer
library provides, such as :

- Sending e-mails directly through a SMTP server of your choice, a locally
  installed MTA agent such as sendmail or the mail functionality
  provided by PHP.
- Sending HTML e-mails.
- Adding file attachments to e-mails.
- Adding inline images to e-mails.

The module also lets you theme e-mails so that they reflect the general look and
feel of your site and at the same time appear correctly in various e-mail
browsers. Images required by the theme can be attached to the e-mail and
displayed inline.

The Swift Mailer module depends on the mailsystem module. This module is
responsible for actually making the Swift Mailer module available to Drupal.

1.0 Configuration

Please go to 'admin/config/swiftmailer/transport' to configure the Swift Mailer
module.

The module requires you to download the Swift Mailer library with composer. You
should require the module itself with composer. Please the the documentation on
Drupal.org for further information on this topic.

https://www.drupal.org/docs/develop/using-composer/using-composer-to-manage-drupal-site-dependencies#managing-contributed

After the module has been configured with the Swift Mailer library you are
advised to make sure that the Swift Mailer library sends e-mails using the
right transport option. You can choose between SMTP, sendmail (or any other
locally installed MTA) and PHP's mail() function. Please observe the various
configuration options which are available for each of the transport options.

Tip! You are advised to configure Swift Mailer to send e-mails using a locally
     installed MTA if performance is a concern. A locally installed MTA will
     most often accept e-mails right away and queue them up for sending. Your
     users will thus not have to wait for an SMTP server to respond back with
     wether the e-mail will be sent or for some reason was rejected.

You also have the chance to decide whether e-mail messages should be sent as
plain text or HTML by default. Swift Mailer will also, if configured, respect
any e-mail format dictated by the e-mail. Furthermore, you can also configure
the character set which is to be used as default. You are advised not to change
any of these options if you are uncertain about what they mean.

Finally, you can test whether the Swift Mailer library sends e-mail messages
correctly when the module has been configured and you are ready to put it into
action.

2.0 Theming

All e-mails sent by the SwiftMailer module can be themed. This is useful when
e-mails should reflect the look and feel of the website it was sent from. The
Swift Mailer modules delegates theming to the Drupal theming system. This means
that you can theme e-mails the same way you theme other content.

2.1 CSS

The theme hook you need to use when interacting with the theming system is
'swiftmailer'.  You can set the CSS files to use for mail messages using the
Drupal libraries configuration mechanism.

2.1.1. In your theme.
For example, the following code will add CSS from my_theme.mail.css to all HTML
mails. CSS is automatically inlined for wider compatibility with mail clients.

-- start of 'my_theme.libraries.yml' --

swiftmailer:
  css:
    theme:
      css/my_theme.mail.css: {}

-- end of 'my_theme.libraries.yml' --

2.1.2. In your module.
We can add styles to emails generated by a certain module.

-- start of 'my_module.module' --

/**
 * Implements hook_preprocess_HOOK().
 */
function MYMODULE_preprocess_swiftmailer(&$variables) {
  // Process for emails by this module.
  if ($variables['message']['module'] !== 'MYMODULE') {
    return;
  }

  // Add own styles with a library.
  $variables['#attached']['library'][] = 'MYMODULE/email_styles';
}

-- end of 'my_module.module' --

2.2 Template

The easiest way to theme e-mails is to create a file named
'swiftmailer.html.twig' in your theme folder. Please see section 2.1.1 for an
overview of the variables you can use in the 'swiftmailer.html.twig' file. The
theme file should hold markup which wraps the actual content of the message. The
below block of code demonstrates what a 'swiftmailer.html.twig' might look like.

-- start of 'swiftmailer.html.twig' --

<div>
 {{ body }}
</div>

-- end of 'swiftmailer.html.twig' --

2.2.1 Theme File Variables

This section is an overview of the basic variables that are available within
the template 'swiftmailer.html.twig'. Additional variables might be added by
Drupal or other modules.

$key
  The key which identifies the e-mail. This is the $key provided to
  drupal_mail() which identifies the e-mail.
$to
  The recipient's e-mail address.
$from
  The sender's e-mail address.
$language
  The language used to compose the e-mail.
$params
  An array of parameters. This is the $params optionally provided to
  \Drupal\Core\Mail\MailManagerInterface::mail
$subject
  The subject.
$body
  The actual content.

Tip! You can make even more variables available in 'swiftmailer.html.twig' if
     you implement your own preprocess function. If you would like to make more
     variables available from a module you control, then you simply just need to
     implement the following preprocess function:

     [yourmodule]_preprocess_swiftmailer(&$variables).

     Similarly, if you want to make more variables available from your theme,
     then all you need is to implement the following preprocess function:

     [yourtheme]_preprocess_swiftmailer(&$variables). You can read more about
     which preprocess and process functions that are available from the
     'swiftmailer' hook in the Drupal 8 documentation for the function theme().

3.0 Attatchments, Inline Images and Advanced Usage

This section is targeted towards developers. It demonstrates how the message
format of any given e-mail can be set, along with how files and inline images
can be attached to e-mails.

3.1 Attachments

You can easily add attachments to e-mails. This can be done programatically by
defining one or more files to attach.

All files which are to be attached to an e-mail need to be represented as
instances of stdClass. This makes it easy for you to add files that are managed
by Drupal, as the file_load() function will return an stdClass instance which
represents a given file.

All stdClass instances returned by Drupal which represents files are populated
with the fields 'uri', 'filename' and 'filemime'. Thus, if you would like to
attach a file that are not managed by drupal, you then need to create an
instance of stdClass and populate that instance with the fields 'uri',
'filename' and 'filemime'. Drupal's drupal_realpath() will be used to determine
the actual location of the provided file as given in the 'uri' field. Thus,
files from both public and private file systems can be attached to e-mails.

Note! The Swift Mailer module has not been tested with stream wrappers other
      than the default public and private. It might not work other stream
      wrappers.

The below example demonstrates both how to attach a file managed by Drupal and
a file which is not managed by Drupal.

Please note! You can specify which files to add as attachments both from the
code block where you invoke drupal_mail() and from your module's implementation
of hook_mail(). In hook_mail(), simply make sure you add attachments to
$message['files'] and not to the provided $params argument.

/**
 * Send an e-mail.
 */
function test() {

  //File one (managed by Drupal).
  $file_one = file_load(1);

  //File two (not managed by Drupal).
  $file_two = new stdClass();
  $file_two->uri = 'sites/default/files/images/logo.jpg';
  $file_two->filename = 'drupal_logo.jpg';
  $file_two->filemime = 'image/jpeg';

  // Add attachments.
  $p['files'][] = $file_one;
  $p['files'][] = $file_two;

  // Send e-mail.
  $mail_manager = \Drupal::service('plugin.manager.mail');
  $mail_manager->mail('modulename', 'key', 'test@test.com', \Drupal::service("language.default")->get()->getId(), $p);
}

/**
 * Implementation of hook_mail().
 */
function modulename_mail($key, &$message, $params) {

  switch($key) {
    default:
      $text[] = t('<strong>Hi</strong>');
      $text[] = t('<p>This is an automatically generated test e-mail.</p>');

      //File three (managed by Drupal).
      $file_three = file_load(2);
      $message['files'][] = $file_three;

      $message['subject'] = t('Test');
      $message['body'] = $text;
      break;
    }

}

It should be stressed that the module only supports attaching already existing
files. In other words, dynamically generated files which are to be added as
attachment to an e-mail needs to be generated and stored in a permanent or
temporary location before it is provided as an attachment. A recommended way
to handle temporary files is to utilise Drupals file system and mark the files
as temporary. Drupal will then take care of deleting those files after a set
amount of time.

3.1.1 The Attachment Hook

The SwiftMailer module allows other modules to add attachments to e-mails.
This is done through hook_swiftmailer_attach() which other modules need to
implement in order to add attachments to e-mails. The below block of code
demonstrates how a module can add attachments to an e-mail using
hook_swiftmailer_attach().

function swiftmailer_swiftmailer_attach($key) {

    // Load file which is managed by Drupal.
    return file_load(2);

}

3.2 Inline Images

Adding inline images to e-mails is just as easy as adding files as attachments.
You can choose between two ways of adding images. The first one assumes that
your module does all the work, and the second assumes that your theme takes
care of specifying which images to display as inline images.

3.2.1 Let your module do the work!

Images which are to be attached to an e-mail need to be represented as instances
of stdClass. This makes it easy for you to add image files that are managed by
Drupal, as the file_load() function will return an stdClass instance which
represents a given image file. However, in contrast to attachments, you will
need to manually apply the field 'cid' to an image file which is to be used as
an inline image. The 'cid' field needs to hold the id of the image file, and
will be used to establish a link between the attached image and its display
location in the e-mail body.

All stdClass instances returned by Drupal which represents (image) files are
populated with the fields 'uri', 'filename' and 'filemime'. Thus, if you would
like to attach an image file that are not managed by Drupal, you then need to
create an instance of stdClass and populate that instance with the fields 'uri',
'filename' and 'filemime'. Drupal's drupal_realpath() will be used to determine
the actual location of the provided image file as given in the 'uri' field.
Thus, image files from both public and private file systems can be attached to
e-mails.

The below example demonstrates both how to attach an image file that is managed
by Drupal.

/**
 * Implementation of hook_mail().
 */
function modulename_mail($key, &$message, $params) {

  switch($key) {
    default:

      $logo_id = '390ffcm-sdfd94f';

      $text[] = '<img src="cid:' . $logo_id . '" />;
      $text[] = t('<p>This is an automatically generated test e-mail with an
                   inline image..</p>');

      //Inline image (managed by Drupal).
      $inline_image = file_load(1);
      $inline_image->cid = $logo_id;
      $message['images'][] = inline_image;

      $message['subject'] = t('Test');
      $message['body'] = $text;
      break;
    }

}

3.2.2 Let your theme do the work!

Adding inline images from a theme might be even easier than adding inline
images from a module. The first thing you need to do is to add markup to your
the theme file which is responsible for showing the images. Now, the markup
needs to be a little different than the markup required when adding inline
images from a module. Instead of referencing images by CID, you simply
reference images by their path. The below line of code demonstrates this.

<img src="image:/sites/all/themes/mytheme/images/logo.jpg">
<img src="image:<?php print drupal_get_path('module', 'mymodule') .
'/images/drupal.jpg'; ?>" />

The essential part in the above lines of code is to make sure that the path of
the image is prefixed with 'image:'. This tells the Swift Mailer module to
embed that image file. The Swift Mailer module will automatically take care of
assigning the image a CID to establish a link between the image and the markup.

3.3 Message Format

You can specify wether a message should be sent as plain text ('text/plain') or
HTML ('text/html'). Furthermore, the character set can also be set. The below
example demonstrates how this can be achieved.

/**
 * Send an e-mail
 *
 */
function test() {

  // Define message format.
  $p = [
    'format' => 'text/html',
    'charset' => 'UTF-8',
  ];

  // Send message.
  \Drupal::service('plugin.manager.mail')->mail('mymodule', 'key', 'test@test.com', \Drupal::languageManager()->getDefaultLanguage()->getId(), $p);

}

The above example will send an e-mail message as plain text.

4.0 HTML vs. plain Text

You can configure Swift Mailer to send messages as HTML and/or plain text.
E-mails which are configured to be sent as HTML messages will be themed, whereas
e-mails configured to be sent as plain text will not be themed.

You can configure Swift Mailer to automatically generate a plain text version
of HTML e-mails. Plain text versions not be themed and will be displayed by
e-mail clients not capable of showing HTML. This automatic conversion from HTML
to plain text is carried out using html2text. Please refer to
http://www.chuggnutt.com/html2text for more details.

To configure Swift Mailer to carry out automatic conversion for HTML e-mails,
go to admin/config/people/swiftmailer/messages and select the option 'Generate
alternative plain text version' (Note! This requires the default message format
to be set to HTML, or the variable $message['params']['convert'] to be set to
'TRUE'.

If you would like to provide both HTML and plain text versions yourself, then
simply provide the HTML version in $message['body'] and the plain text version
in $message['plain']. Please make sure you set $message['params']['format'] to
'text/html'. The Swift Mailer module will not attempt to generate a plain text
version if one is already available.

5.0 Custom settings/behavior

If you want to add custom settings or behavior to the mailer or the message,
before the it is sent, you can implement hook_swiftmailer_alter().
See swiftmailer.api.php for an example.
