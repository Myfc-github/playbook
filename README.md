# Organização de código

Um dos principais desafios que temos em nossos projetos Rails é saber onde colocar o código e como nomear as classes e módulos. Abaixo uma referência com as principais dúvidas que temos para que possamos padronizar nossos projetos e minimizar as dúvidas.

## Onde colocar classes e modulos que não são models, views ou controllers?

- `app/concerns`: Não é uma boa prática, apenas fragmenta o código. Tentar evitar ao máximo.
- `app/services`: Classes com regras de negócio e interações com serviços externos. Não deve ter `Service` no nome e nem ser um substantivo como `Order`. Cada service deve executar uma única função e ter um verbo no nome, ex: `CancelOrder`.

## app/services/

A maior parte dos artigos sobre organização de código em Rails sugere que seja colocado em services todo código que não se encaixa em `models`, `views` ou `controllers`, uma definição fácil de entender é a seguinte:

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

# Boas práticas

## RESTful controllers

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

## Quais tipos de teste escrever

- Use request specs para testar controllers, [não use controller specs](http://rspec.info/blog/2016/07/rspec-3-5-has-been-released/).

PS: Não devemos mais utilizar Feature test, nem Controller test.

## Factories

### Preencha os objetos com o mínimo necessário

As factories devem sempre preencher o mínimo de informações necessárias para criar um objeto válido, nunca um objeto com todos os dados possíveis. 

Podem chegar vários bugs em produção, pois a factory preenche todos os campos no teste, mas em produção chegam campos `nil` que quebram as views por exemplo.


# Ruby Style Guide

This guide is for edge cases and subjective matters that cannot be automated using tools like Rubocop

## High level guidelines

* Aim for simplicity - ["I think most programmers spend the first 5 years of their career engaging complexity, and the rest of their lives learning simplicity."](https://twitter.com/buzz/status/7203012768).
* Always remember the [Yagni](https://martinfowler.com/bliki/Yagni.html) mantra.
* Don't rewrite existing code to follow this guide.
* Don't violate a guideline without a good reason.
* A reason is good when you can convince a teammate.

## Extra resources

Some extra resources that helped to create this guideline:

* https://github.com/thoughtbot/guides

