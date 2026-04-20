# Relatórios Avançados em SQL Northwind

## Objetivo

Este repositório reúne uma coleção de consultas e relatórios SQL de alto nível, projetados para transformar dados brutos em inteligência de negócios. As análises aqui presentes são escaláveis e aplicáveis a empresas de qualquer porte que buscam consolidar uma cultura data-driven, otimizando processos e fundamentando decisões em insights precisos.

## Relatórios que vamos criar

1. **Relatórios de Receita**
    
    * Qual foi o total de receitas no ano de 1997?

    ```sql
    CREATE VIEW total_revenues_1997_view AS
    SELECT
        SUM((d.unit_price * d.quantity) * (1 - d.discount)) AS faturamento
    FROM orders AS o
    INNER JOIN order_details AS d ON o.order_id = d.order_id AND o.order_date BETWEEN '1997-01-01' AND '1997-12-31';
    ```

    * Faça uma análise de crescimento mensal e o cálculo de YTD

    ```sql
    CREATE VIEW view_receitas_acumuladas AS
    WITH FaturamentoMensal AS (
        SELECT
            EXTRACT(YEAR FROM o.order_date) AS ano
            ,EXTRACT(MONTH FROM o.order_date) AS  mes
            ,SUM((d.unit_price * d.quantity) * (1 - d.discount)) AS faturamento
        FROM orders AS o
        INNER JOIN order_details AS d ON o.order_id = d.order_id AND o.order_date BETWEEN '1997-01-01' AND '1997-12-31'
        GROUP BY 
            EXTRACT(YEAR FROM o.order_date), EXTRACT(MONTH FROM o.order_date)
        ORDER BY 
            EXTRACT(YEAR FROM o.order_date) DESC, EXTRACT(MONTH FROM o.order_date) ASC
    )

    SELECT
        a.ano
        ,a.mes
        ,a.faturamento
        ,LAG(a.faturamento) OVER (ORDER BY a.ano DESC, a.mes ASC) AS mes_anterior
        ,(
            (a.faturamento -
            LAG(a.faturamento) OVER (ORDER BY a.ano DESC, a.mes ASC)) 
        ) AS diferenca_mensal
        ,(
            (a.faturamento -
            LAG(a.faturamento) OVER (ORDER BY a.ano DESC, a.mes ASC)) /
            LAG(a.faturamento) OVER (ORDER BY a.ano DESC, a.mes ASC)
        )*100 AS crescimento_percent
        ,SUM(a.faturamento) OVER (PARTITION BY a.ano ORDER BY a.mes) AS ytd
    FROM FaturamentoMensal AS a;
    ```

2. **Segmentação de clientes**
    
    * Qual é o valor total que cada cliente já pagou até agora?

    ```sql
    CREATE VIEW view_total_revenues_per_customer AS
    SELECT
        a.company_name
        ,SUM((c.unit_price * c.quantity) * (1 - c.discount)) AS pago
    FROM customers AS a
    LEFT JOIN orders AS b ON a.customer_id = b.customer_id
    LEFT JOIN order_details AS c ON  b.order_id = c.order_id
    GROUP BY a.company_name;
    ```

    * Separe os clientes em 5 grupos de acordo com o valor pago por cliente

    ```sql
    CREATE VIEW view_total_revenues_per_customer_group AS
    SELECT
        a.company_name
        ,SUM((c.unit_price * c.quantity) * (1 - c.discount)) AS pago
        ,NTILE(5) OVER (ORDER BY SUM((c.unit_price * c.quantity) * (1 - c.discount))) AS grupo
    FROM customers AS a
    LEFT JOIN orders AS b ON a.customer_id = b.customer_id
    LEFT JOIN order_details AS c ON  b.order_id = c.order_id
    GROUP BY a.company_name;
    ```


    * Agora somente os clientes que estão nos grupos 3, 4 e 5 para que seja feita uma análise de Marketing especial com eles

    ```sql
    CREATE VIEW clients_to_marketing AS
    WITH Clientes AS (
    SELECT
        a.company_name
        ,SUM((c.unit_price * c.quantity) * (1 - c.discount)) AS pago
        ,NTILE(5) OVER (ORDER BY SUM((c.unit_price * c.quantity) * (1 - c.discount))) AS grupo
    FROM customers AS a
    LEFT JOIN orders AS b ON a.customer_id = b.customer_id
    LEFT JOIN order_details AS c ON  b.order_id = c.order_id
    GROUP BY a.company_name
    )

    SELECT
        *
    FROM Clientes
    WHERE grupo IN (3,4,5);
    ```

3. **Top 10 Produtos Mais Vendidos**
    
    * Identificar os 10 produtos mais vendidos.

    ```sql
    CREATE VIEW top_10_products AS
    SELECT 
        b.product_name
        ,SUM(a.quantity) AS quantidade
    FROM order_details AS a
    LEFT JOIN products AS b ON a.product_id = b.product_id
    GROUP BY b.product_name
    ORDER BY quantidade DESC
    LIMIT 10;
    ```

4. **Clientes do Reino Unido que Pagaram Mais de 1000 Dólares**
    
    * Quais clientes do Reino Unido pagaram mais de 1000 dólares?

    ```sql
    CREATE VIEW uk_clients_who_pay_more_then_1000 AS
    SELECT
        a.company_name
        ,SUM((c.unit_price * c.quantity) * (1 - c.discount)) AS pago
    FROM customers AS a
    LEFT JOIN orders AS b ON a.customer_id = b.customer_id
    LEFT JOIN order_details AS c ON  b.order_id = c.order_id
    WHERE
        a.country = 'UK'
    GROUP BY a.company_name
    HAVING SUM((c.unit_price * c.quantity) * (1 - c.discount)) > 1000
    ORDER BY SUM((c.unit_price * c.quantity) * (1 - c.discount)) DESC;
    ```

## Contexto

O banco de dados `Northwind` contém os dados de vendas de uma empresa  chamada `Northwind Traders`, que importa e exporta alimentos especiais de todo o mundo. 

O banco de dados Northwind é ERP com dados de clientes, pedidos, inventário, compras, fornecedores, remessas, funcionários e contabilidade.

O conjunto de dados Northwind inclui dados de amostra para o seguinte:

* **Fornecedores:** Fornecedores e vendedores da Northwind
* **Clientes:** Clientes que compram produtos da Northwind
* **Funcionários:** Detalhes dos funcionários da Northwind Traders
* **Produtos:** Informações do produto
* **Transportadoras:** Os detalhes dos transportadores que enviam os produtos dos comerciantes para os clientes finais
* **Pedidos e Detalhes do Pedido:** Transações de pedidos de vendas ocorrendo entre os clientes e a empresa

O banco de dados `Northwind` inclui 14 tabelas e os relacionamentos entre as tabelas são mostrados no seguinte diagrama de relacionamento de entidades.

![northwind](https://github.com/FerruccioRibeiro/northwind/blob/main/pics/northwind-er-diagram.png?raw=true)

## Objetivo

O objetivo desse 

## Configuração Inicial

### Manualmente

Utilize o arquivo SQL fornecido, `nortwhind.sql`, para popular o seu banco de dados.

### Com Docker e Docker Compose

**Pré-requisito**: Instale o Docker e Docker Compose

* [Começar com Docker](https://www.docker.com/get-started)
* [Instalar Docker Compose](https://docs.docker.com/compose/install/)

### Passos para configuração com Docker:

1. **Iniciar o Docker Compose** Execute o comando abaixo para subir os serviços:
    
    ```
    docker compose up -d
    ```
    
    Aguarde as mensagens de configuração, como:
    
    ```csharp
    Creating network "northwind_psql_db" with driver "bridge"
    Creating volume "northwind_psql_db" with default driver
    Creating volume "northwind_psql_pgadmin" with default driver
    Creating pgadmin ... done
    Creating db      ... done
    ```
       
2. **Conectar o PgAdmin** Acesse o PgAdmin pelo URL: [http://localhost:5050](http://localhost:5050), com a senha `postgres`. 

Configure um novo servidor no PgAdmin:
    
    * **Aba General**:
        * Nome: db
    * **Aba Connection**:
        * Nome do host: db
        * Nome de usuário: postgres
        * Senha: postgres Em seguida, selecione o banco de dados "northwind".

3. **Parar o Docker Compose** Pare o servidor iniciado pelo comando `docker-compose up` usando Ctrl-C e remova os contêineres com:
    
    ```
    docker compose down
    ```
    
4. **Arquivos e Persistência** Suas modificações nos bancos de dados Postgres serão persistidas no volume Docker `postgresql_data` e podem ser recuperadas reiniciando o Docker Compose com `docker-compose up`. Para deletar os dados do banco, execute:
    
    ```
    docker compose down -v
    ```
