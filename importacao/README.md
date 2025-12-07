# 📘 Projeto de Importação e Modelagem no Neo4j  
### Base de Exercícios – Atividades 1 a 8  

---

## 🧩 Atividade 1 – Importação Básica (LOAD CSV)

**Arquivo:** [clientes.csv](https://raw.githubusercontent.com/Brudesa/Neo4J/main/clientes.csv)  

```cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/Brudesa/Neo4J/main/clientes.csv' AS linha
CREATE (:Cliente {
  nome: linha.nome,
  email: toLower(trim(linha.email)),
  cidade: linha.cidade,
  data_importacao: date()
});
```

**Objetivo:** Importar dados básicos de clientes, normalizando e-mails e adicionando a data de importação.

---

## 🧹 Atividade 2 – Tratamento de Dados Sujos

**Arquivo:** [clientes_sujos.csv](https://raw.githubusercontent.com/Brudesa/Neo4J/main/clientes_sujos.csv)

```cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/Brudesa/Neo4J/main/clientes_sujos.csv' AS linha
WITH trim(linha.nome) AS nome_limpo,
     toLower(trim(linha.email)) AS email_limpo,
     toInteger(linha.idade) AS idade_num
WHERE email_limpo CONTAINS '@'
CREATE (:Cliente {
  id: toInteger(linha.id),
  nome: nome_limpo,
  email: email_limpo,
  idade: idade_num,
  data_importacao: date()
});
```

**Objetivo:** Limpar campos com `trim()`, normalizar e-mails e descartar linhas inválidas.

---

## 🔁 Atividade 3 – MERGE vs CREATE

**Arquivo:** [usuarios.csv](https://raw.githubusercontent.com/Brudesa/Neo4J/main/usuarios.csv)

```cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/Brudesa/Neo4J/main/usuarios.csv' AS linha
MERGE (u:Usuario {user_id: toInteger(linha.user_id)})
SET u.nome = linha.nome,
    u.perfil = linha.perfil;
```

**Explicação:**  
- `CREATE` sempre insere um novo nó, mesmo se já existir.  
- `MERGE` garante que o nó será criado **apenas se ainda não existir**.

---

## 🔗 Atividade 4 – Importação com Relacionamentos

**Arquivos:**  
- [clientes.csv](https://raw.githubusercontent.com/Brudesa/Neo4J/main/clientes.csv)  
- [pedidos.csv](https://raw.githubusercontent.com/Brudesa/Neo4J/main/pedidos.csv)

```cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/Brudesa/Neo4J/main/pedidos.csv' AS linha
MATCH (c:Cliente {id: toInteger(linha.cliente_id)})
CREATE (p:Pedido {
  pedido_id: toInteger(linha.pedido_id),
  valor: toFloat(linha.valor),
  data: date(linha.data)
})
CREATE (c)-[:FEZ_PEDIDO]->(p);
```

---

## ⚙️ Atividade 5 – Transações em Lote

**Arquivo:** pedidos.csv

```cypher
USING PERIODIC COMMIT 500
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/Brudesa/Neo4J/main/pedidos.csv' AS linha
MATCH (c:Cliente {id: toInteger(linha.cliente_id)})
CREATE (p:Pedido {
  pedido_id: toInteger(linha.pedido_id),
  valor: toFloat(linha.valor),
  data: date(linha.data)
})
CREATE (c)-[:FEZ_PEDIDO]->(p);
```

**Objetivo:** Importar grandes volumes de forma eficiente.

---

## 🌐 Atividade 6 – Importação via API (APOC.JSON)

```cypher
CALL apoc.load.json('https://jsonplaceholder.typicode.com/users')
YIELD value
CREATE (:Usuario {
  id: value.id,
  nome: value.name,
  email: value.email,
  cidade: value.address.city
});
```

**Dica:** Se estiver usando Aura e APOC estiver bloqueado, use JSON manual via `LOAD CSV` exportado.

---

## ⚡ Atividade 7 – Índices e Constraints

**Arquivo:** [produtos.csv](https://raw.githubusercontent.com/Brudesa/Neo4J/main/produtos.csv)

```cypher
CREATE CONSTRAINT produto_id_unico IF NOT EXISTS
FOR (p:Produto) REQUIRE p.produto_id IS UNIQUE;

CREATE INDEX produto_categoria IF NOT EXISTS
FOR (p:Produto) ON (p.categoria);
```

**Objetivo:** Melhorar performance de `MATCH` e garantir integridade.

---

## 🔍 Atividade 8 – Validação Pós-Importação

### 1. Contar total de nós por label
```cypher
CALL db.labels() YIELD label
CALL {
  WITH label
  RETURN count { MATCH (n:`${label}`) RETURN n } AS total
}
RETURN label AS tipo_de_no, total
ORDER BY total DESC;
```

---

### 2. Contar relacionamentos por tipo (compatível com Neo4j Aura)
```cypher
CALL db.relationshipTypes() YIELD relationshipType AS tipo
CALL {
  WITH tipo
  MATCH ()-[r]->()
  WHERE type(r) = tipo
  RETURN count(r) AS total
}
RETURN tipo AS tipo_relacionamento, total
ORDER BY total DESC;
```

📊 **Saída esperada:**
| tipo_relacionamento | total |
|----------------------|--------|
| LEU | 22 |
| PERTENCE_A | 20 |
| AMIGO_DE | 6 |
| AVALIOU | 3 |

---

### 3. Verificar valores nulos
```cypher
MATCH (n)
WHERE any(k IN keys(n) WHERE n[k] IS NULL)
RETURN labels(n) AS label, keys(n) AS propriedades, n;
```

---

### 4. Conferir relacionamentos quebrados
```cypher
MATCH ()-[r]->()
WHERE startNode(r) IS NULL OR endNode(r) IS NULL
RETURN type(r) AS relacionamento, r;
```

---

## ✅ Conclusão

Com este fluxo você:
- Importa dados limpos e sujos via `LOAD CSV`.
- Controla duplicatas com `MERGE`.
- Cria relacionamentos (`FEZ_PEDIDO`, `AVALIOU`).
- Define constraints e índices.
- Valida resultados no **Neo4j Aura** com queries 100% compatíveis.
