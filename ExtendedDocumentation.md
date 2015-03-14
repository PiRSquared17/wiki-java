This should be read in conjunction with the Javadoc comments in the source file.

### Example program ###
A typical program would go something like this:

```
Wiki wiki;
File f = new File("wiki.dat");
if (f.exists()) // we already have a copy on disk
{
   ObjectInputStream in = new ObjectInputStream(new FileInputStream(f));
   wiki = (Wiki)in.readObject();
}
else
{
   try
   {
       wiki = new Wiki("en.wikipedia.org"); // create a new wiki connection to en.wikipedia.org
       wiki.setThrottle(5000); // set the edit throttle to 0.2 Hz
       wiki.login("ExampleBot", password); // log in as user ExampleBot, with the specified password
   }
   catch (FailedLoginException ex)
   {
       // deal with failed login attempt
   }
}
try
{
   for (String page : pages) // pages generated from (say) getCategoryMembers()
   {
       try
       {
           // do something with page
       }
       // this exception isn't fatal - probably won't affect the task as a whole
       catch (CredentialException ex)
       {
           // deal with protected page
       }
   }
}
// these exceptions are fatal - we need to abandon the task
catch (CredentialNotFoundException ex)
{
   // deal with trying to do something we can't
}
catch (CredentialExpiredException ex)
{
   // deal with the expiry of the cookies
}
catch (AccountLockedException ex)
{
   // deal with being blocked
}
catch (IOException ex)
{
   // deal with network error
}
```

Don't forget to release system resources held by this object when done.
This may be achieved by logging out of the wiki. Since `logout()` is entirely offline, we can have a persistent session by simply serializing this wiki, then logging out as follows:

```
File f = new File("wiki.dat");
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(f));
out.writeObject(wiki); // if we want the session to persist
out.close();
wiki.logout();
```

Long term storage of data (particularly greater than 20 days) is not
recommended as the cookies may expire on the server.

### Assertions ###

Without too much effort, it is possible to emulate assertions supported
by [mw:Extension:Assert Edit](http://mediawiki.org/wiki/Extension:Assert_Edit). The extension need not be installed for these assertions to work. Use `setAssertionMode(int mode)` to set the assertion mode. Checking for login, bot flag or new messages is supported by default. Other assertions can easily be defined, see [Programming With Assertions](http://docs.oracle.com/javase/1.4.2/docs/guide/lang/assert.html). Assertions are applied on write methods only and are disabled by default.

IMPORTANT: You need to run the program with the flag -enableassertions
or -ea to enable assertions, example: <tt>java -ea Mybot</tt>.