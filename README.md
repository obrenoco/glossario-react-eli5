Essa é uma tradução da thread [Explain Like I'm Five #46](https://github.com/reactwg/react-18/discussions/46), aberta pelo time React.

Aqui são explicados conceitos da biblioteca em uma linguagem extremamente simples para a comunidade Dev, sendo muitos desses termos utilizados dentro do contexto das novas atualizações do React 18.

A tradução para o português tem como objetivo promover um maior acesso do público lusófano aos conteúdos React.

> tl; dr A terminologia técnica complicada muitas vezes representa um obstáculo para as pessoas aprenderem novas tecnologias. Este tópico contém definições em inglês simples e explain-like-I'm-five dos termos e conceitos usados em outras discussões. Role até os comentários para encontrar um termo e explicações relacionadas a ele

---

 Em outras palavras, vamos ajudar os novos desenvolvedores do React a ficarem tão animados com este lançamento quanto nós (time React)!

---

## ✨ Concurrency (Concorrência)

Concorrência significa que as tarefas podem se sobrepor.

Vamos usar ligações telefônicas como analogia.

Sem simultaneidade significa que só posso ter uma conversa telefônica por vez. Se eu estiver falando com Alice e Bob me ligar, terei que encerrar a ligação com Alice antes de falar com Bob.

![](https://user-images.githubusercontent.com/810438/121394782-9be1e380-c949-11eb-87b0-40cd17a1a7b0.png)

Simultaneidade significa que posso ter mais de uma conversa ao mesmo tempo. Por exemplo, posso colocar Alice em espera, conversar um pouco com Bob e depois voltar a falar com Alice.

![](https://user-images.githubusercontent.com/810438/121394880-b4ea9480-c949-11eb-989e-06a95edb8e76.png)

Observe que a simultaneidade não significa necessariamente que eu falo com * duas pessoas ao mesmo tempo *. Significa apenas que, a qualquer momento, posso estar em várias ligações e escolho com quem falar. Por exemplo, com base em qual conversa é mais urgente.

Agora, para traduzir a analogia, no caso do React, "ligações" são suas ligações `setState`. Anteriormente, o React só podia funcionar em uma atualização de estado por vez. Portanto, todas as atualizações eram "urgentes": quando você começa a renderizar novamente, não pode parar. Mas com `[<startTransition />](https://github.com/reactwg/react-18/discussions/41)`, você pode marcar uma atualização não urgente como uma transição.

![](https://user-images.githubusercontent.com/810438/121396132-f760a100-c94a-11eb-959c-b95a6647d759.png)

Você pode pensar nas atualizações "urgentes" do setState como semelhantes à chamadas telefônicas urgentes (por exemplo, seu amigo precisa da sua ajuda), enquanto as transições são como conversas relaxadas que podem ser colocadas em espera ou mesmo interrompidas se não forem mais relevantes.

## ✨ Suspense

[# 28](https://github.com/reactwg/react-18/discussions/28) é um tópico muito útil com duas analogias principais para Suspense

[@Skn0tt](https://github.com/Skn0tt)

> Imagine um componente que precisa fazer alguma tarefa assíncrona antes de ser renderizado, por exemplo, buscando alguns dados. Antes do Suspense, tal componente manteria um estado isLoading e retornaria algum tipo de fallback (um estado vazio, esqueleto, spinner, ...) com base nele. Com o Suspense, um componente pode agora, durante a renderização, gritar " ESPERA! Ainda não estou pronto. Não continue a renderizar, por favor - aqui está um pager, vou enviar um ping quando estiver pronto!1". O React vai então subir na árvore para encontrar o limite do Suspense mais próximo, que é definido por você e contém o fallback que agora será renderizado até que o pager toque.

[@gaearon](https://github.com/gaearon) 

> Outra maneira que gosto de explicar é que é como throw / catch, mas para estados de carregamento. Quando um componente diz "Não estou pronto" (situação excepcional, como lançamento), o bloco Suspense mais próximo acima (como captura) mostra o fallback. Então, quando a Promise é resolvida, nós "tentamos novamente" a renderização. Uma diferença chave com a programação tradicional baseada em Promise é que as Promises não são expostas ao código do usuário. Você não lida com eles diretamente. Você não faz um Promisse.All neles ou um .then neles ou qualquer coisa assim. Em vez de "esperar que a Promise resolva e execute algum código do usuário", o paradigma é "O React tentará novamente a renderização quando for resolvido". De certa forma, é mais simples, mas leva algum "desaprendizado" porque o usuário perde algum controle ao qual ele poderia estar acostumado da programação tradicional baseada em Promise.

## ✨ Batching e Automatic Batching - Loteamento e Loteamento Automático

### Batching

Imagine que você está fazendo um café da manhã.

Você vai ao mercado para comprar leite. Você volta. Então você percebe que precisa de granola. Você vai ao mercado para comprar granola. Você volta. Então você percebe que precisa de cookies. Você vai ao mercado e compra cookies. Finalmente, você pode comer seu café da manhã.

Mas certamente existe uma maneira mais eficiente de fazer isso! Em vez de comprar cada item no momento em que você pensa, é uma boa ideia compilar primeiro uma lista de compras. Então você vai ao mercado, pega tudo o que precisa e prepara o seu café da manhã.

Isso é batching.

No caso do React, cada item individual é uma chamada `setState`.

```jsx
setIsFetching(false);
setError(null);
setFormStatus('success')
```

O React pode renderizar novamente ("ir à loja") após cada uma dessas chamadas. Mas não é eficiente. Seria melhor registrar todas as atualizações de estado primeiro. E então, quando você terminar de atualizar todas as partes do estado, renderize novamente uma vez.

O desafio é * quando * o React deve fazer isso. Como ele pode saber que você concluiu a atualização do estado?

Para manipuladores de eventos, é fácil:

```jsx
setIsFetching(false);
setError(null);
setFormStatus('success')
```

React é o que chama seu gerenciador de cliques.

```jsx
// Dentro do React
handleClick()
```

Portanto, o React será renderizado novamente logo após:

```jsx
// Dentro do React
handleClick()
updateScreen();
```

É assim que o React sempre funcionou.

### Automatic batching

Voltando a este exemplo:

```jsx
setIsFetching(false);
setError(null);
setFormStatus('success')
```

E se isso não acontecer durante um evento? Por exemplo, talvez este código seja executado como resultado de uma busca. O React não tem ideia de quando você vai "parar" os estados de configuração. Portanto, é necessário reservar algum tempo para atualizar a tela.

Isso é semelhante a como talvez você não esteja tomando café da manhã, mas apenas tendo desejos aleatórios ao longo do dia. Você vai ao mercado no momento em que sente que está com fome? Ou você espera um pouco (digamos, 30 minutos) e depois vai ao mercado com uma lista de tudo o que descobriu até agora?

Anteriormente, o React costumava renderizar novamente neste caso. Isso significa que você obterá três atualizações de tela para três chamadas `setState`. Isso não é muito eficiente.

Com o envio em lote automatizado no React 18, ele sempre os agrupará. Na prática, significa que o React irá "esperar um pouco" (o termo técnico se "até o final da tarefa ou da microtarefa"). Em seguida, o React atualizará a tela.

## ✨ Hydration (Hidratação)

A explicação de Dan sobre hidratação em [#37](https://github.com/reactwg/react-18/discussions/37) (no subtítulo "o que é SSR?") vai direto ao ponto

Aqui está um trecho:

> O processo de renderizar seus componentes e anexar manipuladores de eventos (event handlers) é conhecido como "hydration" (hidratação). É como regar o HTML "seco" com a "água" da interatividade e dos manipuladores de eventos.

Para obter mais contexto, consulte o tópico do [tweet de Bruno Nardini](https://twitter.com/megatroom/status/1402434746186178563)

> A analogia de que gosto para "hidratação" é que ela carrega a seco e, em seguida, aplica-se óleo nas engrenagens para movê-la.

[@meganesu](https://github.com/meganesu)

> Pelo que entendi:
> A hidratação é usada apenas com renderização do lado do servidor. Quando você usa SSR, o processo de renderização de componentes em seu navegador tem duas etapas:
> 1. Envie o HTML e CSS renderizado do lado do servidor não interativo para o navegador. Dessa forma, seus usuários têm algo para olhar enquanto esperam o JS para sua página carregar.
> 2. "Hidrate" esse HTML e CSS quando seu JS terminar de carregar, para que os usuários possam interagir com as coisas em sua página (botões de clique, menus suspensos de alternância etc.).

(Ainda estou aprendendo sobre SSR, então, por favor, lmk se eu entendi algo errado!)

## ✨ SSR - Server-Side Rendering (Renderização no lado do Servidor)

[@laurieontech](https://github.com/laurieontech)

> SSR pode ser explicado bem com alimentos.
> Suponha que você vá a um bufê de saladas e obtenha cada um de seus ingredientes para fazer sua própria salada. Você os junta, sobrepõe as coisas da maneira que quiser e depois comê-las. Essa é a renderização do lado do cliente. Você está juntando as peças (HTML, CSS, JS, você mesmo).
> Como alternativa, você pode ir ao bufê de saladas e comprar uma salada pré-preparada. Está tudo pronto para você, basta comê-lo. Isso é renderização do lado do servidor. Você obtém o HTML pronto para renderização entregue diretamente do servidor.

[@sophiebits](https://github.com/sophiebits)

> A maioria das pessoas pensa no React como sendo executado no cliente (ou seja, no navegador do usuário). Quando os componentes são renderizados, isso resulta na alteração do HTML que está na página. Quando um usuário visita uma página da Web usando React, primeiro precisamos baixar React e o código JavaScript para cada componente React e, em seguida, executar esse código em seu navegador para descobrir qual HTML queremos mostrar na tela. Assim que isso acontecer, o usuário poderá ver o aplicativo e interagir com ele.
> A renderização do lado do servidor (SSR) é uma ferramenta projetada para acelerar esse processo para que as páginas que usam o React carreguem mais rápido. Ao usar o SSR, primeiro executamos os componentes no servidor que está enviando os arquivos da página ao usuário. Pegamos o HTML gerado por esses componentes e enviamos * esse HTML * para o navegador do usuário. Assim que chegar, o usuário poderá ver todo o conteúdo da página. Observe que o HTML por si só é um instantâneo da aparência inicial da página e não é interativo por si só. Portanto, ainda precisamos fazer o download do React e do código JS para cada componente do React e, em seguida, precisamos dizer ao React para "assumir o controle" do HTML que já está na página, o que é chamado de hidratação. Assim que terminar, o usuário poderá ver todo o conteúdo da página e interagir com ele.
> SSR ajuda a tornar a página inicial visível mais cedo, mas não a torna interativa mais cedo porque você ainda precisa esperar para baixar e executar todo o mesmo código JS - com SSR, estamos executando todos os componentes no servidor agora, mas nós também está executando todos eles no cliente ainda. [O próximo trabalho em Server Components](https://reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html) - um novo experimento da equipe React - ajudará faça com que você possa executar alguns componentes * apenas * no servidor, o que pode ajudar nisso, mas não estará pronto por um tempo.)
> Para usar o SSR, você precisa fazer duas coisas: (a) certificar-se de que cada componente React em sua base de código usa apenas recursos que funcionam no servidor e no cliente (existem alguns requisitos, como não usar o objeto `window` que está disponível apenas em navegadores) e (b) alterar seu código do lado do servidor para chamar React e fazer algo com o HTML resultante. Algumas estruturas como Next.js vêm com suporte SSR pronto para uso, o que significa que eles fizeram a parte (b) para você.

## ✨ Passive Effects (Efeitos Passivos)

Existem dois tipos de efeitos:

- `useEffect` =" efeitos "
- `useLayoutEffect` =" efeitos de layout "

Às vezes, quando você diz "efeitos", não está claro se você se refere a ambos ou apenas ao primeiro tipo. É por isso que o primeiro tipo (`useEffect`) às vezes é chamado de" passivo ".

Evitamos essa terminologia em qualquer lugar dos documentos, mas a usamos entre as discussões da equipe principal e talvez alguns comentários no GitHub. Eu acho que é onde os mantenedores da biblioteca podem ter aprendido. Eu não esperaria que os autores de aplicativos conhecessem ou se importassem com essa terminologia - mas se você ver, tudo o que significa é `useEffect`, nada mais.

## ✨ Discrete Events (Eventos Discretos)

[@gaearon](https://github.com/gaearon)

> Eventos "discretos" não é um conceito que usamos em qualquer lugar dos documentos. Não é um conceito que esperamos que os desenvolvedores de aplicativos React conheçam ou se preocupem.
> No entanto, ele surge quando explicamos como funciona o [automatic batching](https://github.com/reactwg/react-18/discussions/21) (lote automático). O que temos que esclarecer é que para alguns eventos, como cliques, garantimos que a tela é atualizada antes que o próximo evento aconteça. Isso é importante para interfaces de usuário como contadores (imagine o usuário pressionando o botão duas vezes muito rápido) ou formulários (onde o envio de um formulário pode desativá-lo e, se não acontecer antes do segundo clique, você corre o risco de enviar um formulário duas vezes).
> Chamamos eventos como esses de "discretos". Isso significa que cada evento é ** separadamente ** pretendido pelo usuário. Se eu clicar duas vezes, é porque * pretendo * clicar duas vezes. Se eu digitar "h", "e", "l", "l", "o", é porque eu * pretendo * digitar "hello".
> Isso é diferente de eventos como o movimento do mouse. Posso mover o mouse sobre dez coordenadas diferentes, mas não quero * dizer * intencionalmente qualquer uma dessas coordenadas separadas. Como usuário, não tenho ideia de quantos eventos mousemove acabei de realizar. Eu apenas movi o mouse. Chamamos esses eventos de "contínuos" - o que importa é o *último*, não *quantos* aconteceram.
> Esses conceitos só importam para o React internamente, porque ele pode fazer atualizações em lote em vários eventos contínuos, mas ainda atualizaria a tela para cada evento discreto em uma linha. No entanto, como um usuário React, você provavelmente não precisará pensar sobre isso.

## ✨ Promise Tick (Tick de Promessas)

Uma promessa é um objeto que representa algo que estará disponível no futuro. Você pode pensar em uma promessa como algo semelhante a um cronômetro que você pode obter em algumas lanchonetes de fast food. Quando o cronômetro tocar, você atende seu pedido.

```jsx
let comida = await cozinhar()
comer(comida)
```

Quando você vê o código usando `await`, é importante entender que sua função não executa de uma vez. Em vez disso, cada um espera “dividir” sua função. Há uma parte que é executada antes (`cozinhar()` lhe dá uma promessa). Então você diz para retomar quando a comida estiver pronta. Outro código poderá ser executado nesse meio tempo. Finalmente, quando a comida estiver pronta, você retornará a esta função, inicializará a variável `comida` e comê-la-á.

Um “tique de promessa” refere-se ao momento em que o JavaScript retoma sua função que estava esperando por uma promessa. É chamado de “tique” porque é como um cronômetro. Quando o resultado está pronto, o Promises não retoma suas funções imediatamente. Em vez disso, eles fazem isso * o mais rápido possível * depois que o código atualmente em execução termina de ser executado. Você pode pensar nisso como o dono de um restaurante esperando para ligar para o seu "cronômetro" até que suas mãos estejam livres, e não no momento em que terminam de fazer a refeição.

## ✨ Flush Sync

`flushSync` é o nome do método que permite controlar quando o React atualiza a tela (" limpa "as atualizações de estado). Em particular, permite que você diga ao React para fazer isso * agora * ("de forma síncrona"):

```jsx
flushSync(() => {
  setState(algo);
});
// Por esta linha, a tela foi atualizada
```

"Flush synchronously" = "Atualizar a tela agora"

Normalmente, você não deve precisar do `flushSync`. Você só o usaria se precisar solucionar algum problema em que o React atualiza a tela mais tarde do que o necessário. É muito raramente usado na prática.

## ✨ Code Splitting (Divisão de Código)

[Uma postagem nesse blog tenta explicar](https://laurieontech.com/posts/code-splitting/)

O resumo são os limites de bagagem das companhias aéreas. Você só pode ter 20kg em uma bolsa. Mas você precisa trazer 100kg em mercadorias. Então você o divide em cinco malas para que cada uma delas possa voar ao mesmo tempo, mas ainda estar abaixo do limite.

## ✨ Debouncing e Throttling (Desbloqueio e Aceleração)

[@gaearon](https://github.com/gaearon)

### O que é debouncing e throttling?

Debouncing e throttling são técnicas muito comuns para lidar com coisas que acontecem "com muita frequência". Imagine que você está conversando com um amigo e ele está contando uma história, mas tem dificuldade para fazer uma pausa enquanto fala. Digamos que você queira escuta-lo e, ao mesmo tempo, responder ao que ele têm a dizer, quando possível. (Eu sei que isso pode ser um pouco artificial, mas tenha paciência!)

Digamos que vocês nunca possam falar ao mesmo tempo.

Você tem algumas estratégias:

### Synchronous (Síncrono)

Você pode responder a cada frase no momento em que ela terminar:

![](https://user-images.githubusercontent.com/810438/121427718-f3457b00-c96c-11eb-9515-0ab2974414c9.png)

Isso pode funcionar bem se suas respostas forem curtas. Mas se suas respostas forem mais longas, isso pode tornar muito difícil para ele terminar a história. Portanto, essa estratégia não é ótima.

### Debounce

Você pode esperar que eles parem de falar. Por exemplo, se ele pausar por tempo suficiente, você pode começar a responder:

![](https://user-images.githubusercontent.com/810438/121427785-05271e00-c96d-11eb-88f2-b0c80221fe06.png)

Essa estratégia funciona bem se seu amigo estiver fazendo pausas ocasionalmente. No entanto, se ele está falando por alguns minutos sem pausa, isso não permite que você responda:

![](https://user-images.githubusercontent.com/810438/121428868-348a5a80-c96e-11eb-8b7d-c0b7bb45971c.png)

### **Throttled (**Estrangulado)

Você pode decidir responder *no máximo* uma vez por minuto. Aqui, você mantém uma contagem de quanto tempo você não falou. Depois de não falar por um minuto, você insere sua resposta logo após a próxima frase do amigo:

![](https://user-images.githubusercontent.com/810438/121428472-c2b21100-c96d-11eb-962c-ea1c190815cd.png)

Essa estratégia é útil se o seu amigo deseja que você responda enquanto ele fala, mas ele nunca cria pausas para você fazer isso. Mas não é ótimo se eles estão criando pausas, mas você ainda está esperando sem motivo:

![](https://user-images.githubusercontent.com/810438/121429118-77e4c900-c96e-11eb-9ac1-88f3816487e6.png)

### Como isso se relaciona com o React?

As "frases" de amigos são eventos como cliques em botões ou digitação no teclado. Suas "respostas" estão atualizando a tela.

Você usa o debouncing ou throttling quando o usuário está fazendo algo muito rápido (por exemplo, digitar) e atualizar a tela em resposta a * cada * evento individual é muito lento. Portanto, ou você espera que o usuário pare de digitar (debouncing) ou atualiza a tela de vez em quando, como uma vez por segundo (throttling).

### E o React 18?

O interessante é que `[<startTransition />](https://github.com/reactwg/react-18/discussions/41)` torna *ambas* as estratégias desnecessárias para muitos casos. Vamos adicionar mais uma estratégia que você não tinha antes do React 18:

### **Cooperativo**

Você começa a responder *imediatamente* no momento em que seu amigo termina uma frase, sem esperar. Mas você concordou com seu amigo que eles podem interrompê-lo se quiserem continuar. Nesse caso, você abandonará sua tentativa de responder. Logo após a próxima frase, você tentará novamente. E assim por diante:

![](https://user-images.githubusercontent.com/810438/121429595-ffcad300-c96e-11eb-81dc-29900d9b4cb9.png)

(Você *também* concorda em interrompê-los * se tiver passado tempo suficiente e você ainda não tiver respondido. Isso evita o caso de você não conseguir falar por muito tempo.)

Observe como nesta estratégia  *você não está perdendo tempo* e *também não impede seu amigo de continuar*. Isso lhe dá o melhor de todas as três abordagens anteriores. (Quem diria que ser cooperativo ajuda!)

Como exemplo de código, isso significa que o React começa a renderizar *imediatamente* depois que o usuário digita. Não há necessidade de atrasar a execução (como o debouncing faz). Se o usuário digitar novamente, o React simplesmente abandonará o trabalho e começará novamente. Se a renderização foi rápida o suficiente para caber na pausa entre os eventos, então você não perdeu tempo (esperando desnecessariamente). Você também não está perdendo tempo terminando uma renderização que não é mais necessária.

- *Você pode ver uma demonstração da diferença [aqui](https://react-beta-seven.vercel.app/). Tente digitar na entrada. Quanto mais você digita, mais complexo desenhamos o gráfico (artificialmente reduzido para mostrar o ponto). Veja o comportamento diferente entre as três estratégias. Como eles se comportam quando você continua digitando um texto mais longo na entrada?

Qual parece mais suave?

## ✨ Sub-Tree Basis (Base de Sub-Árvore)

[@gaearon](https://github.com/gaearon)

> Acredito que isso se refira a uma frase de [[# 4](https://github.com/reactwg/react-18/discussions/4)]
> Esses recursos são ativados em uma base de sub-árvore sem habilitar o StrictMode para todo o seu aplicativo.
> Esta é uma formulação confusa. "Em uma base de subárvore" aqui significa "para partes individuais da árvore em vez de todo o aplicativo". Aqui está como eu reformularia esta frase:
> Você pode começar a usar esses recursos em uma pequena parte da sua árvore sem habilitar o Modo estrito para todo o seu aplicativo.
> Eu editei isso. Obrigado por notar!

## ✨ State Transition (Estado de Transição)

[@gaearon](https://github.com/gaearon)

> Coloquialmente, uma “State Transition” é o mesmo que uma “mudança de estado”. Por exemplo, talvez você altere o estado `activeTab` de`"home"`para`"profile"`. Você poderia dizer que esta é uma “transição de estado” de Casa para Perfil.
> Normalmente, por “transições” as pessoas se referem a atualizações maiores. Como quando mais coisas acontecem. Talvez alguns dados precisem ser buscados, uma grande parte da tela seja alterada, talvez haja alguma animação.
> React 18 baseia-se nessa intuição e introduz um conceito explícito de “transições” [# 41](https://github.com/reactwg/react-18/discussions/41). Ao marcar uma atualização de estado como uma Transição, você dá uma dica ao React de que pode envolver muito trabalho. O React permite que o Transitions seja interrompido por atualizações mais urgentes, como digitar em uma entrada. No futuro, também estamos considerando a integração desse conceito com animações.

## ✨ Server Components (Componentes do Servidor)

Os componentes do servidor são um novo recurso experimental do React. Provavelmente não fará parte do React 18, mas seguirá em algum ponto depois. É altamente recomendável [assistir a palestra de introdução](https://reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html) que explica a ideia porque não há nada *exatamente* como ele, por isso é difícil compará-lo com as coisas existentes.

Veja como você pode pensar sobre isso. Você tem um servidor (onde seu banco de dados pode residir) e um cliente (por exemplo, um telefone ou um computador). O navegador é executado no cliente. Seu aplicativo React também é executado no cliente. Portanto, seus componentes:

```jsx
<Panel>
  <SearchField />
  <FriendList />
</Panel>
```

também é executado no cliente (no telefone ou computador do usuário).

Isso significa que:

1. Eles não funcionam até que a tag JavaScript `<script />` com eles termine de carregar.
2. Se eles precisam obter alguns dados do servidor (como a lista de amigos!), Você precisa escrever um código especial para isso.

Os componentes do servidor permitirão que você coloque * seus componentes * no servidor. Por exemplo, se os componentes `<Panel>` e `<FriendList>` não tiverem manipuladores de eventos (como clique) ou estado, não há razão para que eles devam ser executados no computador do usuário. Eles podem ser executados no servidor, assim como sua API!

Por que isso é melhor?

1. Agora, o único componente em sua tag `<script />` é `SearchField`. Não há necessidade de baixar o código para `Panel` ou`FriendList`.
2. Agora, o `FriendList` não precisa de código especial para" buscar "dados do servidor. Ele pode ler dados diretamente porque * já está no servidor *.

Portanto, os componentes do servidor são uma maneira de:

- Mantenha alguns componentes do React no servidor para que você não envie seus códigos para o computador (isso mantém seus pacotes menores e seu aplicativo mais rápido!)
- Simplifique a busca de dados (em vez de um componente "buscar" seus dados em efeitos, ele pode lê-los do servidor diretamente porque * está * no servidor)
- Pense no servidor e no cliente como uma única árvore, em vez de duas partes desconectadas que precisam "conversar" uma com a outra.

## ✨ Hooks e Hooks personalizados

[@shaundai](https://github.com/shaundai)

> Existem várias maneiras diferentes de escrever componentes, mas os componentes funcionais são fantásticos porque permitem que você faça cálculos complexos e carregue a lógica de uma maneira muito fácil. Se você quiser pegar um arquivo `.json` longo com dados e, em seguida, pegar esses dados e renderizá-los como uma tabela no site de alguém, por exemplo, você pode essencialmente fazer tudo isso com uma função. Os componentes da função são * perfeitos *, mas há um "pegadinho": eles não podem manter o estado.
> Antigamente (antes dos ganchos), se você quisesse usar o estado, teria que usar vários tipos de componentes (componentes de classe para qualquer coisa com componentes de estado e função para fazer algo com as informações) e era muito menos limpo.
> Digite HOOKS. Os ganchos permitem que você "se conecte" aos recursos do React, como métodos de estado e ciclo de vida, para que você possa ter um código bonito e elegante. Alguns ganchos vêm "fora da caixa" (para obter e atualizar o estado, você tem `useState`, ou para mudar as coisas usando métodos de ciclo de vida que você pode usar`useEffect`).

[@gaearon](https://github.com/gaearon)

> Eu diria que ambos os "function components" e "functional components" são bons coloquialmente. Mudamos de "functional" (funcional) para "function" (função) ao apresentar os Hooks porque estávamos preocupados que os puristas da programação funcional fariam um grande estardalhaço com os Hooks "não sendo funcionais" no sentido de programação funcional. (Embora eles meio que sejam - mas esse é um outro assunto.) Eu não fui criticar isso, exceto nos documentos oficiais. Vamos chamá-los de "componentes", já que os componentes de classe estão desaparecendo como um conceito especializado e mais legado.

## ✨Fast Refresh (Atualização Rápida)

[@meganesu](https://github.com/meganesu)

> Analisei isso em meu [repositório TIL](https://github.com/meganesu/TIL/blob/master/react/fast-refresh.md)
> Fast Refresh é um recurso do React que processa novamente os componentes imediatamente quando você faz alterações em seu código em um editor (como o VS Code). Com o Fast Refresh, seu navegador visualiza as atualizações sem perder o estado do componente, o que leva a uma melhor experiência do desenvolvedor. Fast Refresh é uma nova iteração no recarregamento a quente. O recarregamento a quente pretendia ter benefícios semelhantes, mas não funcionou bem com componentes funcionais e ganchos. Ele também não lidava bem com os erros, então você teria que reiniciar o aplicativo após cometer um erro de digitação. A atualização rápida melhora essas deficiências.

## ✨ App State, Component State, UI State (Estado do App, Estado do Componente e Estado da UI)

[@gaearon](https://github.com/gaearon)

> Seus componentes React transformam informações em IU. Existem dois tipos de informação.
> Algumas dessas informações nunca mudam durante uma sessão de usuário. Por exemplo, o layout do seu site, os ícones nos botões e a ordem das seções na página.

> Outras informações podem mudar em resposta à interação do usuário. Por exemplo, é um botão pairado ou não. Quantas vezes o usuário clicou em “adicionar”. O conteúdo do carrinho de compras ou uma mensagem que eles estavam digitando.
> As informações que mudam em resposta às interações do usuário são chamadas de “estado”.
> Qualquer informação tem vida útil. Está ligado a algo. Por exemplo, variáveis JS regulares são anexadas às chamadas de função. Eles só vivem enquanto você estiver nesta função. Quando você sai de uma função, as variáveis dessa chamada param de "existir". (Além de encerramentos, que ocorrem quando você aninha funções e que permitem que uma função aninhada "continue vendo" as variáveis da chamada de função externa.)

> No React, cada vez que um componente é renderizado, é uma chamada de função separada. Portanto, se você tentasse armazenar algumas informações como o conteúdo do carrinho de compras em uma variável regular, ela simplesmente desapareceria na próxima renderização. É por isso que o React tem um conceito de “variáveis de estado” que você define com `useState`. Eles estão associados a um local na árvore - como uma entrada de texto específica ou um carrinho de compras que você vê na tela - em vez de uma chamada de função.
> Onde você declara uma variável de estado é importante. Veja como pensar sobre isso. Se você renderizar algo em dois lugares na tela, gostaria que eles fossem sincronizados ou não?

> Por exemplo, renderizar dois campos de comentário não significa que você deseja que um seja atualizado ao digitar no outro. Portanto, faz sentido colocar o estado * dentro * do campo de comentário. Cada cópia é independente.
> Mas talvez você esteja renderizando um comentário já enviado em dois lugares. Se você fizer uma edição em um deles e * salvá-lo *, será razoável esperar que ambos precisem refletir sua edição. Portanto, as informações sobre uma lista de comentários precisam estar em algum lugar * fora * do seu componente - talvez, em seu pai compartilhado ou em algum tipo de cache.
> Isso é o que as pessoas querem dizer com diferentes tipos de estado. Estes não são termos técnicos. Mas “estado da IU” geralmente significa algum estado específico para um widget de IU concreto. Como o texto da entrada de texto. Considerando que "estado do aplicativo" geralmente significa algumas informações que são compartilhadas entre muitos componentes e precisam estar em sincronia. Portanto, geralmente é colocado fora deles.
