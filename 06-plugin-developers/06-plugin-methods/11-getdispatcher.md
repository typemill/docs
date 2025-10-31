# getDispatcher

The method `$this->getDispatcher()` retrieves the event dispatcher. This is helpful when you want to dispatch a new event, or if you want to dispatch an existing event to make resources available that depend on this event. 

* **@return:** (\Symfony\Component\EventDispatcher\EventDispatcher) Returns the event dispatcher.

## Example Usage

```php
private function sendConfirmationEmail()
{
   # we have to dispatch onTwigLoaded to get the mail-function from the email plugin into the container

   $dispatcher = $this->container->get('dispatcher');
   $dispatcher->dispatch(new OnTwigLoaded(false), 'onTwigLoaded');
   if($this->container->has('mail'))
   {
      # send mails here
   }
}
```

