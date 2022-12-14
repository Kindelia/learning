# Capítulo 5
### 1 Polimorfismo
Neste capítulo, continuamos nosso desenvolvimento de conceitos básicos de programação. Nossos novos princípios essenciais são o polimorfismo (funções de abstração sobre o
tipos de dados que eles manipulam) e funções de ordem superior (funções de tratamento como dados). Começamos com o polimorfismo.

- 1.1. Listas Polimórficas. 

Nos últimos dois capítulos, trabalhamos com listas polimórficas, você pode só não ter percebido. Obviamente, programas interessantes também precisam ser capazes de
manipular listas com elementos de outros tipos – listas de strings, listas de booleanos,
listas de listas, etc. Poderíamos apenas definir um novo tipo de dados indutivo para cada um deles,
por exemplo...

```rust
ListBool : Type
ListBool.nil : (List Bool)
ListBool.cons (head: Bool) (tail: ListBool Bool) : (ListBool Bool)
```

.. mas isso rapidamente se tornaria tedioso, em parte porque temos que compensar
nomes de construtor diferentes para cada tipo de dados, mas principalmente porque também
precisamos definir novas versões de todas as nossas funções de manipulação de listas (length, rev, etc.)
para cada nova definição de tipo de dados.

Para evitar toda essa repetição, o *Kind* suporta definições de tipos indutivos polimórficos.
Por exemplo, aqui está um tipo de dados de lista polimórfica e que já vimos no capítulo anterior:

```rust
List <a: Type> : Type
List.nil <a> : (List a)
List.cons <a> (head: a) (tail: List a) : (List a)
```

Esse tipo já exite no Kind e podemos perceber que ele é idententico ao ListBool, mas com um tipo `a`, esse *tipo* recebe qualquer outro tipo, seja *Nat*, *Bool*, *Maybe* e etc.
Não precisamos criar um tipo de lista para cada um dos tipos de dados, podemos usar esse que adota todas as formas existentes.

Que tipo de coisa é a própria Lista? Uma boa maneira de pensar sobre isso é que List é
uma função de Tipos para defi   nições indutivas; ou, em outras palavras, List é
uma função de Tipos para Tipos. Para qualquer tipo específico x, o tipo List x é um
conjunto indutivamente definido de listas cujos elementos são do tipo x.

Com esta definição, quando usamos os construtores Nil e Cons para construir listas,
precisamos dizer ao Kind o tipo dos elementos nas listas que estamos construindo - isto
é, que Nil e Cons agora são construtores polimórficos. Observe os tipos de
construtores:

```rust
Pair (a: Type) (b: Type) : Type
Pair.new <a> <b> (fst: a) (snd: b) : (Pair a b)
```

Nosso tipo *Pair* recebe outros dois tipos, o `a` e o `b` e retorna um par dos dois tipos. Não foi necessário definir se o par era de números naturais,
booleanos, listas, bits ou outros pares, nós deixamos a função apta a tratar todos os pares possíveis e isso é graças ao *polimorfismo*.

Agora podemos voltar e fazer versões polimórficas de todas as listas de processamento
funções que escrevemos antes. Aqui está a repetição, por exemplo:

```rust
List.repeat <a: Type> (xs: a) (count: Nat)  : List a
List.repeat xs Nat.zero                     = List.nil
List.repeat xs (Nat.succ count)             = List.cons xs (List.repeat xs count)
 ```

Tal como acontece com Nil e Contras, podemos usar repeat aplicando-o primeiro a um tipo e depois a
seu argumento de lista:

```rust
Test_repeat1 : (Equal (Repeat 4 (U60.to_nat 2))(List.cons 4 (List.cons 4 List.nil)))
Test_repeat1 = Equal.refl
```

Para usar repeat para construir outros tipos de listas, simplesmente instanciamos com um parâmetro de tipo apropriado:

```rust
Test_repeat2 : (Equal (Repeat Bool.false (U60.to_nat 1)) (List.cons Bool.false List.nil))
Test_repeat2 = Equal.refl
```

1.1.2 Inferência de anotação de tipo.

Vamos escrever a definição do `repeat` novamente, mas dessa vez omitindo o tipo, mas atenção, essa não é uma boa prática usar o `hole`, servirá apenas para compreender o poder do Kind e como ele pode ajudar o usuário a encontrar o que deseja.

```rust
List.repeat (val: _) (count: Nat)   : List _
List.repeat val Nat.zero            = ?
```

Ao rodar o *Type Check* o terminal nos retorna:
```bash
Inspection.
- Goal: (List _)
Kind.Context:
- val : _
On 'main.kind2':
   3 | List.repeat val Nat.zero          =  ? 
```

Para o caso do contador ser zero, que é o nosso ponto de parada, nós precisamos retornar uma lista do tipo não definido. 
Como fizemos quando nosso tipo era definido, estamos criando uma lista que não repete o termo nenhuma vez, retornamos um *List.nil*, depois nós verificamos para o caso de uma lista que repetirá o valor *count* vezes, para isso nós usaremos a recursão por meio do `Nat.succ pred`, isto é, o nosso *count* é igual ao sucessor do predecessor dele. 

```rust
List.repeat val (Nat.succ pred)          = ?
```

E o o *Type Check* nos retorna:

```bash
Inspection.
- Goal: (List _)
Kind.Context:
- val   : _
- pred : Nat
On 'main.kind2':
   4 | List.repeat val (Nat.succ pred)   = ? 
```

Agora basta construir a lista com o valor e chamar a função para o predecessor de count, assim construindo a lista até que chegue a zero.

```rust
List.repeat (val: _) (count: Nat)    : List _
List.repeat val Nat.zero             = List.nil
List.repeat val (Nat.succ pred)      = List.cons val (List.repeat val pred)
```

Podemos perceber que, apesar de não definir o tipo de *val*, o *Kind* é poderoso para descobrir o tipo que é nosso val quando usamos o *hole* `_`. Embora seja possível e possa até facilitar construir uma aplicação inteira usando essa notação, não é uma boa prática, já que, a depender do caso, pode ser inferido um tipo diferente do que o desejado. É interessante sempre definir o tipo do nosso elemento, mesmo que seja um tipo polimórico. 

No primeiro caso, quando definimos o tipo `a`, já abarcamos todos os tipos possíveis, não sendo necessário o uso do hole e essa é a mágica do polimorfismo, ele nos permite usar uma mesma função para diversos tipos diferentes.

Para usar uma função polimórfica, nós precisamos passar um ou mais tipos em adição aos outros argumentos. Por exemplo, no caso do *repeat*, nós passamos o tipo `<a: Type>` e que cada elemento da nossa lista é desse tipo. Fizemos o mesmo com o tipo *Pair*, que recebia como argumento dois tipos *a* e *b*. 

Agora fica muito mais fácil compreender os exemplos que usamos no capítulo anterior, quando apresentamos funções como a de *length* e *append*:

```rust
List.length <a> (xs: List a) : Nat
List.length a (List.nil t)            = Nat.zero
List.length a (List.cons t head tail) = (Nat.succ (List.length a tail))

List.append <a: Type> (xs: List a) (x: a) : List a
List.append a (List.nil xs.a)            x = List.pure x
List.append a (List.cons xs.a xs.h xs.t) x = List.cons xs.h (List.append xs.t x)
```

Perceba, há duas notações, uma onde apenas usamos `<a>` e outra onde usamos `<a: Type>`, podemos usar qualquer uma delas, o *Kind* é capaz de compreender as duas formas, será de escolha do desenvolvedor qual ele usará e da complexidade do que será desenvolvido, uma vez que, em códigos muito complexos, talvez seja interessante deixar explícito a outros programadores o que é cada coisa.

Agora é a hora de implementar nossas funçõe com o tipo implícito, usando o `hole` e `sugar syntax`:

```rust
App_implicito (xs: List _) (x: _)       : List _
App_implicito [] x                      = []
App_implicito (List.cons xs.h xs.t) x   = List.cons xs.h (App_implicito xs.t x)
```

Aqui nós aprendemos mais uma coisa, o *sugar syntax* para uma lista vazia e que é apenas `[] `e isso poderia nos induzir a escrever o nosso *List.cons* com `[xs.h, xs.t]`, mas isso está errado, uma vez que o *head* é um elemento de um tipo e o *tail* é uma lista de elementos desse tipo e, dessa forma, estariamos fazendo uma lista de lista. Ao usar o *sugar syntax* erraado, o que o *Kind* esperaria seria:
```bash
- Expected: (List t3_)
- Detected: (List (List t3_))
```

Perceba, ele espera uma lista e recebe uma lista de lista, pois nós declaramos o nosso *tail* como sendo ualgo do tipo t3_ (ignore a escrita do tipo, isso ocorre porque não definimos o tipo e o Kind cria um temporário apenas para retorno da mensagem), o que faz com que ela deixe de ser uma lista, causando essa incongruência no retorno. 

Portanto, é sempre importante saber exatamente o que está sendo feito, ainda mais quando usamos *sugar syntax*, ela serve pra facilitar a nossa vida, mas pode causar alguns problemas quando usada de forma indevida e isso serve igualmente para o *hole* e tipos polimórficos nos auxiliam a excrever um programa mais seguro e, ao mesmo tempo, capaz de servir para inúmeros casos. 

Outra função que podemos reescrever é a de reverse:

```rust
Rev <a> (xs: List a)        : List a
Rev []                      = []
Rev (List.cons xs.h xs.t)   = (List.concat (Rev xs.t)[xs.h])
Length <a> (xs: List a)      : Nat
Length []                    = Nat.zero
Length (List.cons xs.h xs.t) = Nat.succ (Length xs.t)
```

Feito isso, basta apenas provar que nossas funções são verdadeiras:

```rust
Test_rev1 : Equal (Rev [1, 2, 3]) [3, 2, 1]
Test_rev1 = Equal.refl

Test_rev2 : Equal (Rev (Bool.true)) (List.cons Bool.true List.nil)
Test_rev2 = Equal.refl

Test_length1 : Equal (Length [1, 2, 3]) (U60.to_nat 3)
Test_length1 = Equal.refl
```

1.1.3

Aqui estão alguns exercícios simples, assim como os do capítulo Listas, para praticar com polimorfismo. Complete as provas abaixo.

```rust
App_nil_r <a> (xs: List a) : Equal (List.concat xs List.nil) xs
App_nil_r xs = ?

App_assoc <a> (xs: List a) (ys: List a) (zs: List a) : Equal (List.concat xs (List.concat ys zs)) (List.concat (List.concat xs ys) zs)
App_assoc xs ys zs = ?

App_length <a> (xs: List a) (ys: List a) : Equal (List.length (List.concat xs ys)) (Nat.add (List.length xs) (List.length ys))
App_length xs ys = ?
```

1.1.4

Aqui estão alguns um pouco mais interessantes...

```rust
Rev_app_distr <a> (xs: List a) (ys: List a) : Equal (Rev (List.concat xs ys)) (List.concat (Rev ys) (Rev xs))
Rev_app_distr xs ys = ?

Rev_involutive <a> (xs: List a) : Equal (Rev (Rev xs)) xs
Rev_involutive xs = ?
```

1.2 Pares polimórficos 

Seguindo o mesmo padrão, a definição de tipo
que demos no último capítulo para pares de números podem ser generalizados para polimórficos
pares:

```rust
   type Pair <a: Type> <b: Type> {
      new (fst: a) (snd: b)
   }  
```

Essa é exatamente a primeira definição de pares que vimos no capítulo anterior e, agora, podemos compreender perfeitamente o que são os tipos `a` e `b` na definição do tipo *Pair*.

Nós podemos refazer as funções de *Pares*, mas agora para tipos polimórficos:

```rust
Pair.fst <a> <b> (pair: Pair a b) : a
Pair.fst (Pair.new fst snd) = fst

Pair.snd <a> <b> (pair: Pair a b) : b
Pair.snd (Pair.new fst snd) = snd
```

A seguinte função recebe duas listas e combina elas numa lista de pares. Nas linguagens funcionais, isso é comumente chamada de *Zip*. 

```rust
Zip <a> <b> (xs: List a) (ys: List b)           : (List (Pair a b))
Zip List.nil ys                                 = List.nil
Zip xs List.nil                                 = List.nil
Zip (List.cons xs.h xs.t) (List.cons ys.h ys.t) = (List.cons (Pair.new xs.h xs.t) (Zip xs.t ys.t))
```

Exercício 1.2.1
Sem rodar o programa, tente responder a seguinte pergunta:
- O que a combinação de [1, 2] e [Bool.true, Bool.false, Bool.false, Bool.true] retornará? 
 
Agora rode o código e veja se acertou. 

Exercício 1.2.2
A função *Split* é o inverso da *Zip*, ela recebe uma lista de pares e retorna um par de listas. Em muitas linguagens funcionais ela é chamada de *Unzip*. 

Preencha a definição de divisão abaixo. Certifique-se de que ela passe no teste unitário fornecido.

```rust
Split <a> <b> (xs: List (Pair a b)) : Pair (List a) (List b)
Split xs = ?

Test_split : Equal (Split [(Pair.new 1 Bool.false), (Pair.new 2 Bool.false)]) (Pair.new ([1, 2]) ([Bool.false, Bool.false]))
Test_split = ?
```

Exercício 1.2.3 
No capítulo anterior, nós vimos que as funções *List.head* nos retornava um tipo *Maybe*, mas vimos apenas para o tipo *Nat*, agora é a hora de tornar nossa função polimórfica. 

```rust
Maybe <a: Type> : Type
Maybe.none <a> : (Maybe a)
Maybe.some <a> (value: a) : (Maybe a)
```

Dessa forma, podemos escrever a função do *enésimo* erro para ele ser usado com todos os tipos de listas:

```rust
Nth_error <a> (n: Nat)(xs: List a)                    : Maybe a
Nth_error n List.nil                                = Maybe.none
Nth_error Nat.zero xs                               = List.head xs
Nth_error (Nat.succ n) (List.cons xs.head xs.tail)  =
  let ind = Nth_error n xs.tail
  Bool.if (Nat.equal (Nat.succ n) Nat.zero) (Maybe.some(xs.head)) (ind)


Test_nth_error1 : Equal (Nth_error Nat.zero [4, 5, 5, 7]) (Maybe.some 4)
Test_nth_error1 = Equal.refl

Test_nth_error2 : Equal (Nth_error (Nat.succ (Nat.succ Nat.zero)) [Bool.true]) Maybe.none
Test_nth_error2 = Equal.refl

Test_nth_error3 : Equal (Nth_error (Nat.succ Nat.zero) [[1], [2]]) (Maybe.some [2])
Test_nth_error3 = Equal.refl

```

1.2.4 
Complete a definição de
uma versão polimórfica da função hd_error do último capítulo. Certifique-se de que ele passe nos testes de unitários abaixo.

```rust
Hd_error <a> (xs: Lista a) : Maybe a
hd_error xs = ?

Test_hd_error1 : Equal (Hd_error [1, 2]) (Maybe.some 1)
Test_hd_error1 = ?

Test_hd_error2 : Equal (Hd_error [[1], [2]]) (Maybe.some [1])
Test_hd_error2 = ?
```

=== === === === === === continuar daqui === === === === === === 

### Funções como dados

Como muitas outras linguagens de programação modernas – incluindo todas as linguagens funcionais (ML, Haskell, Scheme, Scala, Clojure etc.) – o Kind trata as funções como cidadãos de primeira classe, permitindo que sejam passadas como argumentos para outras funções,
retornados como resultados, armazenados em estruturas de dados, etc.

#### 2.1 Funções de Alta Ordem (Higher-Order Functions.)

Funções que manipulam outras funções são frequentemente chamadas de funções de alta ordem (ou ainda de "ordem superior"). Aqui está um exemplo simples:

```rust
Doit3times <x> (f: x -> x) (n: x) : x
Doit3times f x = (f (f (f x)))

Test_doit3times1 : Equal (Doit3times (x => Nat.sub x (U60.to_nat 2))) (U60.to_nat 3)
Test_doit3times1 = Equal.refl

Test_doit3times2 : Equal (Doit3times (x => Bool.not x) Bool.true) (Bool.false)
Test_doit3times2 = Equal.refl
```

#### 2.2 Filtro
Aqui está uma função de alta ordem mais útil, pegando uma lista de xs e um predicado em x (uma função de x para Bool) e “filtrando” a lista, retornando uma nova lista contendo apenas aqueles elementos para os quais o predicado retorna True.

```rust
Filter <x> (test: x -> Bool) (xs: List x) : List x
Filter test List.nil                      = List.nil
Filter test (List.cons xs.h xs.t)         =
   Bool.if (text xs.h) (List.cons xs.h (Filter test xs.t)) (Filter test xs.t)
```

Por exemplo, se aplicarmos o filtro de "é par" numa lista de números, ela nos retornará uma outra lista apena com os números pares

```rust
Test_filter1 : Equal (Filter (x => Nat.is_even x) (List.to_nat [1, 2, 3, 4, 5])) (List.to_nat [2, 4])  
Test_filter1 = Equal.refl

Length_is_one <x> (xs: List x)   : Bool
Length_is_one xs                 = Nat.equal (List.length xs) (Nat.succ Nat.zero)

Test_filter2 : Equal (Filter (x => Length_is_one x) ([[1], [1, 2], [3], [1, 2, 3], [21]])) ([[1], [3], [21]])
Test_filter2 = Equal.refl
```

Podemos usar filter para fornecer uma versão concisa da função countoddmembers do capítulo Listas.

```rust
CountOddMembers (xs: List Nat)   : Nat
CountOddMembers xs               = List.length (Filter (x => Nat.is_odd x) xs)

Test_CountOddMembers1 : Equal (CountOddMembers (List.to_nat ([1, 0, 3, 1, 4, 5]))) (U60.to_nat 4)
Test_CountOddMembers1 = Equal.refl

Test_CountOddMembers2 : Equal (CountOddMembers (List.to_nat ([0, 2, 4]))) (Nat.zero)
Test_CountOddMembers2 = Equal.refl

Test_CountOddMembers3 : Equal (CountOddMembers (List.nil)) (Nat.zero)
Test_CountOddMembers3 = Equal.refl
```


#### 2.3. Funções anônimas. 
É indiscutivelmente um pouco triste, no exemplo acima, ser forçado a definir a função *Length_is_one* e dar-lhe um nome apenas para poder passá-la como um argumento para filtrar, já que provavelmente nunca a usaremos novamente.
Além disso, este não é um exemplo isolado: ao usar funções de ordem superior, muitas vezes queremos passar como argumentos funções “únicas” que nunca mais usaremos; ter que dar um nome a cada uma dessas funções seria tedioso.
Felizmente, existe uma maneira melhor. Podemos construir uma função “on the fly” sem declará-la no nível superior ou dar-lhe um nome.


