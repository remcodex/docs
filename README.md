# RCE Documentation

## Basic Usage

* **SERVER**<p/>
  We will create [server](https://github.com/remcodex/server) first<br/>
  Create **index.php** file and add below code it
    ```php
    use Remcodex\Server\Command;
    use Remcodex\Server\ErrorHandler;
    use Remcodex\Server\Prebuilt\RequestListener;
    use Remcodex\Server\Prebuilt\ResponseListener;
    use Remcodex\Server\Router;
    use Remcodex\Server\Server;
    
    require 'vendor/autoload.php';
    
    //Load http routes
    $collector = Router::load(__DIR__ . '/routes.php');
    
    //Load request commands
    $commands = Command::load(__DIR__ . '/commands.php');
    
    //Create and start server
    Server::create()
        ->setEnvironment(Server::ENV_DEVELOPMENT)
        ->setErrorHandler(new ErrorHandler())
        ->setRouteCollector($collector)
        ->setCommands($commands)
        ->onRequest(new RequestListener())
        ->onResponse(new ResponseListener())
        ->run();
    ```
  Start 3 servers by executing each line below in different terminals
  ```bash
  php -S localhost:9110 index.php 
  php -S localhost:9111 index.php 
  php -S localhost:9112 index.php 
  ```


* **ROUTER**<p/>
  We will now create [router](https://github.com/remcodex/router) now, note that usage of router is entirely
  optional.<br/>
  Create **server.php** file and add below code to it.
  ```php
   use Remcodex\Router\RemoteServer;
   use Remcodex\Router\Server;
    
   require 'vendor/autoload.php';
    
   $serverUri = '0.0.0.0:9000';
   $server = Server::listen($serverUri);
    
   //Add remote server
   $server->addRemoteServer(
      //1
      RemoteServer::create('localhost:9110')
        ->protocol(RemoteServer::UNSECURE)
        ->path('api/http/request'),
      //2
      RemoteServer::create('localhost:9111')
        ->protocol(RemoteServer::UNSECURE)
        ->path('api/http/request'),
      //3
      RemoteServer::create('localhost:9112')
        ->protocol(RemoteServer::UNSECURE)
        ->path('api/http/request'),
  );
    
   //Add error handler
   $server->onError(function (Throwable $exception) {
     echo "Error occurred";
     echo $exception;
   });

   echo "Server starting at: http://{$serverUri}\n";
   $server->start();
  ```
  Start the router by executing below code in your terminal
  ```bash
  nodemon server.php
  ```


* **CLIENT**<p/>
  We will create a [client](https://github.com/remcodex/client) now.<p/>
  Create **test.php** file and add below code to it.
  ```php
   use Guzwrap\Wrapper\Form;
   use Guzwrap\UserAgent;
   use Remcodex\Client\Http\Request;
  
   require 'vendor/autoload.php';
  
   $response = Request::create()
     ->post(function (Form $form){
       $form->action('https://google.com');
       $form->method('post');
       $form->field('name', 'Ahmard');
       $form->field('time', date('H:i:s'));
     })
     ->userAgent(UserAgent::OPERA)
     ->withSharedCookie()
     ->debug()
     ->execute();
  
   var_dump($response->getSuccess()->getData());
  ```
  Now we execute our test file and see how it works
  ```bash
  php test.php
  ```