---
title: Transientes
date: "2020-05-20T22:55:57.783Z"
description: "Essa é a descrição"
---

Este é um assunto sobre Design Declarativo Funcional.

Qual é o problema de uma função pura realizar mutações no seu estado interno desde que não seja exposto ao mundo externo?

### Funções Puras

Funções puras são funções que, para uma dada entrada, sempre vai produzir a mesma saída sem causar efeitos colaterais no sistema. O que é o sistema? O sistema é o contexto em que a função está sendo executada. Por exemplo:

```javascript
const soma = (a, b) => {
	const res = a + b
	console.log(`soma(${a}, ${b}) = ${res}`)
	return res
}
```

Embora `soma` seja uma função pura, ela causa um efeito colateral na saída padrão. Isso está longe do esperado para a função `soma`. É esperado que sejam somados dois valores e que o resultado seja retornado para o contexto que chamou esta função. Isso de fato acontece, porém mais coisas acontecem no contexto da função `soma`, por este motivo, esta função é considerada **impura**.

### Reflexo da pureza

Em linguagens estritamente funcionais, não existe o conceito de mutabilidade. Um pedaço de memória uma vez criado, será constante até que seja coletado pelo coletor de lixo, caso a linguagem tenha suporte a coleta de lixo, ou que o programe chegue ao fim.

Ou seja, abstrações como _loops_ não existem.

Porém nossos computadores não funcionam desta forma, nossos computadores convencionais modernos precisam de mutabilidade. Eles foram desenvolvidos usando conceitos de mutabilidade. A arquitetura que é usada hoje na construção dos computadores modernos é muito parecida com a arquitetura de Von Neumann.

Então como é possível que linguagens puramente funcionais existam hoje? Porquê essas linguagens importam?

> Por tras de uma excelente linguagem, sempre existe um excelente compilador.

Uma linguagem de programação realmente poderosa é nada se o compilador não for igualmente poderoso. Porém linguagens muito mal desenhadas podem ser muito poderosas se munidas de um ótimo compilador. Toda a suíte do V8 do Google Chrome está aí para provar o que estou dizendo.

Linguagens puramente funcionais possuem otimizações a nível de geração de código de máquina. Por exemplo, em Haskell, o código abaixo é tão bem otimizado pelo compilador que, no final do processo, o código gerado é tão eficiente quando o código gerado pelo GCC ou Clang:

```haskell
fib :: Int -> Int
fib 0 _ _ = 0
fib 1 _ _ = 1
fib x a b = fib (x-1) b (a+b)
```

O GHC transforma a chamada pura, recursiva, em um procedimento impuro que usa mutabilidade. O ponto que eu quero chegar é: qual é o problema se em algum momento um compilador ou uma biblioteca usar mutabilidade para melhoria de performance e esconder esses truques do restante do código?

Rich Hickey já fez essa provocação muito tempo atrás. O próprio site do Clojure faz a seguinte provocação:

> If a tree falls in the woods, does it make a sound?

> If a pure function mutates some local data in order to produce an immutable return value, is that ok?

Se você, programador JavaScript, por algum motivo precisar buscar um dado em um array, você escreveria uma função recursiva?

```javascript
const search = (value, arr) => {
  if (arr.length === 0) return false;

  const [head, ...tail] = arr;
  if (head === value) return true;

  return search(value, tail);
}
```

O código acima seria a sua primeira opção?

Uma coisa precisamos entender: recursividade é ótima, conseguimos escrever funções extremamente consisas e claras. Conseguimos usar metodos de otimização muito bons, como programação dinâmica. Porém funções recursivas precisam de uma infraestrutura preparada tanto para suportar chamadas recursivas, quanto para otimizar chamadas recursivas. JavaScript não é esse tipo de linguagem.

Se você é um profissional decente, você sabe que deve escrever código limpo. Então porquê não isolar um código que usa mutabilidade dentro de uma função _"pura"_?

### Transientes

Se você chegou até aqui, já deve ter entendido onde eu quero chegar, talvez já tenha entendido o que valores transientes são.

Mas caso não, veja essas duas funções:

```javascript
const range1 = (start, end) =>
  start === end
  ? []
  : [start, ...range(start + 1, end)] 
```

E

```javascript
const range2 = (start, end) => {
  let arr = [];
  for (let i = start; i < end; i++) arr.push(i);
  return arr;
}
```

Qual versão é pura? Qual é a versão mais eficiente?

Usar transientes não é uma questão de pureza, é uma questão de eficiencia.

Ah, a propósito, transientes no contexto de programação funcional é o ato de esconder mutabilidade atrás de uma interface pura. 