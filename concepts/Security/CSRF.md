# CSRF

Cross-site request forgery ([CSRF](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) is a type of attack which forces an end user to execute unwanted actions on a web application backend with which he/she is currently authenticated.  In other words, without protection, cookies stored in a browser like Google Chrome can be used to send requests to Chase.com from a user's computer whether that user is currently visiting Chase.com or Horrible-Hacker-Site.com.

### Enabling CSRF Protection

Sails bundles optional CSRF protection out of the box. To enable the built-in enforcement, just make the following adjustment to [sails.config.csrf](/#/documentation/reference/Configuration/CSRF.html) (conventionally located in your project's [`config/csrf.js`](http://beta.sailsjs.org/#/documentation/anatomy/myApp/config/csrf.js.html) file):

```js
csrf: true
```

Note that if you have existing code that communicates with your Sails backend via POST, PUT, or DELETE requests, you'll need to acquire a CSRF token and include it as a parameter or header in those requests.  More on that in a sec.



### CSRF Tokens

Like most Node applications, Sails and Express are compatibile with Connect's [CSRF protection middleware](http://www.senchalabs.org/connect/csrf.html) for guarding against such attacks.  This middleware implements the [Synchronizer Token Pattern](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet#General_Recommendation:_Synchronizer_Token_Pattern).  When CSRF protection is enabled, all non-GET requests to the Sails server must be accompanied by a special token, identified by either a header or a parameter in the query string or HTTP body.

CSRF tokens are temporary and session-specific; e.g. Imagine Mary and Muhammad are both shoppers accessing our e-commerce site running on Sails, and CSRF protection is enabled.  Let's say that on Monday, Mary and Muhammad both make purchases.  In order to do so, our site needed to dispense at least two different CSRF tokens- one for Mary and one for Muhammad.  From then on, if our web backend received a request with a missing or incorrect token, that request will be rejected. So now we can rest assured that when Mary navigates away to play online poker, the 3rd party website cannot trick the browser into sending malicious requests to our site using her cookies.

### Dispensing CSRF Tokens

To get a CSRF token, you should either bootstrap it in your view using [locals](http://beta.sailsjs.org/#/documentation/concepts/Views/Locals.html) (good for traditional multi-page web applications) or fetch it using sockets or AJAX from a special protected JSON endpoint (handy for single-page-applications (SPAs).)


##### Using View Locals:

For old-school form submissions, it's as easy as passing the data from a view into a form action.  You can grab hold of the token in your view, where it may be accessed as a view local: `<%= _csrf %>`

e.g.:
```html
<form action="/signup" method="POST">
 <input type="text" name="emailaddress"/>
 <input type='hidden' name='_csrf' value='<%= _csrf %>'>
 <input type='submit'>
</form>
```
If you are doing a `multipart/form-data` upload with the form, be sure to place the `_csrf` field before the `file` input, otherwise you run the risk of a timeout and a 403 firing before the file finishes uploading.





##### Using AJAX/WebSockets

In AJAX/Socket-heavy apps, you might prefer to send a GET request to the built-in `/csrfToken` route, where it will be returned as JSON, e.g.:

```json
{
  "_csrf": "ajg4JD(JGdajhLJALHDa"
}
```




### Spending CSRF Tokens

Once you've enabled CSRF protection, any POST, PUT, or DELETE requests (**including** virtual requests, e.g. from Socket.io) made to your Sails app will need to send an accompanying CSRF token as a header or parameter.  Otherwise, they'll be rejected with a 403 (Forbidden) response.

For example, if you're sending an AJAX request from a webpage with jQuery:
```js
$.post('/checkout', {
  order: '8abfe13491afe',
  electronicReceiptOK: true,
  _csrf: 'USER_CSRF_TOKEN'