# Ruby & Rails Playbook

Este playbook tem como objetivo trazer opiniões e abordagens que estamos usando para desenvolver o Billimatic.

Tais opiniões e abordagens são definidas em conjunto com o time, representam o estado atual do que decidimos e estão sujeitas a serem revistas no futuro.

## Princípios

* Mire a simplicidade - ["I think most programmers spend the first 5 years of their career engaging complexity, and the rest of their lives learning simplicity."](https://twitter.com/buzz/status/7203012768).
* Recorde-se do mantra [Yagni](https://martinfowler.com/bliki/Yagni.html).
* Não reescreva código para se adequar a esse guia.
* Não viole esse playbook sem um boa razão.
* Uma razão é boa quando você convence um(a) colega do time.

## Referências

Algumas referências que usamos para nos ajudar a criar esse playbook:


* https://github.com/thoughtbot/guides
* https://www.betterspecs.org/

## Organização de código

Um dos principais desafios que temos em nossos projetos Rails é saber onde colocar o código e como nomear as classes e módulos. Abaixo uma referência com as principais dúvidas que temos para que possamos padronizar nossos projetos e minimizar as dúvidas.


### Boas práticas na criação de controllers

* um controller deve ter poucas linhas e a responsabilidade de apenas lidar com o comportamento para a view
* ele deve ser comunicar com Form Objects, Presenters e Services, toda comunicação para uma outra camada pode ser encarada como um "code smell". Uma exceção comum a essa regra são controllers que fazem o clássico CRUD do Rails, nestes casos não vemos problema de realizar a comunicação direta com o Model e [keep it simple](https://en.wikipedia.org/wiki/KISS_principle)
* se houver a necessidade de criar métodos privados dentro do controller, é mais um code smell, que o controller está começando a ter uma responsabilidade que não deveria ser dele. Uma solução comum, é criar um Service, que são detalhados abaixo

### Onde colocar classes e modulos que não são models, views ou controllers?

- `app/concerns`: Não é uma boa prática, apenas fragmenta o código. Tentar evitar ao máximo.
- `app/services`: Classes com regras de negócio e interações com serviços externos. Não deve ter `Service` no nome e nem ser um substantivo como `Order`. Cada service deve executar uma única função e ter um verbo no nome, ex: `CancelOrder`.

#### app/services/

Quando uma aplicação Rails cresce, e para evitar os anti-patterns [Fat Model e Fat Controller](https://blog.appsignal.com/2020/08/05/introduction-to-ruby-on-rails-patterns-and-anti-patterns.html), uma das abordagens mais difundida na comunidade é criação de uma camada de Services.

Uma definição fácil de entender é a seguinte:

> Service Object implements the user’s interactions with the application

**Padrão para classes de serviço**
- Herdar de `ApplicationService`.
- Não colocar `Service` no nome da classe, ex: `OrderService`, pois isso estimula o código da classe a crescer e fazer diversas coisas, violando o SRP. A sugestão é usar verbos no nome da classe, para que cada classe faça apenas uma coisa e o aplicativo cresça adicionando código e não editando código existente. Ex: `ShipOrder`, `CancelOrder`, `RefundOrder`.
- Ser chamadas sem instanciar.
- Ser chamadas sempre pelo método `call`.

**Referências sobre classes de serviço**
- [Service objects in Rails will help you design clean and maintainable code. Here's how](https://www.netguru.com/blog/service-objects-in-rails-will-help)
- [Domain Driven Rails by Yan Pritzker](https://vimeo.com/106759024). [slides da palestra](https://speakerdeck.com/skwp/domain-driven-rails).
- [Service Objects](http://railscasts.com/episodes/398-service-objects) para ver as vantages de remover o código de models e controllers e criar services.
- [Rails Service Objects: A Comprehensive Guide](https://www.toptal.com/ruby-on-rails/rails-service-objects-tutorial).

**Trade-offs**

Como mencionado [aqui](https://www.codewithjason.com/rails-service-objects/) e [aqui](https://avdi.codes/service-objects/), há alguns pontos contra o uso de Services, o principal que direciona para um código procedural. No nosso entender, o uso de uma camada de Services é um primeiro passo de evolução do código, que melhora a manutenção, os testes e é simples o suficiente para ser entendido e aplicado. Por isso em resumo, oferece um bom custo-benefício.

## Boas práticas

### RESTful controllers

Para que os controllers não cresçam com o tempo e sejam fáceis de manter, o ideal é que eles sejam [RESTful](https://pt.stackoverflow.com/a/232204/393), ou seja, devem seguir o padrão REST utilizando os verbos HTTP e resources corretamente.

Uma forma simples de saber se um controller é RESTful é ver se ele implementa algum método diferente de: 
- **index**
- **new**
- **create**
- **show**
- **edit**
- **update**
- **destroy**
  
**Somente estes métodos podem existir em um controller.**

Quando for necessário criar ações customizadas em um resource, deve-se criar controllers específicos para estas ações:

```ruby
resources :photos do
  resource :featured, only: [:create, :destroy]
end

resources :orders do
  resource :shipment, only: :create
  resource :refund, only: :create
end
```

Normalmente estas ações serão apenas `create` e `destroy` e utilizarão `POST` e `DELETE`. Para os resources acima teríamos:

```
POST photos/:photo_id/featured
DELETE photos/:photo_id/featured

POST orders/:order_id/shipment
POST orders/:order_id/refund
```

- [CRUD, Verbs, and Actions](https://guides.rubyonrails.org/routing.html#crud-verbs-and-actions)
- [RailsConf 2017: In Relentless Pursuit of REST by Derek Prior](https://www.youtube.com/watch?v=HctYHe-YjnE)

## Teste de Software
### Quais tipos de teste escrever

- Use request specs para testar controllers, [não use controller specs](http://rspec.info/blog/2016/07/rspec-3-5-has-been-released/).
- Não criar mais `feature test`, o custo-benefício deles não está valendo a pena. Temos a intenção de voltar com eles, em um "pipeline" a parte, e com cenários mais complexos e frequentes do uso da aplicação

### Como testar

- Os testes de request devem testar o sistema de forma integrada, portanto evitar mocks
- Os testes de services devem ser unitários

### Uso de mocks

- Quando a intenção do teste é ser unitário, se faz necessário mockar as dependências, para tanto garantir o teste a nível da unidade, como para melhorar a performance de tais testes. Sendo que ao usar factories, devem ser usado o método `build_stubbed`


### Factories

#### Preencha os objetos com o mínimo necessário

As factories devem sempre preencher o mínimo de informações necessárias para criar um objeto válido, nunca um objeto com todos os dados possíveis. 

Podem chegar vários bugs em produção, pois a factory preenche todos os campos no teste, mas em produção chegam campos `nil` que quebram as views por exemplo.

### Use let ou let!
Quando você tiver que atribuir uma variável, no lugar de criar uma variavel de instancia dentro de um before, use let. Assim, permitindo que a variável use lazy load apenas quando for usada pela primeira vez no teste e seja armazenada em cache até que o teste específico seja concluído. 

Exemplo:
```
let(:payment_information) { FactoryGirl.create(:payment_information) }
let(:receivables) do
  [
    FactoryGirl.create(:receivable, :with_boleto_cobrato),
    FactoryGirl.create(:receivable),
  ]
end
let(:invoice) { FactoryGirl.create(:invoice, :received, payment_information: payment_information, receivables: receivables) }
```

Mais detalhes: 
- [Better specs lets](https://www.betterspecs.org/#let)
- [when-to-use-rspec-let](https://stackoverflow.com/questions/5359558/when-to-use-rspec-let/5359979#5359979)

### Use context
Para deixar melhor organizado o codigo e mais claro. Sempre começar com "when", "with" ou "without", exceto casos onde o contexto é aninhado. Aninhar no maximo 2 contextos.

Exemplo:
```
context "when invoice is received" do 
```

### Use subject
Sempre que um teste chamar varias vezes o mesmo subject, seja uma classe ou função, deve ser usado subject. [DRY](https://www.infoq.com/br/news/2012/07/DRY-acoplamento-duplicacao/)

Exemplo:
```
subject { assigns('message') }
it { is_expected.to match /it was born in Billville/ }
```
