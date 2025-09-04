---
title: "Algoritmos Desacoplados: O Guia Prático do Strategy Pattern para um Código Kotlin Mais Inteligente."
layout: post
tags: [design patterns, stategy, kotlin]
category: Design Patterns
image:
  path: https://refactoring.guru/images/patterns/diagrams/strategy/structure.png
  alt: 
---

---

Vamos falar sobre um problema muito comum que todos desenvolvedores enfrentam: blocos condicionais (_if/else_ ou _when_) que não param de crescer. Conforme novas regras de negócio são adicionadas, aquele método que era simples e elegante começa a se tornar um **monstro de complexidade**. 😱

Felizmente, existe uma solução clássica e poderosa para isso: o **_Design Pattern Strategy_**. 🎉

Vamos primeiro entender a **teoria** por trás do padrão _Strategy_ e, em seguida, refatorar um código simples em **_Kotlin_** com **_Spring Boot_**, saindo de uma abordagem cheia de _ifs_ para uma solução **limpa**, **extensível** e de **fácil manutenção**.

## Entendendo o Conceito do Padrão Strategy

Antes de codificar, vamos à definição. O Padrão __Strategy_ é um padrão de projeto **comportamental** (_Behavioral Design Pattern_) que permite definir uma família de algoritmos, encapsular cada um deles em classes separadas e torná-los intercambiáveis.

A ideia central é permitir que o algoritmo (a "estratégia") varie independentemente dos clientes (o "contexto") que o utilizam.

## Os Componentes Principais

O padrão _Strategy_ é composto por **três elementos fundamentais**:

* **Contexto (Context)**: É a classe que precisa executar uma determinada ação, mas que pode realizar essa ação de diferentes maneiras. Em vez de implementar todas as variações da ação internamente, o Contexto mantém uma referência a um objeto _Strategy_. **O Contexto não sabe os detalhes da implementação da estratégia, ele apenas sabe como se comunicar com ela através da interface**.

* **_Strategy_ (Interface ou Classe Abstrata)**: Define a interface comum para todos os algoritmos suportados. O Contexto usa essa interface para chamar o algoritmo implementado por uma **Estratégia Concreta**.

* **_Concrete Strategy_ (Estratégia Concreta)**: Cada classe de Estratégia Concreta implementa um **algoritmo específico**, seguindo o contrato definido pela interface _Strategy_. 
> ⚠️ **É aqui que a lógica real de cada variação reside** ⚠️.

## Quando Usar o Padrão Strategy?

* Quando você tem **uma classe** com um grande bloco condicional (_if/else_ ou _switch/case_) que seleciona **diferentes comportamentos com base em uma condição**.

* Quando você precisa de **diferentes variações de um mesmo algoritmo**.

* Quando você quer permitir que o **cliente escolha o algoritmo a ser usado em tempo de execução**.

* Para **isolar lógicas de negócio complexas** e voláteis do restante da aplicação, facilitando a manutenção e os testes.

> Agora que entendemos a teoria, vamos aplicá-la em um exemplo prático.

## O Cenário: Um Serviço de Pagamentos

Imagine que estamos construindo uma API REST para processar pagamentos. Inicialmente, nosso serviço precisa aceitar pagamentos via Cartão de Crédrito, Cartão de Débito e PIX.

### A Implementação Inicial (_v1_) - O Problema ☠️

Uma primeira abordagem, mais direta, poderia ser algo assim. Primeiro, definimos o nosso _PaymentRequest_ e um _enum_ para os métodos de pagamento.

```kotlin
data class PaymentRequest(
    val method: PaymentMethod,
    val value: Long
)
```

```kotlin
enum class PaymentMethod { CREDIT_CARD, DEBIT_CARD, PIX }
```

Em seguida, criamos um _PaymentService_ que centraliza toda a **lógica de decisão e execução**.

```kotlin
@Service
class PaymentService {

    fun pay(method: PaymentMethod, value: Long) {

        if (method == PaymentMethod.CREDIT_CARD) {
           println("Processing credit card payment of $${value.toFloat()/100}")
           return
       }
        if (method == PaymentMethod.DEBIT_CARD) {
           println("Processing debit card payment of $${value.toFloat()/100}")
           return
       }
        if (method == PaymentMethod.PIX) {
           println("Processing PIX payment of $${value.toFloat()/100}")
            return
       }

    }

}
```

E, por fim, um _Controller_ para expor nossa funcionalidade.

```kotlin
@RestController
@RequestMapping("/v1")
class Controller(
    private val paymentService: PaymentService
) {

    @PostMapping("/pay")
    fun pay(@RequestBody request: PaymentRequest) {
        paymentService.pay(request.method, request.value)
    }

}
```

Funciona? Sim. Mas **qual é o problema aqui?**

* **Violação do Princípio Aberto/Fechado (_Open/Closed Principle_)**: Este princípio diz que nosso código deve ser **"aberto para extensão, mas fechado para modificação"**. No nosso _PaymentService_, se precisarmos adicionar um novo método de pagamento (como Boleto ou Criptomoedas), **teremos que modificar a classe, adicionando mais um _if_**.

* **Baixa Coesão**: A classe _PaymentService_ está assumindo **múltiplas responsabilidades**. Ela sabe como processar todos os tipos de pagamento. Com o tempo, ela pode se tornar uma **_"God Class"_ ♱**.

## A Solução: Refatorando com o Padrão _Strategy_ 🏆

> Vamos aplicar os conceitos do padrão _Strategy_ para resolver esses problemas.

## **Passo 1**: Definir a Interface da Estratégia (_Strategy Interface_)

Primeiro, criamos um **contrato comum** que todas as nossas estratégias de pagamento deverão seguir.

```kotlin
interface PaymentStrategy {
    fun pay(value: Long)
}
```

## **Passo 2**: Criar as Estratégias Concretas (_Concrete Strategies_)

Agora, movemos a lógica de cada _if_ para sua **própria classe**, implementando a interface _PaymentStrategy_.

### Estratégia para Cartão de Crédito

```kotlin
class CreditCardStrategy: PaymentStrategy {
    override fun pay(value: Long) {
        println("Processing credit card payment of $${value.toFloat()/100}")
    }
}
````

### Estratégia para Cartão de Débito

```kotlin
class DebitCardStrategy: PaymentStrategy {
    override fun pay(value: Long) {
        println("Processing debit payment of $${value.toFloat()/100}")
    }
}
````

### Estratégia para PIX

```kotlin
class PixStrategy: PaymentStrategy {
    override fun pay(value: Long) {
        println("Processing PIX payment of $${value.toFloat()/100}")
    }
}
````

> ⚠️ Um ponto importante: O _println_ é apenas um **exemplo**. ⚠️

Você pode estar pensando: _"Mas a lógica dentro de cada estratégia é apenas um println. Por que criar uma classe inteira para isso?"_.

Essa é uma ótima observação. Em nosso exemplo, usamos um _println_ para **manter o foco no padrão**. No entanto, em um cenário real, cada método _pay_ conteria **lógicas muito mais complexas**:

* **_CreditCardStrategy_**: Poderia chamar a _API_ de um _gateway_ de pagamento, lidar com retentativas, calcular taxas específicas de cartão de crédito e salvar a transação no banco de dados.

* **_PixStrategy_**: Poderia gerar um _QR Code_, consultar o _status_ do pagamento em um serviço de mensageria (como _Kafka_ ou _RabbitMQ_) e notificar o usuário sobre a confirmação.

* **_DebitCardStrategy_**: Poderia se comunicar diretamente com um serviço do banco, realizar validações de saldo e registrar logs de auditoria detalhados.

Imagine toda essa complexidade misturada dentro de um único método com vários _if/else_. **A separação em classes (Estratégias) não só organiza o código, mas também isola essas complexidades**, tornando cada fluxo de pagamento independente, mais fácil de desenvolver, testar e dar manutenção.

## **Passo 3**: Criar o Contexto (Nosso novo _Service_)

Agora precisamos de uma classe que **decida qual estratégia usar** em tempo de execução. Este é o nosso Contexto.

```kotlin
@Service
class PaymentStrategyService {

    private val strategies = mapOf(
        CREDIT_CARD to CreditCardStrategy(),
        DEBIT_CARD to DebitCardStrategy(),
        PIX to PixStrategy()
    )

    fun pay(method: PaymentMethod, value: Long) {
        strategies[method].pay(value)
            ?: throw IllegalArgumentException("Unsupported payment method: $method")
    }

}
```

🪄 **A mágica acontece aqui!** 🪄

Em vez de um bloco _if/else_, usamos um _Map_ para associar cada _PaymentMethod_ à sua implementação de _PaymentStrategy_. **O método _pay_ agora tem uma única responsabilidade**: encontrar a estratégia correta e delegar a execução para ela.

### **Passo 4**: O Novo Controller (_v2_)

Finalmente, criamos um novo _endpoint_ (_/v2_) que **utiliza nosso serviço refatorado**.

```kotlin
@RestController
@RequestMapping("/v2")
class StrategyController(
    private val paymentService: PaymentStrategyService
) {

    @PostMapping("/pay")
    fun pay(@RequestBody request: PaymentRequest) {
        paymentService.pay(request.method, request.value)
    }

}
```

## Vantagens da Nova Abordagem

Ao comparar a _v2_ com a _v1_, os benefícios são claros:

* **Fidelidade ao Princípio Aberto/Fechado**: A principal vantagem é a facilidade de extensão. Para adicionar um novo método de pagamento, como "Boleto", o processo é incrivelmente simples e seguro, sem a necessidade de alterar código que já funciona:
1. Crie a nova classe _BoletoStrategy_ que implementa _PaymentStrategy_.
2. Adicione o valor BOLETO ao __enum PaymentMethod__.
3. Adicione a nova entrada no _map_ do _PaymentStrategyService_: ```BOLETO to BoletoStrategy()```.
4. **Pronto!** O novo método de pagamento está totalmente integrado ao fluxo, sem que tenhamos modificado a lógica principal do serviço.

* **Alta Coesão e Baixo Acoplamento**: Cada estratégia cuida da sua própria lógica complexa, e o serviço apenas orquestra a chamada.

* **Facilidade de Teste**: Podemos testar _PixStrategy_ ou _CreditCardStrategy_ de forma totalmente isolada, com todas as suas particularidades.

* **Legibilidade**: A intenção do código em _PaymentStrategyService_ é muito mais clara do que uma longa cadeia de _ifs_.

## Conclusão

O padrão 
_Strategy_ é uma ferramenta fantástica para transformar condicionais complexos em um código flexível, manutenível e limpo. Ao encapsular algoritmos variantes em seus próprios objetos, ganhamos um sistema muito mais robusto e preparado para futuras mudanças.

> 💭 Que tal tentar aplicar o _Strategy_ no seu próximo desafio? 💭

---

**Bibliografia**

Livro: Gamma, E., Helm, R., Johnson, R., & Vlissides, J. (1994). Design Patterns: Elements of Reusable Object-Oriented Software. Addison-Wesley. (O livro original do "Gang of Four" que introduziu o padrão).

Website: Refactoring.Guru. Strategy Design Pattern. Uma excelente fonte online com explicações claras e exemplos em várias linguagens.

Vídeo: Build & Run. Passo a passo: Implementando o Design Pattern Strategy no Java. Um guia prático mostrando a implementação do padrão.

Vídeo: Renato Augusto. Padrão de Projeto Strategy: Tudo o Que Você Precisa Saber Sobre Esse Design Pattern (GOF). Uma explicação conceitual detalhada sobre o funcionamento e os benefícios do Strategy.

