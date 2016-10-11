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

Existem dois tipo de interceptors, o Request interceptor e o Response interceptor um vai inteceptar o request e o outro o response, vamos ver agora um exemplo de um interceptor que faz a authorização de cada requisição usando JWT, primeiro de tudo vamos ver o serviço q faz login:
<i>login.service.js</i>
````js
(function() {
    'use strict';

    angular
        .module('app')
        .factory('LoginService', LoginService);

    LoginService.$inject = ['$http', '$localStorage'];

    /* @ngInject */
    function LoginService($http, $localStorage) {

        var service = {
            login: login,
            logout: logout
        };

        return service;

        function login(user, callback) {
          $http.post('/token', {email: user.email, password: user.password})
            .success(function(response) {

              if(response.token) {
                var token = 'JWT ' + response.token;
                $localStorage.currentUser = {email: user.email, token: response.token};
                $http.defaults.headers.common.Authorization =  response.token;
                localStorage.setItem('token',token);

                callback(true);
              } else {
                callback(false);
              }
            })
        }

        function logout() {
          delete $localStorage.currentUser;
          $http.defaults.headers.common.Authorization = '';
        }

    }
})();
````
podemos ver aqui que é feito uma req do tipo POST em '/token' essa req retorna um token válido se o usuário e senha estiverem corretos, então se a promisse entrar em success vamos fazer um test verificar se existe um token e a partir da ai setamos os headers com o email e token e depois usamos <b>localStorage.setItem('token',token);</b> passando o token que agora está disponível no localstorage e vamos agora usar esse artificio para 'pegar' o token lá no interceptor vamos lá...

<i>authInterceptor.js</i>
````js
(function() {
    'use strict';

    angular
        .module('app')
        .factory('authInterceptor', authInterceptor);

    authInterceptor.$inject = ['$q','$location'];

    /* @ngInject */
    function authInterceptor($q, $location) {

        var service = {
            request: request,
            responseError: responseError
        };

        return service;

        function request(config) {
          config.headers = config.headers || {};
          if(localStorage.getItem('token')) {
            config.headers.Authorization = localStorage.getItem('token');
          }
          return config;
        }

        function responseError(rejection) {
          if(rejection.status === 401 || rejection.status === 403) {
            $location.path('/error/nao-auth');
          }
          return $q.reject(rejection);
        }
    }
})();
````
podemos ver aqui como é simples, HAHA muita cara de pau minha dizer isso que passei 3 dias pra conseguir implementar maaas , kkk vamos ao que interessa... , o interceptor vai ter um request recebendo config e vamos novamente em localStorage e se tivermos um token setamos esse token no header do config e retornamos o config como header <b>Authorization</b> e temos o responseError ou seja se o status da resposta for 401 ou 403 enviamos para uma url de erro e retornamos $q.reject e retornamos a rejection, e agora criamos um config para adicionar o interceptor no array de interceptor.

<i>interceptorConfig.js</i>
````js
angular
    .module('app')
    .config(config);

function config($httpProvider) {
  // $httpProvider.interceptors.push('timestampInterceptor');
  $httpProvider.interceptors.push('authInterceptor');

}
````
e adicionamos o inteceptor <b>authInterceptor</b> no array de interceptors do $httpProvider e agora a cada req já temos o interceptor fazendo seu trabalho.
