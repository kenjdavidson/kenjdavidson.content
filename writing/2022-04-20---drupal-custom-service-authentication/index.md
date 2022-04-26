---
title: Drupal Custom Service w/ Authentication
description: Connecting Drupal to a Google Authenticated Back End
date: 2022-04-20
tags: [Technology, Drupal, PHP, GuzzleHttp]
slug: "/2022/04/20/drupal-custom-service-authentication"
---

Since I've had the __opportunity__ (for lack of a better word) to continue down the Drupal rabbit hole - the next requirements of our project required that we have Drupal communciating with our back end services (Java).  Our application is based around Google OAuth and therefore it made sense to implement:

- A custom Http client on Drupal
- Using Google Service Accounts as authentication
- Performing the appropriate validation on the back end

## Custom Drupal Client

As much as __I absolutely love__ Drupal, the Drupal 8 Service configuration and Injection Container was actually a delight to work with.   I found a number of great articles on creating an Http Client using the built in `GuzzleHttp\Client` library.

- [https://drupalize.me/blog/201512/speak-http-drupal-httpclient](https://drupalize.me/blog/201512/speak-http-drupal-httpclient) being the primary one

There are a number of different approaches that can be taken here:

1. Implementing a Factory to provide a standard `GuzzleHttp\Client`
2. Implementing a custom client

To give an idea of the differences, it's all dependent on how much (or little) custom functionality we need to provide.  The way I look at it is that I'll provide a number of levels:

- The `Factory` which will create the `GuzzleHttp\Client`
- The `GuzzleHttp\Client` __service__
- A standard data wrapping service `DataService` wrapping Drupal logging functionality
- Any number of added services `MemberService`, etc. that will be used to provide `#getContactInfo()` specific functions

### Module Service YML File

At this point, we'll just get the Service and Factory configured and available:

```yml:title=module.services.yml
  module.service_client:
    class: GuzzleHttp\Client 
    factory: module.service_client_factory:get 
  module.service_client_factory:
    class: Drupal\module\Services\ServiceClientFactory
```

### Implementing the Factory

Next we'll need to implement the factory, the primary methods are:

- `#get()` which returns the simple `new Client()`
- `#handler_stack()` which we use to build the appropriate middleware

```php:title=ServiceClientFactory
class ServiceClientFactory {
    function get() {
        $config = [
            'base_uri' => Settings::get('client_uri'),
            'handler' => $this->handler_stack()
        ];
        
        return new Client($config);
    }
    
    function handler_stack() {
        $stack = HandlerStack::create();
        $stack->push(new Authentication(Settings::get('client_email'), Settings::get('client_pkey'), Settings::get('client_key')));
        return $stack;
    }    
}
```

At this point we are able to inject the basic client into any `Service`(s) or `Controllers`(s) using the standard injection code:

```php:title=Member Profile Controller
class MemberProfileController extends ControllerBase {

    /**
     * @var \GuzzleHttp\Client
     */
    protected $client;

    public function __construct(Client $client) {
        $this->Client = $client;
    }

    public static function create(ContainerInterface $container) {
        return new static(
            $container->get('module.service_client')
        );
    }

    public function memberProfileTitle() {
        return "Member Profile {$this->currentUser()->getAccountName()}";
    }

    public function memberProfile($userId) {
        $memberProfile = $this->client->get("members/#{$this->currentUser()->memberNumber()}/profile");
        return [
            '#markup' => "This is the member profile {dump($memberProfile)}"
        ];
    }
}
```

> At this point with the appropriate configuration for the `MemberProfileController` routing we should start seeing some errors!!  We still need authentication setup.

## Google Authentication

We're using Google Service Accounts for this, but essentially any authentication mechanism will work.  To get up and running with Google Service Accounts we can jump over to [https://cloud.google.com/iam/docs/understanding-service-accounts](https://cloud.google.com/iam/docs/understanding-service-accounts) and get started.  

> Magic happens and we download our `service-account.json` file

The json file you get back will look like this:

```json:title=Service Account Json
{
  "type": "service_account",
  "project_id": "project_id",
  "private_key_id": "private_key_id",
  "private_key": "-----BEGIN PRIVATE KEY-----\n-----END PRIVATE KEY-----\n",
  "client_email": "client@service.google.com",
  "client_id": "client_id",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://custom_public_key_url"
}
```

In order to process the JWT I started using the [firebase/php-jwt](https://github.com/firebase/php-jwt) library.

### Drupal Settings

To make the configuration available, we need to pop some of these items into the `settings.local.php` file (or .env file if preferred).

```php:title=settings.local.php
$settings['client_uri'] = 'http://localhost:8080/api/';
$settings['client_email'] = 'client@service.google.com';
$settings['client_pkey'] = '-----BEGIN PRIVATE KEY-----=
-----END PRIVATE KEY-----';
$settings['client_key'] = 'private_key_id';
```

> One important thing to note, firebase/php-jwt does not like the private key with `\n` in it.  You'll need to replace the `\n` with actual new lines in the `settings.local.php` file.

Now that we have the configuration available, we can implement the

### Authentication Middleware

The authentication middleware is solely responsible for applying the appropriate `Authentication: Bearer <JWT>` header to each of our requests.  We can see that there are two functions `#header()` and `#body()` responsible for generating the appropriate arrays, which is then generated using `JWT::encode()` and assigned to the `Authentication` header:

```php:title=Authentication Middleware
class Authentication {
    // Constructor and field definition
    
    public function __invoke(callable $handler) {
        return function(RequestInterface $request, array $options = []) use ($handler) {        
            $body = $this->body();
            $jwt = JWT::encode($body, $this->privateKey, "RS256", $this->keyId, $this->headers());
            $request = $request->withHeader("Authentication", "Bearer: {$jwt});

            return $handler($request, $options);
        };
    }
    
    private function headers() {
        return [
            "alg" => "RS256",
            "typ" => "JWT",
            "kid" => $this->keyId
        ];
    }
    
    private function body() {        
        $issuedAt = new DateTimeImmutable();
        return [
            "iss" => $this->email,
            "sub" => $this->email,
            "aud" => "custom audience here",
            "iat" => $issuedAt->getTimestamp(),
            "exp" => $issuedAt->modify('+1 hour')->getTimestamp()
        ];
    }
}
```

At this point you should be able to open up your configured url `/members/12352/profile` and see the `MemberProfileController` firing requests off to the service.  There are still a number of features left to implement:

- The backend must implement the appropriate `GoogleAuthenticationTokenVerifier` using the supplied information from the `json` file
- Logging and error handling should be implemented in the Client, specific to Drupal services

But at this point we should be in the position to start making authenticated requests.
