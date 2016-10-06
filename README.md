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
<img src="img do slide 8"/>

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
https://www.owasp.org/index.php/XSS
https://www.google.com/about/appsecurity/learning/xss/

<b>Cross-Site Request Forgery (CSRF)</b>
