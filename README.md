# Documentação do Projeto Simon EVA - Front-end Modularizado

Esta documentação detalha a arquitetura front-end do projeto Simon-EVA, que utiliza Module Federation para criar uma abordagem modular e escalável. Essa estratégia permite o compartilhamento de componentes e funcionalidades entre diferentes aplicações, facilitando a reutilização de código e a gestão de projetos complexos.

## O que é o Module Federation?

Module Federation é um recurso do Webpack (e agora disponível para outras ferramentas de build como Vite, através de plugins) que permite que diferentes aplicações JavaScript compartilhem e consumam módulos de forma dinâmica em tempo de execução. Isso é fundamental para a arquitetura de micro-frontends, onde várias equipes podem desenvolver e implantar partes independentes de uma aplicação web maior.

Seus principais benefícios incluem:
- Reutilização de Código: Componentes, bibliotecas e lógicas de negócio podem ser expostos por uma aplicação (o "remote") e consumidos por outra (o "host").
- Implantação Independente: As micro-frontends podem ser implantadas separadamente, reduzindo a dependência entre as equipes e o tempo de lançamento.
- Compartilhamento de Dependências: Permite o compartilhamento de bibliotecas comuns (como React, ReactDOM) para evitar duplicação e reduzir o tamanho dos bundles.
- Escalabilidade: Facilita a expansão e manutenção de grandes aplicações, dividindo-as em partes menores e gerenciáveis.

## Diagrama exemplo para module federation

![Image](https://github.com/user-attachments/assets/97a3c489-bc5f-4544-b39c-0b3a9d679d8e)


## Como ele é utilizado no Vite com o ```js @originjs/vite-plugin-federation ```?


O ```js @originjs/vite-plugin-federation ``` é um plugin que traz a funcionalidade do Module Federation para o ecossistema Vite. Ele se integra ao processo de build do Vite, permitindo que você configure quais módulos serão expostos por uma aplicação e quais módulos de outras aplicações serão consumidos.

## Configuração Básica no vite.config.ts

A configuração central para o Module Federation no Vite é feita dentro do array plugins do arquivo ```vite.config.ts```:

```js

import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import federation from "@originjs/vite-plugin-federation"; // Importa o plugin

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: "nomeDaAplicacao", // Nome único da aplicação (ex: layoutSimon)
      filename: "remoteEntry.js", // Nome do arquivo que conterá o manifesto dos módulos expostos
      exposes: {
        // Módulos que esta aplicação irá expor para outras consumirem
        "./ComponenteA": "./src/components/ComponenteA.tsx",
      },
      remotes: {
        // Módulos que esta aplicação irá consumir de outras
        outraAplicacao: "http://localhost:porta/assets/remoteEntry.js",
      },
      shared: ["react", "react-dom"], // Dependências compartilhadas para evitar duplicação
    }),
  ],
  build: {
    target: "esnext",
    minify: false, // Pode ser true em produção
    cssCodeSplit: false, // Pode ser true se preferir CSS separado
  },
});


```

- **name**: Um nome único para sua aplicação no contexto do Module Federation.

- **filename**: O nome do arquivo JavaScript gerado que atua como um manifesto, listando os módulos que esta aplicação expõe.

- **exposes**: Um objeto onde as chaves são os "apelidos" dos módulos que você está expondo (como serão importados por outras aplicações) e os valores são os caminhos relativos para os arquivos desses módulos dentro do seu projeto.

- **remotes**: Um objeto onde as chaves são os "apelidos" das aplicações remotas que você está consumindo e os valores são as URLs para os seus respectivos arquivos remoteEntry.js.

- **shared**: Uma lista de dependências que o Module Federation tentará compartilhar entre as aplicações. Se uma dependência já estiver carregada e sua versão for compatível, ela não será carregada novamente, otimizando o tamanho do bundle.


## O uso de pacote npm para tipagens


Em um ecossistema modular com Module Federation, onde componentes são compartilhados entre diferentes repositórios, a gestão de tipagens torna-se crucial para garantir a segurança de tipo e a experiência de desenvolvimento. Criar um pacote npm dedicado para as tipagens oferece várias vantagens:

- **Centralização**: Todas as interfaces e tipos relacionados aos módulos compartilhados são definidos em um único local.

- **Reutilização**: Ambos os projetos (o que expõe e o que consome os módulos) podem instalar e utilizar este pacote, garantindo que estejam usando as mesmas definições de tipo.

- **Autocompletar e Validação**: Editores de código (como VS Code) podem fornecer autocompletar e validação de tipo para os componentes remotos, melhorando a produtividade e reduzindo erros.

- **Desacoplamento**: Separa a definição de tipos da implementação dos componentes, tornando a arquitetura mais limpa.

## Proposta de arquitetura para os projetos Simon, no que se diz respeito ao front

Para os projetos Simon, a partir do ques está sendo construído no projeto do EVA, os próximos projetos poderão consuir módulos existentes em outros servidores remotos.
