# JWT Authentication com AngularJS

## Visão geral
<ul>
  <li>Recapitular: Session Identifiers</li>
  <li>Cookies, da maneira correta</li>
  <li>Introdução a JWT</li>
  <li>Access Tokens e Refresh Tokens</li>
  <li>Armazenando JWTs no browser</li>
  <li>Angular especificidades</li>
</ul>

### Recapitular: Session Identifiers

Identificadores de sessão, esse é uma maneira prática e simples para gerenciar uma sessão para um usuário que consiste básicamente em três passos:

<li>Verificar usuário e senha </li>
<li>Criar uma session ID e associar ao usuário</li>
<li>Guardar a session ID em um cookie</li>

a imagem abaixo detalha como isso acontece:
<img src="img/img1-session-cookie.png"/>

podemos ver que o usuário faz login e quando isso acontece é enviando o login e senha via POST estando tudo certo o servidor retorna um http statusCode 200 e seta um ID para a sessão
armazena esse session ID em um cookie e quando o usuário fizer uma requisição para profile por exemplo é feito um get nesse cookie e nesse momento ocorre uma verificação se o session ID for igual ao que foi enviando pelo servidor então e retornado um 200 e para cada requisição tudo ocorre asim.

<b>Preocupação com session ID</b>

<ul>
  <li>eles não tem nenhum significado são códigos gerados aleatoreamente</li>
  <li>aumenta a carga ao banco de dados, é feita uma pesquisa pelo session ID em cada requisição</li>
  <li>precisam ser protegidos para evitar ataques a sessão</li>
</ul>

### Cookies, da maneira correta

Cookies podem ser facilmente comprometidos:

<ul>
  <li>Man-in-the-middle (MITM)</li>
  <li>Cross-Site Scripting (XSS)</li>
  <li>Cross-Site Request Forgery (CSRF)</li>
</ul>

<b>Man-in-the-middle (MITM)</b>

Alguém pode ouvir a conversa entre o browser e o servidor e interceptar e roubar o cookie durante essa 'conversa'

<i>Solução:</i>
<li>Usar HTTPS/TLS em todos os locais onde um cookie irá transitar</li>
<li>Set secure flag no cookie</li>

<b>Cross-Site Scripting (XSS)</b>

Esse é um problema real, e ocorre quando alguém roda um script mal intencionado dentro do browser direcionado para o dominio a ser atacado e pode ser usado para roubar cookies.

aqui temos um exemplo isso [EXEMPLO](https://www.google.com/about/appsecurity/learning/xss/#StoredXSS).
podemos ver no exemplo que é executado um código malicioso no browser e com isso o atacante tem acesso a todos os nossos cookies.

<i>Solução:</i>
<li>Server Side: usar bibliotecas conhecidas e confiáveis para garantir que o HTML dinâmico não contém código executável. que não foi gerado por nós é claro.</li>
<li>Client Side: verificar inputs em forms enviandos pelo usuário existem frameworks que podem fazer isso por nós, mas é sempre bom ver a documentação.</li>
<li>Setar o atributo HttpOnly para authenticação dos cookies. <b>Com HttpOnly os cookies não são acessíveis pelo javascript environment.</b></li>

links de exemplo:
https://www.owasp.org/index.php/XSS </br>
https://www.google.com/about/appsecurity/learning/xss/

<b>Cross-Site Request Forgery (CSRF)</b>

Explora o fato de tags HTML não seguir a mesma política de origem ao fazer solicitações GET.

exemplo: um atacante coloca uma imagem com conteúdo malicioso dentro de uma pagina WEB que seus usuários visitam.
````
<img src="https://trustyapp.com/transferMoney?to=BadGuy&amount=10000"/>
````
o que irá acontecer aqui é:
<li>o browser irá enviar cookies para trustapp.com.</li>
<li>o servidor enviar um cookie confiável e assume que foi uma ação do usuário.</li>
<li>transfere o dinheiro.</li>

<i>Solução:</i>
<li>Sincronizar token (para aplicações form-based)</li>
<li>Double-submit cookie ( para aplicações modernas)</li>

Double-submit cookie </br>

<li>cliente tem dois cookies, o um é session ID e o segundo é um valor random.</li>
<li>Client envia de volta um valor random no header, desencadeando o same-origin-policy.</li>

a imagem abaixo detalha como isso acontece:
<img src="img/img2-session-cookie-random.png"/>

e o servidor agora sabe tratar e rejeitar a ação maliciosa.

a imagem abaixo detalha como isso acontece:
<img src="img/img3-session-cookie-reject.png"/>

para saber mais:
https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF) </br>
https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy </br>

### Introdução a JWT

<b>Definição</b>

<li><b>Authenticação:</b> é quando provamos quem somos.</li>

<li><b>Authorização:</b> é quando garantimos que temos acesso algum resource.</li>

<li><b>Tokens:</b> são usados para persistir a authenticação e para pegar ( get ) a authorização.</li>

<li><b>JWT:</b> é um formato de token.</li>

<b> JSON web tokens (JWT) </b>

um JWT parece com uma string sem sentido algo do tipo
````
eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1NiJ9.eyJ
pc3MiOiJqb2UiLA0KICJleHAiOjEzMDA4MTkzODAsDQo
gImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnV
lfQ.dBjftJeZ4CVPmB92K27uhbUJU1p1r_wW1gFWFOEj
Xk
````
mas ele não é 'sem sentido', o JWT consiste em uma estrutura
com três partes encodadas em Base64-URL:

<img src="img/img4-JWT-div.png"/>

e esse mesmo JWT decodificado fica dessa forma:

<img src="img/img5-JWT-decoded.png"/>

o body do JWT é a parte mais importante e está dividida da seguinte forma:

````
{
  "iss":"http://trustapp.com/", <---- quem emitiu o token.
  "exp":"1300819380", <---- quando vai expirar.
  "sub":"users/258594", <---- quem representa ( usuário ).
  "scope":"self api/buy", <--- o que pode fazer.
}
````

<b>Enviando e verificando JWTs </b>
