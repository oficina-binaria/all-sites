# All Sites

Página "em construção" única para vários domínios estacionados.

Em vez de manter uma página por domínio, todos eles apontam para esta aplicação. Ela lê o host
da requisição e monta na hora uma landing page personalizada, com o nome da empresa gerado a
partir do domínio.

## Para que serve

- Dar uma página apresentável a domínios registrados que ainda não têm site.
- Evitar páginas de erro, telas padrão do provedor ou anúncios de estacionamento nesses domínios.
- Permitir apontar um domínio novo sem nenhuma alteração de código: basta configurar o DNS para a
  aplicação.

## Como funciona

A rota `/` (`App\Http\Controllers\LandingPageController`) remove o `www.` do host e transforma o
domínio em um nome:

| Domínio acessado            | Nome exibido    |
| --------------------------- | --------------- |
| `acme.com`                  | Acme            |
| `www.acme-solucoes.com.br`  | Acme Solucoes   |
| `loja.nova-era.net`         | Nova Era        |
| `blue_ocean.co.uk`          | Blue Ocean      |

Regras:

- Sufixos de dois níveis com código de país (`com.br`, `co.uk`, `net.br`…) são removidos inteiros.
  Os demais sufixos (`.com`, `.io`…) perdem só o último rótulo.
- Subdomínios são ignorados: o nome vem do rótulo imediatamente antes do sufixo.
- Hífens e sublinhados viram espaços e cada palavra começa com maiúscula.
- Domínios internacionalizados (`xn--…`) são convertidos para Unicode quando a extensão `intl`
  está disponível.
- Se a página for acessada por endereço IP, o nome usado é o `APP_NAME`.

A página (`resources/views/landing.blade.php`) está em português e tem suporte a tema escuro.

### Cache na edge

A aplicação foi pensada para rodar atrás de uma CDN (a edge da Laravel Cloud), para que bots e
visitantes dos domínios estacionados não precisem acordar o container a cada acesso:

- A rota `/` fica fora do grupo de middleware `web`: não há sessão, cookies nem CSRF, e a resposta
  não envia `Set-Cookie`.
- `/` responde com `Cache-Control: public, max-age=300, s-maxage=86400` e ETag.
- Qualquer outro caminho retorna 404 com `Cache-Control: public, max-age=300, s-maxage=3600`
  (configurado em `bootstrap/app.php`).
- `/up` é o endpoint de health check.

A aplicação não usa banco de dados.

## Stack

- PHP 8.5 e Laravel 13
- Tailwind CSS 4 e Vite
- Pest para testes
- Laravel Sail para o ambiente local (opcional)
- Laravel Cloud em produção

## Desenvolvimento

Instalação (dependências, `.env`, chave da aplicação e build dos assets):

```bash
composer setup
```

Servidor de desenvolvimento com Vite:

```bash
composer dev
```

Também dá para usar o Sail:

```bash
vendor/bin/sail up -d
```

Para ver a página de um domínio específico localmente, acesse com outro host, por exemplo
adicionando `127.0.0.1 acme-solucoes.com.br` ao `/etc/hosts`.

## Testes

```bash
php artisan test --compact
```

Os testes de `tests/Feature/LandingPageTest.php` cobrem a geração de nomes e os cabeçalhos de cache.

Antes de commitar alterações em PHP, formate o código:

```bash
vendor/bin/pint --dirty
```

## Deploy

A aplicação roda na Laravel Cloud. Para colocar um novo domínio no ar, adicione-o ao ambiente na
Laravel Cloud e aponte o DNS. Nenhuma alteração de código é necessária.
