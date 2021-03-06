# Gerando o Projeto

```console
rails new OneBitContacts --api
```

* Se quiser utilizar o PostgreSQL adicione ao comando:

```console
--database=postgresql
```

## Versionando a API

1 – Dentro da pasta controllers crie uma pasta chamada api;
2 – Dentro da pasta criada crie uma nova pasta chamada v1;
3 – Dentro da pasta criada crie um arquivo chamado api_controller.rb e coloque nele:

```rb
module Api::V1
 
 class ApiController < ApplicationController
 
   # Métodos globais
 
 end
 
end
```

## Habilitando o CORS

Um pouco sobre o que é o [CORS](https://www.treinaweb.com.br/blog/o-que-e-cors-e-como-resolver-os-principais-erros)

Acrescente no seu Gemfile:

```rb
  gem 'rack-cors'
```

Instale a gem rodando:

```console
  bundle install
```
Acrescente no seu config/application.rb:

```rb
config.middleware.insert_before 0, Rack::Cors do
 allow do
   origins '*'
   resource '*',
     headers: :any,
     methods: %i(get post put patch delete options head)
 end
end
```

## Configurando o Rack Attack

Sobre o [rack-attack](https://github.com/rack/rack-attack)

Acrescente no seu Gemfile:

```rb
gem 'rack-attack'
```

Instale a Gem:
```console
bundle
```

Acrescente em config/application:

```rb
config.middleware.use Rack::Attack
```

Crie um arquivo chamado rack_attack.rb no seu config/initializers/ e coloque nele:

```rb
class Rack::Attack
 
 Rack::Attack.cache.store = ActiveSupport::Cache::MemoryStore.new
 
 # Allow all local traffic
 
 safelist('allow-localhost') do |req|
 
   '127.0.0.1' == req.ip || '::1' == req.ip
 
 end
 
 # Allow an IP address to make 10 requests every 10 seconds
 
 throttle('req/ip', limit: 5, period: 5) do |req|
 
   req.ip
 
 end
 
 # Throttle login attempts by email address
 
 #throttle("logins/email", limit: 5, period: 20.seconds) do |req|
 
 #  if req.path == '/users/sign_in' && req.post?
 
 #    req.params['email'].presence
 
 #  end
 
 #end
 
end
```

## Instalando o Devise

Acrescente no seu Gemfile:

```rb
gem 'devise'
```
Instale rodando:

```console
bundle install
```

Em config/environments/development.rb coloque:

```rb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

Rode no console:

```console
rails generate devise:install
```

Rode no console:

```console
rails generate devise User
 
rails g migration addNameToUser name:string
```

Gere as migrations:

```console
rails db:migrate
```

## Devise Simple Token

Adicione ao seu Gemfile:

```rb
gem 'simple_token_authentication', '~> 1.0'
```

Instale rodando:

```console
bundle install
```

Adicione ao seu User Model:

```rb
acts_as_token_authenticatable
```

Rode no console:

```console
rails g migration add_authentication_token_to_users "authentication_token:string{30}:uniq"
 
rake db:migrate
```

Coloque no seu controler api/v1/api_controller.rb:

```rb
acts_as_token_authentication_handler_for User

before_action :require_authentication!

private

def require_authentication!

  throw(:warden, scope: :user) unless current_user.presence

end
```
## Gerando nossos Models

Rode no console

```console
rails g model Contact name:string email:string phone:string description:text user:references # Esse `user:references`vai ligar o model contact com user colocando o nosso famoso belongs_to :user

rails db:migrate
```

Adicione ao seu Model User:

```rb
has_many :contacts, dependent: :destroy # Esse `dependet: detroy` significa que quando o usuário for deletado isso deletará todos os contacts também
```

## Gerando nossos Controllers

Rode no console

```console
rails g controller api/v1/contacts
```

Coloque isso no controller gerado:

```rb
class Api::V1::ContactsController < ApplicationController
  before_action :set_contact, only: [:show, :update, :destroy]
  before_action :require_authorization!, only: [:show, :update, :destroy]

  # GET /api/v1/contacts
  def index
    @contacts = current_user.contacts
    render json: @contacts
  end

  # GET /api/v1/contacts/1
  def show
    render json: @contact
  end

  # POST /api/v1/contacts
  def create
    @contact = Contact.new(contact_params.merge(user: current_user))

    if @contact.save
      render json: @contact, status: :created
    else
      render json: @contact.errors, status: :unprocessable_entity
    end
  end

  # PATCH/PUT /api/v1/contacts/1
  def update
    if @contact.update(contact_params)
      render json: @contact
    else
      render json: @contact.errors, status: :unprocessable_entity
    end
  end

  # DELETE /api/v1/contacts/1
  def destroy
    @contact.destroy  
  end

  private

  # Use callbacks to share common setup or constraints between actions.
  def set_contact
    @contact = Contact.find(params[:id])
  end

  # Only allow a trusted parameter "white list" through.
  def contact_params
    params.require(:contact).permit(:name, :email, :phone, :description)
  end

  def require_authorization!
    unless current_user == @contact.user
      render json: {}, status: :forbidden 
    end
  end
end

```

## Ajustando as Rotas

Atualize o seu arquivo routes.rb colocando:

```rb
Rails.application.routes.draw do
  devise_for :users
  namespace :api do
    namespace :v1 do
      resources :contacts
    end
  end
 
  # constraints subdomain: 'api' do
  #   scope module: 'api' do
  #     namespace :v1 do
  #       resources :contacts
  #     end
  #   end
  # end
end

```

## Subindo o Server

 Rodar o comando dentro da pasta onde está o seu projeto
```console
rails s

ou 

bin/rails server
```