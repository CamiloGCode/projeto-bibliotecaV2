<h1 align="center">Sistema de Gerenciamento de Biblioteca</h1>

<p align="center">
  <img src="http://img.shields.io/static/v1?label=STATUS&message=EM%20DESENVOLVIMENTO&color=RED&style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Laravel-11-FF2D20?style=flat&logo=laravel&logoColor=white"/>
  <img src="https://img.shields.io/badge/Inertia.js-1.x-9553E9?style=flat&logo=inertia&logoColor=white"/>
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=flat&logo=react&logoColor=black"/>
  <img src="https://img.shields.io/badge/TypeScript-5.x-3178C6?style=flat&logo=typescript&logoColor=white"/>
  <img src="https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat&logo=postgresql&logoColor=white"/>
  <img src="https://img.shields.io/badge/License-Todos%20os%20direitos%20reservados-lightgrey?style=flat"/>
</p>

> Status do Projeto: (Em desenvolvimento)

### Tópicos

🔹 [Descrição do projeto](#descrição-do-projeto)
🔹 [Funcionalidades](#funcionalidades)
🔹 [Arquitetura](#arquitetura)
🔹 [Deploy da Aplicação](#deploy-da-aplicação)
🔹 [Alguns Arquivos Importantes](#alguns-arquivos-importantes)
🔹 [Pré-requisitos](#pré-requisitos)
🔹 [Iniciando/Configurando banco de dados](#iniciandoconfigurando-banco-de-dados)
🔹 [Casos de Uso](#casos-de-uso)
🔹 [Usuários](#usuários)
🔹 [Linguagens, dependências e libs utilizadas](#linguagens-dependências-e-libs-utilizadas-)
🔹 [Resolvendo Problemas](#resolvendo-problemas-)
🔹 [Desenvolvedores / Contribuintes](#desenvolvedores--contribuintes-octocat)
🔹 [Licença do Projeto](#licença)

## Descrição do projeto

<p align="justify">
O Sistema de Gerenciamento de Biblioteca é um Trabalho de Conclusão de Curso Técnico em Informática, com o objetivo de digitalizar o controle de acervo, empréstimos e devoluções de uma biblioteca, oferecendo cadastro de livros, autores, alunos e funcionários, além de relatórios em PDF e um dashboard com indicadores de uso.
</p>

<p align="justify">
Esta é a segunda versão do projeto, evoluída a partir de uma implementação inicial em PHP procedural. A V1 cumpriu seu papel didático, mas acumulou problemas estruturais — lógica de negócio misturada com HTML, consultas SQL vulneráveis a injection, senha armazenada em texto puro e nenhuma separação entre camadas. Esta versão aplica Domain-Driven Design tático e Clean Architecture sobre uma stack de monólito moderno (Laravel + Inertia.js + React), mantendo o domínio de negócio isolado de qualquer framework ou biblioteca de UI.
</p>

## Sobre esta reestruturação (v1 → v2)

<p align="justify">
Este repositório nasceu de um exercício de estudo: pegar um projeto já funcional (a v1, em PHP procedural) e reconstruí-lo aplicando, de forma deliberada, conceitos e práticas de engenharia de software que estão sendo estudados no momento. A ideia não é apenas "trocar de framework", mas usar os problemas reais encontrados na v1 como motivação concreta para cada conceito — em vez de aprender a teoria isolada, ela é aplicada para resolver um defeito específico que já existia.
</p>

| Conceito estudado | Problema na v1 que motivou o estudo | Como foi aplicado na v2 |
|---|---|---|
| **Domain-Driven Design (tático)** | Regra de negócio espalhada dentro de arquivos de view, sem entidades nem linguagem ubíqua | Entidades (`Livro`, `Emprestimo`), Value Objects e bounded contexts (`Catálogo`, `Circulação`, `Pessoas`) em `app/Domain/` |
| **Clean Architecture / Arquitetura em camadas** | Um único arquivo `.php` misturava HTML, SQL e validação | Separação em `Domain` → `Application` → `Infrastructure` → `Http`, com dependências sempre apontando para o domínio |
| **Inversão de Dependência** | `conexao.php` acoplava a regra de negócio diretamente ao `mysqli` | Interfaces de repositório (`LivroRepositoryInterface`) no domínio, implementadas no `Infrastructure`, permitindo trocar a persistência sem alterar regra de negócio |
| **Segurança da informação** | SQL Injection por concatenação de string, senha em texto puro, credenciais hardcoded | Eloquent/Query Builder (prepared statements por padrão), `Hash::make()`/`Hash::check()`, variáveis de ambiente via `.env` |
| **Versionamento de schema** | Um único `.txt` com o `CREATE TABLE`, editado manualmente e sem histórico | Migrations do Laravel, com `up()`/`down()` e histórico rastreável no Git |
| **Monólito moderno (Laravel + Inertia + React)** | Lógica de rota e validação duplicada entre PHP e JavaScript solto no `<script>` de cada página | Inertia.js elimina a necessidade de uma API REST separada, reaproveitando as rotas e validações (`FormRequest`) já existentes no backend |
| **Testabilidade** | Nenhum teste automatizado; qualquer alteração exigia teste manual em cada tela | Domínio isolado de framework permite testes unitários sem mock de banco (`tests/Unit`), além de testes de fluxo completo (`tests/Feature`) |

<p align="justify">
Essa tabela funciona como um diário de estudo: cada linha representa um
conceito que passou de teoria para prática dentro deste repositório. Ela
tende a crescer conforme novos tópicos forem sendo estudados e aplicados nas
próximas etapas do projeto.
</p>

## Funcionalidades

- [x] Cadastro e controle de livros
- [x] Cadastro e controle de autores
- [x] Cadastro e controle de funcionários, com autenticação
- [x] Cadastro de alunos
- [x] Empréstimo e devolução de exemplares
- [x] Dashboard com indicadores (acervo, empréstimos ativos, atrasos)
- [x] Geração de relatórios em PDF
- [x] Notificação de devolução próxima do prazo
- [x] Perfil de acesso (bibliotecário x administrador)

## Arquitetura

O projeto segue Clean Architecture com táticas de DDD, em quatro camadas. As
dependências sempre apontam para dentro, em direção ao domínio:

```
                            ┌─────────────────────────────────────────────┐
                            │  Presentation  (Http: Controllers, Inertia) │
                            │                    ↓                        │
                            │        Application  (Use Cases)             │
                            │                    ↓                        │
                            │      Domain  (Entidades, Value Objects,     │
                            │          interfaces de repositório)         │
                            │                    ↑                        │
                            │    Infrastructure  (Eloquent, PDF, filas)   │
                            └─────────────────────────────────────────────┘
```

Bounded contexts do domínio: 
- **Catálogo** (Livro, Autor), 
- **Circulação** (Empréstimo, Devolução) 
- **Pessoas** (Aluno, Funcionário).

Detalhes de decisões de design em [🏗️ Arquitetura do Sistema](docs/architecture.md).

## Deploy da Aplicação

<!-- Substituir pelo link do ambiente publicado, ou por um print/gif do sistema em funcionamento. -->
🔗 Link do deploy: _a definir_

## Alguns arquivos importantes

[⚙️ Setup do Ambiente](docs/setup.md) - Como configurar o ambiente e rodar o projeto localmente.

[🌿 Workflow de Desenvolvimento](docs/workflow.md) - Padrões de Git, Branches, Pull Requests, Testes e Deploy.

[🏗️ Arquitetura do Sistema](docs/architecture.md) - Tecnologias e Decisões de Design.

## Pré-requisitos

- PHP 8.2+
- Composer
- Node.js 18+
- PostgreSQL 16

## Iniciando/Configurando banco de dados

```bash
# 1. Clonar o repositório
git clone https://github.com/<seu-usuario>/<nome-do-repo>.git
cd <nome-do-repo>

# 2. Instalar dependências PHP
composer install

# 3. Instalar dependências JS
npm install

# 4. Configurar ambiente
cp .env.example .env
php artisan key:generate

# 5. Editar o .env com as credenciais do seu PostgreSQL local
#    DB_DATABASE, DB_USERNAME, DB_PASSWORD

# 6. Criar o banco e rodar as migrations + seeders
psql -U postgres -c "CREATE DATABASE biblioteca;"
php artisan migrate --seed

# 7. Subir o build de assets em modo desenvolvimento
npm run dev

# 8. Em outro terminal, subir o servidor Laravel
php artisan serve
```

A aplicação sobe em `http://localhost:8000`. Passo a passo detalhado em
[⚙️ Setup do Ambiente](docs/setup.md).

## Casos de Uso

<!-- Descrever os principais fluxos, ex: -->
- Funcionário realiza login e acessa o dashboard.
- Funcionário cadastra um novo livro vinculado a um autor existente.
- Funcionário registra o empréstimo de um exemplar para um aluno.
- Funcionário registra a devolução de um exemplar emprestado.
- Funcionário gera relatório em PDF do acervo.

## Usuários

<!-- Descrever os perfis de acesso do sistema, ex: -->
| Perfil | Permissões |
|---|---|
| Funcionário/Bibliotecário | Cadastro de livros, autores, alunos; empréstimo e devolução |
| Administrador | Todas as permissões acima + gestão de funcionários |

Login de teste (ambiente de desenvolvimento, via seeder):
- CPF: `123.456.789-00`
- Senha: `biblioteca123`

## Linguagens, dependências e libs utilizadas 📚

| Camada | Tecnologia |
|---|---|
| Backend | Laravel 11 (PHP 8.2+) |
| Ponte backend↔frontend | Inertia.js |
| Frontend | React 18 + TypeScript |
| Estilização | Tailwind CSS |
| Banco de dados | PostgreSQL 16 |
| Geração de PDF | barryvdh/laravel-dompdf |
| Autenticação | Laravel Breeze (stack Inertia + React) |
| Testes backend | PHPUnit / Pest |
| Testes frontend | Vitest + Testing Library |
| Build | Vite |

## Resolvendo Problemas ❗

Em [issues](https://github.com/<seu-usuario>/<nome-do-repo>/issues) foram
abertos alguns problemas gerados durante o desenvolvimento desse projeto e
como foram resolvidos.

## Desenvolvedores / Contribuintes :octocat:

O time responsável pelo desenvolvimento do projeto

[<img src="https://avatars.githubusercontent.com/u/163610849?v=4" width=115><br><sub>|Gabriel Camilo|</sub>]|(https://github.com/CamiloGCode)|

<!-- Substituir pelas fotos (avatars.githubusercontent.com/u/<id>?v=4) e links reais do GitHub de cada integrante, como no modelo do IQA Monitor. -->

## Licença

<p align="justify">
Este repositório é disponibilizado exclusivamente para fins de estudo, consulta técnica e apresentação em portfólio. O código permanece protegido por direitos autorais e não possui licença de uso open source. A utilização, reprodução, modificação ou redistribuição, total ou parcial, depende de autorização prévia e expressa do autor.
</p>

Veja [LICENSE.md](LICENSE.md) para mais detalhes.
