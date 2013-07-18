# Pushing messages
Before you can start pushing make sure you have [generated a PEM certificate](certificate.md) you'll need this before you can start pushing. Ready? Lets push something!

## My first pushmessage
In this example we'll send one pushmessage to a device. It's the most basic example to give you an idea of how Notificato works:
```php
// First we get a Notificato instance and tell it what certificate to use as default certificate
$notificato = new Notificato('./certificate.pem', 'passphrase-to-use');

// Now we get a fresh message from Notificato
//  This message will be send to device with pushtoken 'fffff...'
//  it will automaticly be associated with the default certificate
$message = $notificato->createMessage('ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff');

// Let's set App icon badge with this push to 1
$message->setBadge(1);

// The message is ready, let's send it!
//  Be aware that this method is blocking and on failure Notificato will retry a few times
$messageEnvelope = $notificato->send($message);

// The returned envelope contains usefull information about how many retries where needed and if sending succeeded
echo $messageEnvelope->getFinalStatusDescription();
```

## Pushing multiple messages
When sending multiple messages you shouldn't use the `send`-method, but queue all messages and then send them at once. This will improve the performance and prevent unnecessary reconnects to Apple.
```php
// First we get the a Notificato instance and tell it what certificate to use as default certificate
$notificato = new Notificato('./certificate.pem', 'passphrase-to-use');

// Create an array to save the message envelopes in
$messageEnvelopes = array();

// Let's assume $pushinformation contains all push information we need
foreach ($pushinformation as $deviceToken => $badge)
{
	// Now we get a fresh message from Notificato and set the device token and badge
	$message = $notificato->createMessage($deviceToken);
	$message->setBadge($badge);

	// Queue the message for sending
	$messageEnvelopes[] = $notificato->queue($message);
}

// Now all messages are queued, lets send them at once
//  Be aware that this method is blocking and on failure Notificato will retry a few times
$notificato->flush();

// The returned envelopes contains usefull information about how many retries where needed and if sending succeeded
foreach ($messageEnvelopes as $messageEnvelope)
{
	echo $messageEnvelope->getIdentifier() . ' ' . $messageEnvelope->getFinalStatusDescription() . PHP_EOL;
}
```

## The full blown example
Here we try to show as many options in one example as possible to give you an idea of what is possible.
```php
// First we get the a Notificato instance and tell it what certificate to use as default certificate
//  We've disabled validation of the certificate because our PHP/OS doesn't parse it correctly and we set the environment ourselfs
$notificato = new Notificato('./certificate.pem', 'passphrase-to-use', false, Certificate::ENDPOINT_ENV_SANDBOX);
$notificato->setLogger( new Psr\Log\NullLogger() );

// Create an array to save the message envelopes in
$messageEnvelopes = array();

// Let's assume $pushinformation contains all push information we need
foreach ($pushinformation as $deviceToken => $badge)
{
	// Now we get a fresh message from Notificato and set the device token
	$message = $notificato->createMessage($deviceToken);
	$message->setExpiresAt(new \DateTime('+1 hour'));
	$message->setAlert('The numbers are 4, 8, 15, 16, 23 and 42', 'accept-button', 'launch-image');
	$message->setBadge($badge);
	$message->setSound('spookysound');
	$message->setContentAvailable(false);
	$message->setPayload( array('persons' => array('Locke', 'Reyes', 'Ford', 'Jarrah', 'Shephard', 'Kwon')) );

	// Queue the message for sending and set the retry limit to 10 times
	$messageEnvelopes[] = $notificato->queue($message, 10);
}

// Now all messages are queued, lets send them at once
//  Be aware that this method is blocking and on failure Notificato will retry a few times
$notificato->flush();

// The returned envelopes contains usefull information about how many retries where needed and if sending succeeded
foreach ($messageEnvelopes as $messageEnvelope)
{
	echo $messageEnvelope->getIdentifier() . ' ' . $messageEnvelope->getFinalStatusDescription() . PHP_EOL;
}
```

## What you should know
1. It is recommended to set an expiry date whenever you can, not setting this could make Apple discard the message instantly when the device is offline.
2. Sending a message without setting any content in it is completely valid, this is used with Passbook Passes.
3. All options and methods on the Message object can be found in the [API docs](http://wrep.github.com/notificato/master/Wrep/Notificato/Apns/Message.html).
4. You really should check the resulting state of the message envelopes and handle errors, if you don't the failed messages are just gone.
5. Logging is supported and can give you usefull information on what is happening. Set any [PSR-3](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-3-logger-interface.md) compatible logger with `$notificato->setLogger($logger)`.

## What's next?
Now you've send your messages you must [read the feedback service](feedback.md) once in a while.
