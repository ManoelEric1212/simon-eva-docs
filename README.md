# Documentação do Projeto Simon EVA - Front-end Modularizado

Esta documentação detalha a arquitetura front-end do projeto Simon-EVA, que utiliza Module Federation para criar uma abordagem modular e escalável. Essa estratégia permite o compartilhamento de componentes e funcionalidades entre diferentes aplicações, facilitando a reutilização de código e a gestão de projetos complexos.

## O que é o Module Federation?

Module Federation é um recurso do Webpack (e agora disponível para outras ferramentas de build como Vite, através de plugins) que permite que diferentes aplicações JavaScript compartilhem e consumam módulos de forma dinâmica em tempo de execução. Isso é fundamental para a arquitetura de micro-frontends, onde várias equipes podem desenvolver e implantar partes independentes de uma aplicação web maior.

Seus principais benefícios incluem:
- Reutilização de Código: Componentes, bibliotecas e lógicas de negócio podem ser expostos por uma aplicação (o "remote") e consumidos por outra (o "host").
- Implantação Independente: As micro-frontends podem ser implantadas separadamente, reduzindo a dependência entre as equipes e o tempo de lançamento.
- Compartilhamento de Dependências: Permite o compartilhamento de bibliotecas comuns (como React, ReactDOM) para evitar duplicação e reduzir o tamanho dos bundles.
- Escalabilidade: Facilita a expansão e manutenção de grandes aplicações, dividindo-as em partes menores e gerenciáveis.
