# Atividade III - SEDUC DEST2: Banco de Dados Relacional
**Instituição:** SEDUC  
**Componente Curricular:** Banco de Dados / SGBD  
**Domínio Escolhido:** Livraria Páginas Iniciais  
**Data de Entrega:** 03/11 11:45  
**Apresentação:** 04/11 08:00  

---

## 1. Domínio do Banco de Dados
**Domínio:** Gerenciamento de Acervo, Clientes, Autores, Gêneros e Vendas de uma Livraria.

---

## 2. Contextualização
A **Livraria Páginas Iniciais** vende livros físicos de diversas editoras e autores. Atualmente, a empresa gerencia todo o estoque, catálogo e histórico de compras dos clientes usando planilhas soltas do Excel e cadernos de anotações no balcão.

### Problemas Enfrentados sem um SGBD Relacional:
* **Inconsistência e Redundância:** O mesmo livro é cadastrado com nomes ligeiramente diferentes (ex: "O Senhor dos Anéis" e "Senhor dos Aneis, O"), duplicando registros e bagunçando o controle de estoque.
* **Falta de Integridade:** Não há validação automatizada de dados. O sistema aceita a venda de livros que não estão mais fisicamente na prateleira.
* **Histórico Perdido:** É impossível saber com precisão quais livros um cliente frequente comprou no último ano para oferecer recomendações personalizadas.
* **Relatórios Lentos:** Descobrir qual o autor ou gênero mais vendido no mês exige horas de cruzamento manual de dados.

---

## 3. A Proposta de Melhoria
A solução consiste na migração dos dados para um Sistema de Gerenciamento de Banco de Dados Relacional (**MySQL**).

**Impacto Esperado:** Centralização e consistência do acervo através do vínculo correto entre Livros e Autores. Automatização do controle de saídas nas vendas e geração instantânea de relatórios de faturamento, ticket médio e preferências de leitura dos clientes.

---

## 4. Modelagem Lógica do Banco de Dados

### 4.1. Tabelas, Campos e Tipos de Dados

1. **`genero`**
   * `id_genero` INT AUTO_INCREMENT (PK - Primary Key)
   * `nome` VARCHAR(50) NOT NULL

2. **`autor`**
   * `id_autor` INT AUTO_INCREMENT (PK - Primary Key)
   * `nome` VARCHAR(100) NOT NULL

3. **`livro`**
   * `id_livro` INT AUTO_INCREMENT (PK - Primary Key)
   * `titulo` VARCHAR(150) NOT NULL
   * `isbn` VARCHAR(17) UNIQUE NOT NULL
   * `preco` DECIMAL(10,2) NOT NULL
   * `estoque` INT NOT NULL
   * `id_genero` INT (FK - Foreign Key)
   * `id_autor` INT (FK - Foreign Key)

4. **`cliente`**
   * `id_cliente` INT AUTO_INCREMENT (PK - Primary Key)
   * `nome` VARCHAR(100) NOT NULL
   * `cpf` VARCHAR(14) UNIQUE NOT NULL
   * `email` VARCHAR(100) NOT NULL

5. **`venda`**
   * `id_venda` INT AUTO_INCREMENT (PK - Primary Key)
   * `data_venda` DATETIME NOT NULL
   * `id_cliente` INT (FK - Foreign Key)

6. **`item_venda`** *(Tabela Associativa N:N)*
   * `id_venda` INT (PK, FK)
   * `id_livro` INT (PK, FK)
   * `quantidade` INT NOT NULL
   * `preco_unitario` DECIMAL(10,2) NOT NULL

---

### 4.2. Relacionamentos e Cardinalidades

| Tabela Origem | Tabela Destino | Chave de Ligação | Cardinalidade | Descrição da Regra de Negócio |
| :--- | :--- | :--- | :--- | :--- |
| **genero** | **livro** | FK (`id_genero` em `livro`) | **1 : N** | 1 gênero pode ter vários livros; cada livro tem 1 gênero. |
| **autor** | **livro** | FK (`id_autor` em `livro`) | **1 : N** | 1 autor escreve vários livros; cada livro tem 1 autor associado. |
| **cliente** | **venda** | FK (`id_cliente` em `venda`) | **1 : N** | 1 cliente realiza várias compras; cada venda pertence a 1 cliente. |
| **venda** | **livro** | Tabela Associativa (`item_venda`) | **N : N** | 1 venda contém vários livros, e 1 livro pode ser vendido em várias compras. |

---

## 5. Aplicação das Formas Normais (FN)

1. **1FN (Primeira Forma Normal):** 
   * **Aplicação:** Eliminamos todos os atributos multivalorados ou compostos. Cada campo contém apenas valores atômicos (invisíveis a subdivisões). Não existem colunas repetidas na mesma tabela (ex: `telefone1`, `telefone2`).
2. **2FN (Segunda Forma Normal):** 
   * **Aplicação:** O banco atende à 1FN e todos os atributos não-chave dependem da chave primária completa. Na tabela associativa `item_venda` (chave composta por `id_venda` + `id_livro`), o campo `quantidade` depende unicamente do par completo de chaves.
3. **3FN (Terceira Forma Normal):** 
   * **Aplicação:** Foram removidas todas as dependências transitivas entre atributos não-chave. Informações como o nome do autor ou gênero não são gravadas na tabela `livro`, apenas suas respetivas chaves estrangeiras (`id_autor` e `id_genero`).
4. **4FN (Quarta Forma Normal):** 
   * **Aplicação:** Garantimos a ausência de dependências multivaloradas independentes dentro da mesma entidade. O relacionamento N:N entre `venda` e `livro` foi devidamente isolado na tabela associativa `item_venda`.

---

## 6. Instruções SQL

### 6.1. DDL - Criação do Banco de Dados e Tabelas

```sql
CREATE DATABASE paginas_iniciais_db;
USE paginas_iniciais_db;

CREATE TABLE genero (
    id_genero INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(50) NOT NULL
);

CREATE TABLE autor (
    id_autor INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL
);

CREATE TABLE livro (
    id_livro INT AUTO_INCREMENT PRIMARY KEY,
    titulo VARCHAR(150) NOT NULL,
    isbn VARCHAR(17) UNIQUE NOT NULL,
    preco DECIMAL(10,2) NOT NULL,
    estoque INT NOT NULL,
    id_genero INT,
    id_autor INT,
    FOREIGN KEY (id_genero) REFERENCES genero(id_genero),
    FOREIGN KEY (id_autor) REFERENCES autor(id_autor)
);

CREATE TABLE cliente (
    id_cliente INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cpf VARCHAR(14) UNIQUE NOT NULL,
    email VARCHAR(100) NOT NULL
);

CREATE TABLE venda (
    id_venda INT AUTO_INCREMENT PRIMARY KEY,
    data_venda DATETIME NOT NULL,
    id_cliente INT,
    FOREIGN KEY (id_cliente) REFERENCES cliente(id_cliente)
);

CREATE TABLE item_venda (
    id_venda INT,
    id_livro INT,
    quantidade INT NOT NULL,
    preco_unitario DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (id_venda, id_livro),
    FOREIGN KEY (id_venda) REFERENCES venda(id_venda),
    FOREIGN KEY (id_livro) REFERENCES livro(id_livro)
);
![Diagrama Lógico do Banco de Dados]<img width="935" height="728" alt="image" src="https://github.com/user-attachments/assets/f72bbac5-70fe-4e86-b88f-f3b662cbf051" />
