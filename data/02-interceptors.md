# Interceptors angularJS.

Um interceptor é um tipo de serviço que permite a interceptação das requisições e respostas do serviço <b>$http</b>.

<i>exemplo</i>:
````js
app.factory('nomeDoInterceptor',function($q) {
  return {
    request:function(config) {
      return config;
    },
    requestError:function(rejection) {
      return $q.reject(rejection);
    },
    response:function(response) {
      return response;
    },
    responseError:function(rejection) {
      return $q.reject(rejection);
    }
  };
  });
````

criamos uma factory e retornamos um objeto, mas não basta apenas isso para criar um interceptor, o serviço $http possui um array de interceptors que podem ser configurados na inicialização da aplicação:

<i>exemplo</i>:
````js
app.config(function($httpProvider) {
  $httpProvider.interceptors.push('nomeDoInterceptor1');
  $httpProvider.interceptors.push('nomeDoInterceptor2');
  $httpProvider.interceptors.push('nomeDoInterceptor3');
  $httpProvider.interceptors.push('nomeDoInterceptor4');
})
````

Existem dois tipo de interceptors, o Request interceptor e o Response interceptor um vai inteceptar o request e o outro o response, 
