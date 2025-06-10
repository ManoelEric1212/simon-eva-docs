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

Para os projetos Simon, a partir do ques está sendo construído no projeto do EVA, os próximos projetos poderão consumir módulos existentes em outros servidores remotos. De início o processo é criar um repositório para acumular os módulos a compartilhar e os demais projetos deverão consumir a partir desse módulo.


![Image](https://github.com/user-attachments/assets/d798ff50-0d8c-4d70-b1bb-819696d5294f)

### !! Ponto de atenção !!

Para que o módulo possa consumir o recurso o exportado, o projeto que estás a exportar deve estar rodando seja localmente seja em um servidor.

## Organização da estrutura de projetos e repositórios existentes

Seu projeto Simon EVA está organizado em três repositórios distintos para otimizar a modularidade e o desenvolvimento colaborativo:

### Tipos (simon-eva-types): [Link para o repositório](http://10.1.5.95:7171/industrial/simon2.0/modulos/modulosfrontend/simon2.0types.git)

- **Propósito**: Este repositório contém as definições de tipo ```(TypeScript .d.ts files)```(TypeScript .d.ts files) para todos os módulos que serão compartilhados entre as aplicações. É publicado como um pacote npm (simon-eva-types).
- **Conteúdo**: Interfaces, tipos e declarações de módulo (declare module "nomeDoModulo/Componente").
- **Vantagem**: Garante a consistência e segurança de tipo em todo o ecossistema de micro-frontends.


### Módulo (simon-layout): [Link para o repositório](http://10.1.5.95:7171/industrial/simon2.0/modulos/modulosfrontend/simon2.0layout.git)

- **Propósito**: Atua como o "remote" (ou "produtor de módulos"). Ele é responsável por criar e expor componentes reutilizáveis, como Header e Footer, que são parte do layout comum das aplicações Simon.
- **Tecnologias**: React, Vite, Tailwind CSS, @originjs/vite-plugin-federation.
- **O que expõe**: Componentes de UI que podem ser consumidos por outras aplicações.

### Simon-EVA-Front (simon-eva-front): [Link para o repositório](http://10.1.5.95:7171/industrial/simon2.0/injecao-eva/frontsimon2.0eva)

- **Propósito**: Atua como o "host" (ou "consumidor de módulos"). Esta é a aplicação principal do Simon EVA que integra os módulos expostos pelo layout-simon e quaisquer outras funcionalidades específicas.
- **Tecnologias**: React, Vite, Tailwind CSS, @originjs/vite-plugin-federation.
- **O que consome**: Módulos de outras aplicações, como o Header e Footer do layout-simon entre outros módulos que virão a ser construídos.
- **Dependências**: Inclui simon-eva-types para ter acesso às tipagens dos módulos remotos.

Essa estrutura permite que o layout-simon seja desenvolvido e mantido de forma independente, enquanto simon-eva-front e os demais projetos que virão a existir, encontram-se em sua lógica de negócio, consumindo os elementos padronizados.

## Como está organizado no git

Para início de projeto, apenas dois módulos existem, sendo eles o módulo de tipagem e o segundo o módulo de layout, que irá exporar os principais componentes para iniciar um projeto simon, como o Header, Tarja, Modal, Footer etc... Porém, em um futuro cada outra funcionalidade irá se tornar um módulo, como o módulo de Amostra, PLM, Alertas etc...

![image](https://github.com/user-attachments/assets/d71f9dd2-2052-4f0d-912c-5ea583f85001)

Além disso, os projetos que irão consumir os módulos podem ser criados de forma distinta em outras pastas e repositórios.

![image](https://github.com/user-attachments/assets/65518d5b-b512-44ce-b903-999802497fb6)

## Estruturas de exportação e consumo com o Vite
### Exportação de Módulos (layout-simon)
O repositório layout-simon é configurado para expor seus componentes como módulos via Module Federation.

vite.config.ts do layout-simon

```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import federation from "@originjs/vite-plugin-federation";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
    federation({
      name: "layoutSimon", // Nome único para esta aplicação (remote)
      filename: "remoteEntry.js", // Arquivo manifesto dos módulos expostos
      exposes: {
        // Módulos que estão sendo expostos
        "./Footer": "./src/components/Footer/Footer.tsx",
        "./Header": "./src/components/Header/Header.tsx",
      },
      shared: ["react", "react-dom", "react-router-dom"], // Dependências compartilhadas
    }),
  ],
  build: {
    target: "esnext",
    minify: false, // Desabilitado para facilitar a depuração (pode ser true em produção)
    cssCodeSplit: false, // Garante que o CSS seja embutido no JS para facilitar o MFE
  },
});

```

- O name: "layoutSimon" define o identificador único deste "remote".

- O exposes lista os componentes (./Footer, ./Header) e seus respectivos caminhos que serão acessíveis por outras aplicações.

- shared garante que react, react-dom e react-router-dom sejam carregados uma única vez, otimizando o bundle.

### Código do Componente Footer.tsx (Exemplo de Módulo Exportado)
```js
import { Link, useLocation } from "react-router-dom";

export interface FooterItem {
  rota: string;
  img: string;
  alt: string;
  label: string;
}

interface FooterProps {
  items: (FooterItem | "|")[];
  footerClassName?: string;
  linkClassName?: string;
  imgClassName?: string;
  spanClassName?: string;
}

export default function Footer({
  items,
  footerClassName,
  linkClassName,
  imgClassName,
  spanClassName,
}: FooterProps) {
  const { pathname } = useLocation();

  return (
    <footer
      className={
        footerClassName ??
        "flex items-center justify-evenly h-full px-1 text-white text-[1.75rem]"
      }
    >
      {items.map((item, index) =>
        item === "|" ? (
          <div key={index} className="w-[1px] h-6 bg-gray-300 mx-2" />
        ) : (
          <Link
            key={item.rota}
            to={item.rota}
            className={`flex justify-center items-center rounded-md border-none flex-grow min-h-[8vh] w-[20%] mx-1 text-white gap-1 no-underline bg-slate-700 hover:bg-slate-600 ${
                pathname.includes(item.rota)
                  ? "bg-gray-300 text-black font-semibold"
                  : ""
              } ${linkClassName ?? ""}`}
          >
            <img
              src={item.img}
              alt={item.alt}
              className={
                imgClassName ??
                "bg-blue-200 h-7 min-w-[1.8em] rounded-full text-lg"
              }
            />
            <span
              className={
                spanClassName ?? "text-center leading-tight whitespace-pre-line"
              }
            >
              {item.label}
            </span>
          </Link>
        )
      )}
    </footer>
  );
}


```

Este é o código-fonte do componente Footer que é exposto pelo layout-simon.
### Consumo de Módulos (simon-eva-front)
O repositório simon-eva-front é o "host" que consome os módulos expostos pelo layout-simon.

vite.config.ts do simon-eva-front

```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import federation from "@originjs/vite-plugin-federation";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
    federation({
      name: "simon-eva-front", // Nome único para esta aplicação (host)
      remotes: {
        // Módulos que estão sendo consumidos
        layoutSimon: "http://localhost:3002/assets/remoteEntry.js", // URL do remoteEntry.js do layout-simon
      },
      shared: ["react", "react-dom", "react-router-dom"], // Dependências compartilhadas
    }),
  ],
  build: {
    target: "esnext",
  },
});

```

- O name: "simon-eva-front" define o identificador único deste "host".
- O remotes configura qual aplicação remota (layoutSimon) será consumida e onde encontrar seu arquivo remoteEntry.js. A URL http://localhost:3002/assets/remoteEntry.js aponta para o servidor onde o layout-simon está sendo executado ou servido.
- shared também é configurado aqui para garantir que as dependências sejam compartilhadas.

### Declaração de Tipos para Consumo:
Para que o TypeScript entenda os módulos remotos, você precisa referenciar o pacote de tipos. Esta linha geralmente é adicionada em um arquivo de declaração de tipos, como src/vite-env.d.ts ou um arquivo similar.

```js

/// <reference types="vite/client" />
/// <reference types="simon-eva-types" /> // Importante para as tipagens dos módulos remotos

```

### Uso do Componente no Projeto (App.tsx do simon-eva-front)

```js
import IconOEE from "./assets/oee4.png";
import IconAmostra from "./assets/amostra3.png";
import IconQualidade from "./assets/qualidade3.png";
import IconAnalise from "./assets/dashboard3.png";
import IconAlerta from "./assets/alerta.png";
import Logo from "./assets/simon20.png";
import Footer from "layoutSimon/Footer"; // Importa o Footer do módulo remoto
import Header from "layoutSimon/Header"; // Importa o Header do módulo remoto
import AppRoutes from "./service/routesPath";
import HomeIcon from "@mui/icons-material/Home";
import PowerSettingsNewIcon from "@mui/icons-material/PowerSettingsNew";

const footerItems = [
  { rota: "/OeeProduto", img: IconOEE, alt: "oeeIcon", label: "Produto" },
  { rota: "/Oeeturno", img: IconOEE, alt: "oeeIcon", label: "Turno" },
  { rota: "/Amostra", img: IconAmostra, alt: "amostraIcon", label: "Amostra" },
  { rota: "/Plm", img: IconAmostra, alt: "plmIcon", label: "PLM" },
  {
    rota: "/Qualidade",
    img: IconQualidade,
    alt: "qualidadeIcon",
    label: "Qualidade 2.0",
  },
  { rota: "/Analise", img: IconAnalise, alt: "analiseIcon", label: "Análise" },
  { rota: "/Alertas", img: IconAlerta, alt: "alertaIcon", label: "Alertas" },
];

export default function App() {
  return (
    <div className="flex flex-col min-h-screen bg-gray-900 text-white" id="app">
      <header className="bg-gray-800 p-4 border-b border-gray-700">
        <Header
          image={Logo}
          menuPath="/"
          version="1.0.3"
          horimetro="18h"
          inicioHorimetro="01/06/2024"
          buttons={[
            { icon: <HomeIcon />, path: "/" },
            { label: "Logout", onClick: () => alert("Saindo...") },
            {
              icon: <PowerSettingsNewIcon />,
              label: "Desligar",
              onClick: () => console.log("Desligando..."),
            },
          ]}
        />
      </header>
      <main className="flex-1 p-6">
        <AppRoutes />
      </main>
      <div className="pb-2 pt-2 ">
        <Footer
          items={footerItems}
          footerClassName="flex items-center justify-evenly h-full px-1 text-white text-[1.35rem]"
          // imgClassName="bg-blue-200 h-10 min-w-[2.8em] rounded-full text-lg"
        />
      </div>
    </div>
  );
}

```

- Note que Footer e Header são importados diretamente do módulo remoto (layoutSimon/Footer, layoutSimon/Header), como se fossem módulos locais. Isso é possível graças à configuração do remotes no vite.config.ts e às declarações de tipo.


### Exemplos de uso do componente Footer modularizado

O componente Footer, exposto pelo layout-simon, é consumido no simon-eva-front como um componente React comum, recebendo suas propriedades.


```js
// Dentro do App.tsx do simon-eva-front
<div className="pb-2 pt-2 ">
  <Footer
    items={footerItems} // Array de objetos que define os links do footer
    footerClassName="flex items-center justify-evenly h-full px-1 text-white text-[1.35rem]"
    // Você pode passar outras classes para customizar a aparência
    // imgClassName="bg-blue-200 h-10 min-w-[2.8em] rounded-full text-lg"
  />
</div>
```

O footerItems é uma constante definida no App.tsx que segue a interface FooterItem declarada no pacote simon-eva-types. O Footer é renderizado com base nesses itens, demonstrando a total integração entre os módulos.




## Passo-a-passo para a criação de um componente, exportação e uso em outro módulo

Vamos criar um novo componente Sidebar e integrá-lo ao seu ecossistema modular.

### Passo 1: Criar o Componente no Repositório layout-simon

Crie um novo arquivo src/components/Sidebar/Sidebar.tsx no projeto layout-simon:

```js

// layout-simon/src/components/Sidebar/Sidebar.tsx
import React from 'react';

export interface SidebarItem {
  icon: JSX.Element;
  label: string;
  path: string;
}

interface SidebarProps {
  items: SidebarItem[];
  sidebarClassName?: string;
  itemClassName?: string;
  labelClassName?: string;
}

const Sidebar: React.FC<SidebarProps> = ({
  items,
  sidebarClassName,
  itemClassName,
  labelClassName,
}) => {
  return (
    <nav className={sidebarClassName ?? "bg-gray-800 text-white p-4 h-full"}>
      <ul className="space-y-2">
        {items.map((item, index) => (
          <li key={index}>
            <a
              href={item.path}
              className={itemClassName ?? "flex items-center space-x-2 p-2 rounded-md hover:bg-gray-700"}
            >
              {item.icon}
              <span className={labelClassName ?? "text-lg"}>{item.label}</span>
            </a>
          </li>
        ))}
      </ul>
    </nav>
  );
};

export default Sidebar;
```

### Passo 2: Atualizar os Tipos no Repositório Types (simon-eva-types)

No seu repositório Types, adicione a declaração para o novo módulo Sidebar. Crie um arquivo types/layoutSimon/Sidebar.d.ts:


```js
/ simon-eva-types/types/layoutSimon/Sidebar.d.ts
declare module "layoutSimon/Sidebar" {
  import React from "react";

  export interface SidebarItem {
    icon: JSX.Element;
    label: string;
    path: string;
  }

  interface SidebarProps {
    items: SidebarItem[];
    sidebarClassName?: string;
    itemClassName?: string;
    labelClassName?: string;
  }

  const Sidebar: React.FC<SidebarProps>;
  export default Sidebar;
}

```

Após adicionar, você precisará publicar uma nova versão do seu pacote simon-eva-types no npm para que o projeto consumidor possa atualizá-lo.

### Passo 3: Exportar o Componente no Repositório layout-simon

Atualize o vite.config.ts do layout-simon para expor o novo componente Sidebar:

```js
// layout-simon/vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import federation from "@originjs/vite-plugin-federation";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
    federation({
      name: "layoutSimon",
      filename: "remoteEntry.js",
      exposes: {
        "./Footer": "./src/components/Footer/Footer.tsx",
        "./Header": "./src/components/Header/Header.tsx",
        "./Sidebar": "./src/components/Sidebar/Sidebar.tsx", // <--- Adicione esta linha
      },
      shared: ["react", "react-dom", "react-router-dom"],
    }),
  ],
  build: {
    target: "esnext",
    minify: false,
    cssCodeSplit: false,
  },
});
```

### Passo 4: Publicar/Servir o Repositório layout-simon
Para que o simon-eva-front possa consumir o novo módulo, o layout-simon precisa ser construído e estar acessível:

- Construa o layout-simon:
  
```
cd layout-simon
npm run build
```

- Sirva o layout-simon: Você pode usar um servidor HTTP simples ou o próprio comando npm run preview do Vite para testar localmente. Certifique-se de que ele esteja rodando na URL configurada em remotes no projeto host (ex: http://localhost:3002).

### Passo 5: Consumir o Componente no Repositório simon-eva-front

No repositório simon-eva-front, você precisará instalar a nova versão do pacote de tipagens e então importar e usar o componente.

- Atualize o pacote de tipos:

```
cd simon-eva-front
npm update simon-eva-types # Ou npm install simon-eva-types@latest
```


- Adicione a referência de tipo: Certifique-se de que /// <reference types="simon-eva-types" /> está no seu arquivo de declarações de tipo.

- Use o componente no App.tsx (ou onde for necessário):


```js
// simon-eva-front/src/App.tsx
import React from 'react';
// ... outros imports

import HomeIcon from "@mui/icons-material/Home";
import SettingsIcon from "@mui/icons-material/Settings";
import InfoIcon from "@mui/icons-material/Info";

import Sidebar from "layoutSimon/Sidebar"; // <--- Importe o novo componente

// ... (footerItems e outras constantes)

const sidebarItems = [
  { icon: <HomeIcon />, label: "Início", path: "/" },
  { icon: <SettingsIcon />, label: "Configurações", path: "/settings" },
  { icon: <InfoIcon />, label: "Sobre", path: "/about" },
];

export default function App() {
  return (
    <div className="flex flex-col min-h-screen bg-gray-900 text-white" id="app">
      <header className="bg-gray-800 p-4 border-b border-gray-700">
        {/* ... Header Component */}
      </header>
      <div className="flex flex-1"> {/* Adicione um container flex para layout de sidebar */}
        <aside className="w-64"> {/* Espaço para a sidebar */}
          <Sidebar items={sidebarItems} /> {/* <--- Use o componente Sidebar */}
        </aside>
        <main className="flex-1 p-6">
          <AppRoutes />
        </main>
      </div>
      <div className="pb-2 pt-2 ">
        {/* ... Footer Component */}
      </div>
    </div>
  );
}

```

Seguindo esses passos, você pode facilmente adicionar novos componentes ao seu repositório layout-simon, gerenciá-los com tipagens centralizadas e consumi-los em qualquer um dos seus projetos host, mantendo a arquitetura modular e escalável.
