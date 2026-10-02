# 🏥 Projeto Java Saúde 

Sistema de gestão de clínica em **Java**, executado no terminal. Permite cadastrar pacientes, médicos e salas, agendar consultas com validação de conflitos, controlar uma fila de espera e consultar indicadores básicos.

![Java](https://img.shields.io/badge/Java-21-orange)
![Maven](https://img.shields.io/badge/Maven-3.9-blue)
![H2](https://img.shields.io/badge/H2-2.3.232-lightgrey)
![JUnit](https://img.shields.io/badge/JUnit-5.11-green)



---

## 📌 Problema

Clínicas e hospitais costumam enfrentar salas de espera cheias, marcações duplicadas e dificuldade para saber quais médicos estão disponíveis.

O **Saúde Simples** resolve uma parte desse problema: ele impede conflitos de horário entre paciente, médico e sala, e organiza quem está aguardando atendimento.

Fonte do problema: [Revista Master](https://revistamaster.emnuvens.com.br/RM/article/download/802/385)

## 👥 Usuários

- Recepcionista
- Secretário(a)
- Administrador da clínica

---

## ✨ Funcionalidades

| # | Opção do menu | O que faz |
|---|---|---|
| 1 | Cadastrar paciente | Cria um paciente (ID, nome, idade). Nasce ativo. |
| 2 | Listar pacientes | Mostra todos os pacientes cadastrados. |
| 3 | Pesquisar paciente | Busca por parte do nome, sem diferenciar maiúsculas, em ordem alfabética. |
| 4 | Editar paciente | Altera nome e idade. |
| 5 | Desativar paciente | Marca o paciente como inativo (não recebe novas consultas). |
| 6 | Cadastrar médico | Cria um médico (ID, nome, especialidade). Nasce disponível. |
| 7 | Cadastrar sala | Cria uma sala (ID, nome). |
| 8 | Agendar consulta | Cria uma consulta aplicando as regras de negócio. |
| 9 | Listar consultas | Mostra as consultas em ordem de data. |
| 10 | Concluir consulta | Muda o status para `REALIZADA`. |
| 11 | Cancelar consulta | Muda o status para `CANCELADA`. |
| 12 | Entrar na fila | Coloca um paciente na fila de espera. |
| 13 | Atender fila | Remove o primeiro paciente da fila. |
| 14 | Indicadores | Mostra os relatórios (veja abaixo). |
| 15 | Exportar CSV | Gera `data/pacientes.csv`. |
| 16 | Salvar paciente no H2 | Grava um paciente no banco de dados. |
| 0 | Sair | Encerra o programa. |

---

## 🧱 Arquitetura

### Classes do domínio

| Classe | Responsabilidade |
|---|---|
| `Paciente` | Guarda ID, nome, idade, situação (ativo/inativo) e suas consultas. |
| `Medico` | Guarda ID, nome, especialidade e disponibilidade. |
| `Sala` | Guarda ID, nome e disponibilidade. |
| `Consulta` | Liga um paciente, um médico e uma sala a uma data/hora. Status: `AGENDADA`, `REALIZADA` ou `CANCELADA`. |
| `FilaEspera` | Lista de pacientes aguardando atendimento. |
| `Clinica` | Classe central: guarda todas as listas e concentra as regras de negócio e os indicadores. |

### Classes auxiliares

| Classe | Responsabilidade |
|---|---|
| `Main` | Menu de terminal e leitura do teclado. Exibe os erros de regra como `ERRO: ...` sem encerrar o programa. |
| `Banco` | Integração com o banco H2 (JDBC). |
| `CSV` | Exportação de pacientes para arquivo CSV. |

### Relacionamentos

- `Clinica` possui vários pacientes, médicos, salas e consultas, além de uma `FilaEspera`.
- `FilaEspera` possui vários pacientes.
- `Paciente` possui várias consultas.
- `Consulta` possui um paciente, um médico e uma sala.

---

## 📏 Regras de negócio

Implementadas na classe `Clinica`:

1. Paciente, médico e sala precisam existir para agendar.
2. Paciente **inativo** não pode receber consultas nem entrar na fila.
3. Médico **indisponível** não pode receber novas consultas.
4. O mesmo **paciente** não pode ter duas consultas no mesmo horário.
5. O mesmo **médico** não pode ter duas consultas no mesmo horário.
6. A mesma **sala** não pode ter duas consultas no mesmo horário.
7. Consultas **canceladas** não contam para conflito de horário.
8. Só é possível **concluir** uma consulta que esteja `AGENDADA`.
9. Uma consulta **já realizada** não pode ser cancelada.
10. Um paciente **não entra duas vezes** na fila de espera.
11. Ao **agendar**, a sala passa a ficar ocupada. Ao **realizar ou cancelar** a consulta, a sala volta a ficar livre.

Quando uma regra é violada, o sistema exibe uma mensagem de erro clara e continua funcionando.

## 📊 Indicadores

| Tipo | Indicador |
|---|---|
| Média | Média de idade dos pacientes |
| Total | Total de consultas cadastradas |
| Maior valor | Maior idade entre os pacientes |
| Ranking | Médicos com mais consultas |
| Indicador próprio | Quantidade de pessoas na fila de espera |

---

## 🔌 Integrações

### Banco de dados H2
Banco leve escrito em Java, sem necessidade de instalação. A conexão usa **JDBC** com a URL `jdbc:h2:./data/saude`, e os arquivos do banco ficam na pasta `data/`. A opção 16 do menu cria a tabela `paciente` (se não existir) e grava o paciente, atualizando-o caso o ID já exista (`MERGE ... KEY(id)`).

### Exportação CSV
A opção 15 gera o arquivo `data/pacientes.csv` com o cabeçalho `id,nome,idade,ativo` e uma linha por paciente. O arquivo pode ser aberto em qualquer editor de planilhas.

> ℹ️ A exportação lê os pacientes que estão **em memória** (cadastrados na sessão atual), e não os do banco H2. As duas integrações funcionam de forma independente.

---

## 🛠️ Tecnologias

- **Java 21**
- **Maven 3.9** (build e gerenciamento de dependências)
- **H2 Database 2.3.232**
- **JUnit 5.11** (testes) com **Maven Surefire 3.5.0**

---

## 🚀 Como executar

### Pré-requisitos

- [JDK 21](https://adoptium.net) instalado, com a variável `JAVA_HOME` configurada
- [Maven 3.9+](https://maven.apache.org/download.cgi) instalado e disponível no `PATH`

Confira com:

```bash
java -version
mvn -version
```

### Passo a passo

```bash
# 1. Clone o repositório e entre na pasta do projeto (onde está o pom.xml)
git clone <url-do-repositorio>
cd SaudeSimples

# 2. Execute os testes
mvn test

# 3. Compile
mvn compile
```

### Rodando o programa

**Pela IDE (IntelliJ, Eclipse ou VS Code):** abra a pasta do projeto e execute a classe `saude.Main`.

**Pelo terminal:**

```bash
mvn compile exec:java -Dexec.mainClass=saude.Main
```

> Execute sempre a partir da raiz do projeto (`SaudeSimples`). A pasta `data/` é criada em relação ao diretório de onde o programa é iniciado.

---

## 🧪 Testes

O teste automatizado `ClinicaTest` fica em `src/test/java/saude/` e cobre dois casos: uma segunda consulta no mesmo horário é recusada, e a sala fica ocupada ao agendar e livre ao concluir ou cancelar.

```bash
mvn test
```

Saída esperada ao final: `BUILD SUCCESS`.

---

## 🎬 Exemplo de uso

1. Cadastre um paciente: `ID 1`, `Ana`, `20 anos` (opção 1)
2. Cadastre um médico: `ID 1`, `João`, `Clínica Geral` (opção 6)
3. Cadastre uma sala: `ID 1`, `Sala 1` (opção 7)
4. Agende uma consulta (opção 8): `ID 1`, paciente `1`, médico `1`, sala `1`, data `10/10/2026 10:00`
5. Tente agendar outra consulta no mesmo horário com o mesmo médico (ou paciente, ou sala). O sistema recusa com uma mensagem como:

```
ERRO: Paciente já possui consulta nesse horário.
```

---

## 📁 Estrutura do projeto

```
SaudeSimples/
├── pom.xml
├── README.md
└── src/
    ├── main/java/saude/
    │   ├── Main.java
    │   ├── Clinica.java
    │   ├── Paciente.java
    │   ├── Medico.java
    │   ├── Sala.java
    │   ├── Consulta.java
    │   ├── FilaEspera.java
    │   ├── Banco.java
    │   └── CSV.java
    └── test/java/saude/
        └── ClinicaTest.java
```

---

## ⚠️ Limitações conhecidas

- Os dados ficam **em memória**: ao fechar o programa, pacientes, médicos, salas e consultas cadastrados são perdidos. O H2 só recebe os pacientes salvos manualmente pela opção 16, e eles não são recarregados ao reiniciar.
- Os IDs são digitados pelo usuário e não há verificação de IDs duplicados.
- A situação da sala (ocupada/livre) é só informativa: ela não impede novos agendamentos em outros horários. O conflito é verificado pela data e hora das consultas.

---

## 🤖 Uso de IA

| Ferramenta | Objetivo | Resumo do uso | Revisão pelo Dev |
|---|---|---|---|
| ChatGPT | Código base e documentação | Auxiliou na criação da estrutura simples, classes, regras de negócio, menu e README. | Código e regras devem ser testados e revisados pelos integrantes antes da apresentação. |
| Claude | Revisão e documentação | Auxiliou na resolução de erros de ambiente (Maven e CSV), na revisão do código e na reescrita deste README. | Conteúdo conferido com o código-fonte. |
