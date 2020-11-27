---
layout: post
author: Allan Soares Duarte
title: "Arquitetura hexagonal: portas e adaptadores"
---

Vamos supor que temos a necessidade de construir e manter um software voltado para gerenciar armazéns logísticos que contém uma quantidade enorme de recursos e estes necessitam de diversas integrações com serviços externos e internos.

O desenvolvimento de um software desse tipo é bastante difícil e desafiador — precisamos que nosso software seja robusto, escalável, tolerante a falhas, com bom desempenho e ao mesmo tempo queremos que seja prático para a criação de novas características, testar, manter e dar suporte.

Todos esses problemas são inevitáveis ​​com a evolução do software, mas com certeza podem ser simplificados ou mesmo evitados ao decidir os estilos arquiteturais de um software.

<p class="center"><img src="/assets/image/posts/abstract-3d-cube.jpg" alt="{{page.title}}"></p>

# Por que portas e adaptadores?

Mesmo com o desenvolvimento de um software que necessite inicialmente uma escala relativamente pequena, ainda precisará projetá-lo primeiro — e projetá-lo adequadamente. Quanto mais cedo você começar a se preocupar com sua arquitetura, mais cedo poderá se beneficiar dela e mais tarde aparecerão muitos problemas causados ​​por arquitetura incorreta ou [Big Ball of Mud](http://www.laputan.org/mud/), que ainda é o projeto de software dos mais populares.

A principal idéia da arquitetura de portas e adaptadores é que o aplicativo que estamos construindo é uma área fechada. Isso significa que toda a sua lógica de negócios deve ser separada dos detalhes técnicos. Muitas vezes, a arquitetura é sobre os limites, assim como portas e os adaptadores.

Caso você se atenha às portas e adaptadores desde o início, essa abordagem deve ajudá-lo a manter sua lógica de negócios separada e facilmente testada e independente de tecnologia de terceiros — você pode escrever uma porta e um adaptador para qualquer serviço de terceiro que você está usando, para que possa ser facilmente estendido ou alternado em favor de outro.

# Hexagonal

A arquitetura de portas e adaptadores também é conhecida por arquitetura hexagonal. De acordo com essa terminologia, a parte interna do seu software — o local em que você coloca sua lógica de negócios — é hexágono enquanto os adaptadores são colocados ao seu redor.

<img src="/assets/image/posts/adaptadores.png" alt="Adaptadores">

O hexágono não deve conter nenhuma referência a outra estrutura, serviço externo, bibliotecas etc. — todos esses elementos devem ser adaptadores. Ao mesmo tempo, a arquitetura não o prescreve para projetar seu hexágono de uma certa maneira — você pode usar a arquitetura Layer, Onion, DDD ou qualquer outra arquitetura adequada ou pode ser uma lógica de negócios pura, sem sofisticação — é você decide.

Por que hexágono? Qualquer figura geométrica com limites poderia funcionar, mas o hexágono representa melhor o conceito de que você tem portas nas bordas do seu aplicativo e adaptadores atrás dele. Da mesma forma, é uma figura assimétrica e descreveremos abaixo por que é importante.

# Domain-Driven-Design

Domínio é um extenso assunto abordado no popular livro do Eric Evans — Domain-Driven-Design. Por em quanto vamos nos ater que Domínio refere-se ao assunto específico para o qual o projeto está sendo desenvolvido. O objetivo do DDD é limitar a complexidade de uma solução, adaptando-a o mais próximo possível de um domínio de negócio, com a ajuda de especialistas nesse domínio. Vamos a alguns possíveis domínios de nosso software:

- Perfil
- Estoque
- Produtos
- Catálogo
- Pagamento
- Assinatura
- Notificações
- Autenticação

| Vamos supor que nosso software necessite notificar vários atores em diferentes circunstâncias. Como desenvolver esta característica de maneira abstrata?

<hr />

# Portas

Toda vez que é necessário interagir com algo além da lógica da aplicação, é necessário agrupar essas ações e descrevê-las em uma porta. A porta é a borda do hexágono e deve ser parte integrante e essencial da aplicação.

A nomeação das portas é muito importante, geralmente é nomeado de acordo com a missão ou serviço. Alguns dos exemplos:

- PushNotifications
- Search
- Persistence
- Authentication

A maioria das linguagens de programação geralmente contém recursos de interfaces ou protocolos, permitindo a construção de uma porta.

Agora, vamos ver como podemos implementar a realização da porta para o Elixir usando sua capacidade de criar comportamentos:

{% highlight elixir %}
defmodule Core.PushNotifications do
    @moduledoc """
    Port for sending push notifications.
    """

    @type message :: %{title: String.t(), body: String.t()}
    @type payload :: Keyword.t
    @type recipients :: [map]

    @adapter :core |> Application.fetch_env!(__MODULE__) |> Keyword.fetch!(:adapter)

    @callback send_notifications(message, recipients, payload) :: {:ok, [map]} | {:error, any}

    defdelegate send_notifications(message, recipients, payload), to: @adapter
end
{% endhighlight %}

O exemplo acima nada mais é do que uma abstração para o uso de push notifications no `Core`. Declaramos o comportamento e um retorno de chamada que especifica o que enviamos e o que podemos esperar como resultado. A implementação exata — adaptador — deve ser passada como uma mensagem no aplicativo já que estamos em uma situação de N adaptadores, como:

{% highlight elixir %}
config :core, Core.PushNotifications, adapter: PushNotifications.APNS
{% endhighlight %}

Se você deseja chamar essa porta do seu aplicativo basta usar a função delegada:

{% highlight elixir %}
defmodule Core do
  alias Core.PushNotifications

  def register_user(params) do
    # business logic ...
     result = PushNotifications.send_notifications(message, recipients, payload)
    # handle the result somehow
  end
end
{% endhighlight %}

Como você pode ver, no `Core` não sabemos nada sobre os detalhes da implementação — apenas enviamos notificações para usuários. No caso ideal, precisamos mover qualquer função impura, qualquer efeito colateral para a borda do sistema — para adaptadores e chamá-los apenas usando portas.

# Driver Adapters

Adaptadores são componentes que são colocados fora do seu aplicativo e do seu hexágono. Eles devem representar a tecnologia, serviço, biblioteca que você precisa para interagir através da porta.

Nós especificamos dois tipos de adaptadores: Driver e Driven.

Os primeiros são algo do lado esquerdo da imagem acima. Pode ser uma página HTML, ponto de extremidade da API, aplicativo CLI, GUI ou qualquer outra camada que direcione seu aplicativo. Isso também significa que o adaptador de driver deve usar uma interface de porta de driver para que seu aplicativo receba uma solicitação independente de tecnologia em suas bordas.

Vamos supor que também tenhamos um aplicativo da web que use nosso `Core`. Se queremos notificar qualquer evento de uma entrega precisamos chamar uma função `Core.register_user/1` de dentro do nosso controlador. Nesse caso, o `UserController` é o nosso adaptador de driver e o `Core` é o aplicativo chamado. Felizmente, no Elixir, temos especificações de tipo que podem desempenhar um papel de especificação da porta do driver, para que você sempre possa ver o que precisamos enviar e o que devemos esperar em resposta.

# Driven Adapters

Um adaptador Driven implementa uma interface fornecida por uma porta acionada. Isso significa que agora o adaptador acionado depende da nossa aplicação, mas não do contrário. O mesmo que o Driver, este adaptador também deve ser colocado fora do nosso hexágono.

Exemplos comuns são:

- Adaptadores de persistência — bancos de dados SQL, NoSQL ou mesmo armazenamento em memória ou arquivo
- Adaptadores de cache — Redis, Memcached, ETS ou armazenamento na memória
- Adaptadores de email — SMTP ou serviços de terceiros
- Adaptadores da fila de mensagens
- APIs de terceiros

<hr />

Vamos continuar com a solução de push notifications que começamos antes. Agora, para implementar o adaptador de driver, precisamos usar a porta `Core.PushNotifications` e seu retorno de chamada `send_notifications`. Adaptaremos a realização do envio de notificação pelo APNS pela especificação que nos foi fornecida por esta porta:

{% highlight elixir %}
defmodule PushNotifications.APNS do
  @moduledoc "APNS adapter for push notifications"
  @behaviour Core.PushNotifications

  @impl true
  def send_notifications(message, recipients, payload) do
    {:ok, recipients
    |> Enum.map(fn r -> build_notification(message, r, payload) end)
    |> Pigeon.APNS.Notification.push()}
  end

   defp build_notification(message, recipient, payload) do
     Pigeon.APNS.Notification.new(message, recipient.device_token, payload)
   end
end
{% endhighlight %}

Agora, nossas push notifications estão quase concluídas. Sempre podemos alterar a implementação — por exemplo, do APNS para o Firebase — ou usar a biblioteca de terceiros sem alterar nosso aplicativo principal — então podemos dizer que essa é uma abordagem independente de uma tecnologia de terceiro.

# Testes

Obviamente, o principal benefício da arquitetura de portas e adaptadores é a testabilidade aprimorada. Em vez de mocar manualmente chamadas para os provedores do mundo real, precisamos apenas criar um adaptador de teste que satisfaça as condições de teste. No caso perfeito, todo adaptador acionado deve ter um analógico de teste, bem como todos os comportamentos das portas do driver, devem ser testados. Vamos escrever um adaptador de teste para a porta PushNotifications:

{% highlight elixir %}
defmodule PushNotifications.TestAdapter do
  @moduledoc "Test adapter for push notifications"
  @behaviour Core.PushNotifications

  @impl true
  def send_notifications(message, recipients, payload) do
     {:ok, [%{message: message, payload: payload, recipients: recipients}]}
  end
end
{% endhighlight %}

Como você podemos ver, não estamos enviando dados para o mundo exterior, mas usamos uma função pura. Em caso de entrada, saberemos com certeza sua saída. Agora, quando testamos o módulo `Core`, apenas precisamos selecionar o adaptador de teste como a implementação da interface `PushNotifications`. No ecossistema Elixir, temos uma ótima biblioteca chamada `Mox` que pode ser usada para esse caso:

{% highlight elixir %}
Mox.defmock(PushNotifications.TestMock, for: Core.PushNotifications)

defmodule CoreTest do
  use Core.DataCase, async: true
  import Mox

 # Make sure mocks are verified when the test exits
  setup :verify_on_exit!

  test "register/1" do
     stub_with(PushNotifications.TestMock, PushNotifications.TestAdapter)
     assert {:ok, _} = Core.register_user(some_params)
  end
end
{% endhighlight %}

Neste exemplo, podemos ver que não estamos enviando notificações ao mundo real, mas sim usando a simulação de teste local. Podemos mudar o adaptador de teste para qualquer finalidade de teste, se quisermos.

A partir de agora, você testará o comportamento da sua porta de driver. Como a próxima etapa, você pode testar exatamente a implementação do adaptador sem nenhuma lógica externa anexada — basta verificar se sua implementação está funcionando corretamente, como previsto. Quanto ao teste de integração, você pode escolher entre os adaptadores do mundo real ou pode usar alguns adaptadores de teste para esse fim — depende de você.

# Prós e Contras

Abordamos o básico da arquitetura de portas e adaptadores. Vamos resumir o que temos:

## Prós

- Testabilidade
- Abordagem independente da tecnologia — você pode terceirizar soluções tecnológicas
- Isolamento do código puro e do código impuro
- Isolamento de efeitos colaterais
- Manutenabilidade

## Contras

- Pode ser uma sobrecarga tecnológica, especialmente para um software que necessite pouca escalabilidade
- Pode não ser necessário se tiver certeza de que as tecnologias do seu projeto permanecerá a mesma ao longo dos anos

É uma boa idéia a arquitetura de portas e adaptadores quando ficar claro que o software usará muitos serviços que poderão ser substituídos no futuro. Essa abordagem oferece muitos benefícios e permite tornar o software ainda mais testável e flexível.
