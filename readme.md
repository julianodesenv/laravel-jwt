## Exemplo de aplicação JWT no Laravel

Foi utilizado o pacote https://github.com/tymondesigns/jwt-auth para a implementação.

## Configurações

No arquivo config/auth.php
<pre>
'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],
</pre>

#### Passo a passo
- Adicionado usuário pela seeder
- No model é implamentado a interface JWTSubject (implementar 2 métodos: getJWTIdentifier e getJWTCustomClaims)
- Gerar chave através do comando: php artisan jwt:secret que ficará no .env

#### Criando endpoint para login de usuário
- Criar controller \Api\AuthController.php
- Importar a trait AuthenticatesUsers
- Criar routes no arquivo routes/api.php
<pre>
Route::name('api.login')->post('login', 'Api\AuthController@login');
</pre>
- Enviar um POST para api/login com e-mail e senha, na saída conterá o token

#### Permitindo acesso à API somente para usuários autenticados
- Criar um group de routes no arquivo routes/api.php
<pre>
Route::group(['middleware' => ['auth:api']], function(){});
</pre>
- Alteração no arquivo \App\Http\Middleware\Authenticate.php para sempre retornar json na route /api

#### Revogando token JWT - Logout
- No controller \App\Http\Controllers\Api\AuthController foi adicionado o método logout()
- Route adicionada no arquivo routes/api.php
<pre>
Route::post('logout', 'Api\AuthController@logout');
</pre>

#### Tempo de expiração do token
- Criar um LaravelServiceProvider para o JWT, o arquivo ficará em config/jwt.php
<pre>
php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"
</pre>
- Neste arquivo é possível determinar o tempo de vida do Token

#### Refresh Token
- No controller \App\Http\Controllers\Api\AuthController foi adicionado o método refresh()
- Route adicionada no arquivo routes/api.php
<pre>
Route::post('refresh', 'Api\AuthController@refresh');
</pre>

#### Auto refresh token
- Criar um nov middleware e registar o mesmo em app/Http/Kernel.php
<pre>
'jwt.refresh' => RefreshToken::class
</pre>
- Adicionar middleware a route
<pre>
Route::group(['middleware' => ['auth:api','jwt.refresh']], function(){});
</pre>
