# Live #5 - Explorando o poder do NoSQL com Apache Cassandra e Apache Hbase

**[NoSQL](https://pt.wikipedia.org/wiki/NoSQL)** é um termo genérico que representa os bancos de dados não relacionais. <br>
**[Apache Cassandra](https://pt.wikipedia.org/wiki/Apache_Cassandra)** é um projeto de sistema de banco de dados distribuído altamente escalável de segunda geração, que reúne a arquitetura do DynamoDB, da Amazon Web Services e modelo de dados baseado no BigTable, do Google. <br>
**[Apache Hbase](https://pt.wikipedia.org/wiki/HBase)** é um banco de dados distribuído open-source orientado a coluna(Column Family ou Wide Column), modelado a partir do Google BigTable e escrito em Java. <br>

* Hbase: dicionário de dicionários / json / map
* Cassandra: tipado, tabela secundária de índice, filtros fora da primary key, CQL

Profº [**Valdir Sevaios**](https://www.linkedin.com/in/valdir-novo-sevaios-junior-8190a096/) <br>

### 1 - Live Demo

### 1.1 - HBase
~~~shell

# start services
sh script_apoio/start_all_service.sh

# Caso esteja no safe mode. Executar no shell. Ele entra no safe mode quando os serviços do hadoop não foram parados antes de desligar a VM.
sudo -u hdfs hadoop dfsadmin -safemode leave

# Verificar o que está corrompido no HDFS. Check Yarn. Executar no shell
sudo -u hdfs hadoop fsck / | egrep -v '^\.+$' | grep -v eplica

# Caso exiba algum arquivo corrompido será necessário apagar ou recuperar.
sudo -u hdfs hdfs dfs -rm <caminho do arquivo>

# Acessando
hbase shell

# Exibir versão do HBase
version

# Lista todas as tabelas no HBase independente do namespace.
list

# Exibir comandos que se referenciam a uma tabela
table_help

# Criando uma tabela
create 'funcionario', 'pessoais', 'profissionais'

# Verificar se a tabela está habilitada ou não
is_enabled 'funcionario'

# Inserindo dados na tabela
put 'funcionario', '1', 'pessoais:nome', 'Maria'
# Visualizando os registros
scan 'funcionario'

# Inserindo dados na tabela
put 'funcionario', '1', 'pessoais:cidade', 'Sao Paulo'
put 'funcionario', '2', 'profissionais:empresa', 'Everis'

# Alterando dados na tabela
## Desabilitando tabela
disable 'funcionario'

## Alterando tabela
alter 'funcionario', NAME=>'hobby', VERSIONS=>5

## Habilitando tabela
enable 'funcionario'

# Inserindo dados na tabela IV
put 'funcionario', '1', 'hobby:nome', 'Ler livros'
put 'funcionario', '1', 'hobby:nome', 'Pescar'

# Visualizando versionamento da tabela
scan 'funcionario', {VERSIONS=>3}

# Contar as rows keys
count 'funcionario'

# Deletar registros da tabela. Ele deleta a última versão do registro
delete 'funcionario', '1', 'hobby:nome'

# Criando Time To Live (TTL). O Exemplo abaixo o registro ficará disponível por 20 segundos
create 'ttl_exemplo', {'NAME'=>'cf', 'TTL' => 20}
# Inserindo registro de exemplo para o Time To Live
put 'ttl_exemplo', 'linha123', 'cf:desc', 'TTL Exemplo'
# Print
get 'ttl_exemplo', 'linha123', 'cf:desc'

# Visualizar detalhes sobre o sistema
status
#Variações: 
status 'simple'
status 'summary'
status 'detailed'

## Comandos utilizandos para operar as tabelas no HBase
# Verifica se uma tabela existe.
exists 'table_name' 

# Cria uma tabela, com as colunas
create 'table_name', 'col_name1', 'col_name2'

# Desabilita a tabela para delete ou exclusão no HBase
disable 'table_name'

# Verifica se uma tabela está desabilitada.
is_disabled 'table_name'

# Habilita uma tabela.
enable 'table_name'

# Verificar se a tabela está habilitada ou não
is_enabled 'table_name'

# Alterar propriedades de tabelas ou acrescentar column family
alter 'table_name', NAME=>'col_name3', VERSIONS => 5
alter 'table_name', 'delete'=>'col_name1'

# Exibe informações de definição de uma tabela.
describe 'table_name'
desc 'table_name'

# Exibe o status das alterações realizadas de uma tabela
alter_status 'table_name'

# Exclui um tabela do HBase.
drop 'table_name'

# Desabilitar as tabelas que atendem dentro do critério Regex
help 'disable_all'
disable_all 'table_n.*'

# Deletar todas as tabelas dentro do Regex
drop_all 'table_n.*'

# Criar uma namespace no HBase
create_namespace 'namespace_name'

## Comandos de manipulação (CRUD)
# Insere/Atualiza um valor em uma determinada célula de uma específica linha de uma tabela.
put '<nome da tabela>', '<rowkey>', '<columnfamily:colname>', '<valor>'

enable 'funcionario'
put 'funcionario', '1', 'pessoais:nome', 'Maria'
put 'funcionario', '1', 'pessoais:cidade', 'São Paulo'
put 'funcionario', '1', 'pessoais:cidade', 'Belo Horizonte'
put 'funcionario', '2', 'profissionais:empresa', 'Everis

# Varre toda a tabela retornando as dados contidos.
scan '<nome da tabela>', '[parâmetros opcionais]
scan 'funcionario', {VERSIONS => 3}
# Exibe todas as últimas 10 versões de cada rowKey e respectivas colunas
scan 'funcionario', {RAW=>true, version=>10}

# Consulta o conteúdo de uma linha ou célula em uma tabela
get '<nome da tabela>', '<rowkey>', [parâmetros opcionais]
get 'funcionario', '1'
get 'funcionario', '1', {COLUMN => 'pessoais:cidade'}
get 'funcionario', '1' , {COLUMN => 'pessoais:cidade', VERSIONS=> 3}

# Exclui um valor de uma célula em uma tabela.
delete '<nome da tabela>', '<rowkey>', '<column name>'
delete <'nome da tabela>', '<rowkey>', '<column name>', '<timestamp'>
delete 'funcionario', '1', 'pessoais:cidade'
delete 'funcionario', '1', 'pessoais:cidade', 129019101

# Exclui todas as células de uma linha em específico.
deleteall '<nome da tabela>','<rowkey>'
deleteall 'funcionario', '1'

# Conta e retorna o número de linhas em uma tabela.
count '<nome da tabela>', CACHE => 1000
count 'funcionario', CACHE=> 1000

# Desabilita, exclui e recria uma tabela em específico.
truncate '<nome da tabela>'
truncate 'funcionario'


~~~

### 1.2 - Cassandra
~~~shell

#Acessando
cqlsh

#Ajuda sobre os comandos
help
help alter #Exemplo

#Criação do keyspace (schema, namespace)
CREATE KEYSPACE empresa
WITH replication = {'class':'SimpleStrategy', 'replication_factor' : 3};

#Criação de tabela
CREATE TABLE empresa.funcionario
  (
    empregadoid int primary key,
    empregadonome text,
    empregadocargo text,
  );
    
#Criação de índice secundário para consulta
CREATE INDEX rempresacargo ON empresa.funcionario (empregadocargo);

#Inserindo dados no keyspace
INSERT INTO
"empresa"."funcionario" (empregadoid, empregadonome, empregadocargo)
VALUES (1, 'Felipe', 'Engenheiro de Dados'); 

#Visualizando keyspace
SELECT * FROM "empresa"."funcionario";
~~~

### 2 - Atividades
1. Criar uma tabela que representa lista de Cidades e que permite armazenar até 5 versões na Column Family com os seguintes campos:
~~~shell
Código da cidade como rowKey
#Column Family=info
#        Nome da Cidade
#        Data de Fundação
#Column Family=responsaveis
#        Nome Prefeito
#        Data de Posse do Prefeito
#        Nome Vice prefeito
#Column Family=estatisticas
#        Data da última Eleição
#        Quantidade de moradores
#        Quantidade de eleitores
#        Ano de fundação

#Criando Tabela
create 'cidade', 'info', 'responsaveis', 'estatisticas'

#Versionamento
alter 'cidade', NAME='info', VERSIONS =>5
alter 'cidade', NAME='responsaveis', VERSIONS =>5
alter 'cidade', NAME='estatisticas', VERSIONS =>5
~~~



2. Inserir 10 cidades na tabela criada de cidades.
[https://github.com/viniciusriosfuck/Aceleracao_Global_Dev4_Everis/blob/master/WebScrappingWiki_DadosCapitais.ipynb](Notebook)
~~~shell
put 'cidade', '1', 'info:cidadenome', 'São Paulo'
put 'cidade', '1', 'info:estadonome', 'São Paulo'
put 'cidade', '1', 'estatisticas:moradoresquantidade', '12325232'
put 'cidade', '1', 'responsaveis:prefeitonome', 'Bruno Covas'
put 'cidade', '1', 'responsaveis:prefeitopartido', 'PSDB'
put 'cidade', '1', 'responsaveis:viceprefeitonome', 'Ricardo Nunes'
put 'cidade', '1', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '1', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '1', 'estatisticas:eleitoresquantidade', '8886325'
put 'cidade', '1', 'info:cidadefundacaodata', '25/1/1554'
put 'cidade', '1', 'estatisticas:cidadefundacaoano', '1554'
put 'cidade', '2', 'info:cidadenome', 'Rio de Janeiro'
put 'cidade', '2', 'info:estadonome', 'Rio de Janeiro'
put 'cidade', '2', 'estatisticas:moradoresquantidade', '6747815'
put 'cidade', '2', 'responsaveis:prefeitonome', 'Eduardo Paes'
put 'cidade', '2', 'responsaveis:prefeitopartido', 'DEM'
put 'cidade', '2', 'responsaveis:viceprefeitonome', 'Nilton Caldeira'
put 'cidade', '2', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '2', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '2', 'estatisticas:eleitoresquantidade', '4898040'
put 'cidade', '2', 'info:cidadefundacaodata', '1/3/1565'
put 'cidade', '2', 'estatisticas:cidadefundacaoano', '1565'
put 'cidade', '3', 'info:cidadenome', 'Salvador'
put 'cidade', '3', 'info:estadonome', 'Bahia'
put 'cidade', '3', 'estatisticas:moradoresquantidade', '2886698'
put 'cidade', '3', 'responsaveis:prefeitonome', 'Bruno Reis'
put 'cidade', '3', 'responsaveis:prefeitopartido', 'DEM'
put 'cidade', '3', 'responsaveis:viceprefeitonome', 'Ana Paula Matos'
put 'cidade', '3', 'estatisticas:ultimaeleicaodata', '15/11/2020'
put 'cidade', '3', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '3', 'estatisticas:eleitoresquantidade', '1948154'
put 'cidade', '3', 'info:cidadefundacaodata', '29/3/1549'
put 'cidade', '3', 'estatisticas:cidadefundacaoano', '1549'
put 'cidade', '4', 'info:cidadenome', 'Fortaleza'
put 'cidade', '4', 'info:estadonome', 'Ceará'
put 'cidade', '4', 'estatisticas:moradoresquantidade', '2686612'
put 'cidade', '4', 'responsaveis:prefeitonome', 'Sarto Nogueira'
put 'cidade', '4', 'responsaveis:prefeitopartido', 'PDT'
put 'cidade', '4', 'responsaveis:viceprefeitonome', 'Élcio Batista'
put 'cidade', '4', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '4', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '4', 'estatisticas:eleitoresquantidade', '1692712'
put 'cidade', '4', 'info:cidadefundacaodata', '13/4/1726'
put 'cidade', '4', 'estatisticas:cidadefundacaoano', '1726'
put 'cidade', '5', 'info:cidadenome', 'Belo Horizonte'
put 'cidade', '5', 'info:estadonome', 'Minas Gerais'
put 'cidade', '5', 'estatisticas:moradoresquantidade', '2521564'
put 'cidade', '5', 'responsaveis:prefeitonome', 'Alexandre Kalil'
put 'cidade', '5', 'responsaveis:prefeitopartido', 'PSD'
put 'cidade', '5', 'responsaveis:viceprefeitonome', 'Fuad Norman'
put 'cidade', '5', 'estatisticas:ultimaeleicaodata', '15/11/2020'
put 'cidade', '5', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '5', 'estatisticas:eleitoresquantidade', '1927460'
put 'cidade', '5', 'info:cidadefundacaodata', '12/12/1897'
put 'cidade', '5', 'estatisticas:cidadefundacaoano', '1897'
put 'cidade', '6', 'info:cidadenome', 'Manaus'
put 'cidade', '6', 'info:estadonome', 'Amazonas'
put 'cidade', '6', 'estatisticas:moradoresquantidade', '2219580'
put 'cidade', '6', 'responsaveis:prefeitonome', 'David Almeida'
put 'cidade', '6', 'responsaveis:prefeitopartido', 'Avante'
put 'cidade', '6', 'responsaveis:viceprefeitonome', 'Marcos Rotta'
put 'cidade', '6', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '6', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '6', 'estatisticas:eleitoresquantidade', '1257129'
put 'cidade', '6', 'info:cidadefundacaodata', '24/10/1669'
put 'cidade', '6', 'estatisticas:cidadefundacaoano', '1669'
put 'cidade', '7', 'info:cidadenome', 'Curitiba'
put 'cidade', '7', 'info:estadonome', 'Paraná'
put 'cidade', '7', 'estatisticas:moradoresquantidade', '1948626'
put 'cidade', '7', 'responsaveis:prefeitonome', 'Rafael Greca'
put 'cidade', '7', 'responsaveis:prefeitopartido', 'DEM'
put 'cidade', '7', 'responsaveis:viceprefeitonome', 'Eduardo Pimentel'
put 'cidade', '7', 'estatisticas:ultimaeleicaodata', '15/11/2020'
put 'cidade', '7', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '7', 'estatisticas:eleitoresquantidade', '1289215'
put 'cidade', '7', 'info:cidadefundacaodata', '29/3/1693'
put 'cidade', '7', 'estatisticas:cidadefundacaoano', '1693'
put 'cidade', '8', 'info:cidadenome', 'Recife'
put 'cidade', '8', 'info:estadonome', 'Pernambuco'
put 'cidade', '8', 'estatisticas:moradoresquantidade', '1653461'
put 'cidade', '8', 'responsaveis:prefeitonome', 'João Campos'
put 'cidade', '8', 'responsaveis:prefeitopartido', 'PSB'
put 'cidade', '8', 'responsaveis:viceprefeitonome', 'Isabella de Roldão'
put 'cidade', '8', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '8', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '8', 'estatisticas:eleitoresquantidade', '1119271'
put 'cidade', '8', 'info:cidadefundacaodata', '12/3/1537'
put 'cidade', '8', 'estatisticas:cidadefundacaoano', '1537'
put 'cidade', '9', 'info:cidadenome', 'Goiânia'
put 'cidade', '9', 'info:estadonome', 'Goiás'
put 'cidade', '9', 'estatisticas:moradoresquantidade', '1536097'
put 'cidade', '9', 'responsaveis:prefeitonome', 'Maguito Vilela'
put 'cidade', '9', 'responsaveis:prefeitopartido', 'MDB'
put 'cidade', '9', 'responsaveis:viceprefeitonome', 'Rogério Cruz'
put 'cidade', '9', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '9', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '9', 'estatisticas:eleitoresquantidade', '957159'
put 'cidade', '9', 'info:cidadefundacaodata', '24/10/1933'
put 'cidade', '9', 'estatisticas:cidadefundacaoano', '1933'
put 'cidade', '10', 'info:cidadenome', 'Belém'
put 'cidade', '10', 'info:estadonome', 'Pará'
put 'cidade', '10', 'estatisticas:moradoresquantidade', '1499641'
put 'cidade', '10', 'responsaveis:prefeitonome', 'Edmilson Rodrigues'
put 'cidade', '10', 'responsaveis:prefeitopartido', 'PSOL'
put 'cidade', '10', 'responsaveis:viceprefeitonome', 'Edilson Moura'
put 'cidade', '10', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '10', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '10', 'estatisticas:eleitoresquantidade', '1043219'
put 'cidade', '10', 'info:cidadefundacaodata', '12/1/1616'
put 'cidade', '10', 'estatisticas:cidadefundacaoano', '1616'
put 'cidade', '11', 'info:cidadenome', 'Porto Alegre'
put 'cidade', '11', 'info:estadonome', 'Rio Grande do Sul'
put 'cidade', '11', 'estatisticas:moradoresquantidade', '1488252'
put 'cidade', '11', 'responsaveis:prefeitonome', 'Sebastião Melo'
put 'cidade', '11', 'responsaveis:prefeitopartido', 'MDB'
put 'cidade', '11', 'responsaveis:viceprefeitonome', 'Ricardo Gomes'
put 'cidade', '11', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '11', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '11', 'estatisticas:eleitoresquantidade', '1098515'
put 'cidade', '11', 'info:cidadefundacaodata', '26/3/1772'
put 'cidade', '11', 'estatisticas:cidadefundacaoano', '1772'
put 'cidade', '12', 'info:cidadenome', 'São Luís'
put 'cidade', '12', 'info:estadonome', 'Maranhão'
put 'cidade', '12', 'estatisticas:moradoresquantidade', '1108975'
put 'cidade', '12', 'responsaveis:prefeitonome', 'Eduardo Braide'
put 'cidade', '12', 'responsaveis:prefeitopartido', 'PODE'
put 'cidade', '12', 'responsaveis:viceprefeitonome', 'Esmênia Miranda'
put 'cidade', '12', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '12', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '12', 'estatisticas:eleitoresquantidade', '659779'
put 'cidade', '12', 'info:cidadefundacaodata', '8/9/1612'
put 'cidade', '12', 'estatisticas:cidadefundacaoano', '1612'
put 'cidade', '13', 'info:cidadenome', 'Maceió'
put 'cidade', '13', 'info:estadonome', 'Alagoas'
put 'cidade', '13', 'estatisticas:moradoresquantidade', '1025360'
put 'cidade', '13', 'responsaveis:prefeitonome', 'João Henrique Caldas'
put 'cidade', '13', 'responsaveis:prefeitopartido', 'PSB'
put 'cidade', '13', 'responsaveis:viceprefeitonome', 'Ronaldo Lessa'
put 'cidade', '13', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '13', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '13', 'estatisticas:eleitoresquantidade', '579962'
put 'cidade', '13', 'info:cidadefundacaodata', '5/12/1815'
put 'cidade', '13', 'estatisticas:cidadefundacaoano', '1815'
put 'cidade', '14', 'info:cidadenome', 'Campo Grande'
put 'cidade', '14', 'info:estadonome', 'Mato Grosso do Sul'
put 'cidade', '14', 'estatisticas:moradoresquantidade', '906092'
put 'cidade', '14', 'responsaveis:prefeitonome', 'Marquinhos Trad'
put 'cidade', '14', 'responsaveis:prefeitopartido', 'PSD'
put 'cidade', '14', 'responsaveis:viceprefeitonome', 'Adriane Lopes'
put 'cidade', '14', 'estatisticas:ultimaeleicaodata', '15/11/2020'
put 'cidade', '14', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '14', 'estatisticas:eleitoresquantidade', '595174'
put 'cidade', '14', 'info:cidadefundacaodata', '21/6/1872'
put 'cidade', '14', 'estatisticas:cidadefundacaoano', '1872'
put 'cidade', '15', 'info:cidadenome', 'Natal'
put 'cidade', '15', 'info:estadonome', 'Rio Grande do Norte'
put 'cidade', '15', 'estatisticas:moradoresquantidade', '890480'
put 'cidade', '15', 'responsaveis:prefeitonome', 'Álvaro Dias'
put 'cidade', '15', 'responsaveis:prefeitopartido', 'PSDB'
put 'cidade', '15', 'responsaveis:viceprefeitonome', 'Aíla Cortez'
put 'cidade', '15', 'estatisticas:ultimaeleicaodata', '15/11/2020'
put 'cidade', '15', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '15', 'estatisticas:eleitoresquantidade', '534582'
put 'cidade', '15', 'info:cidadefundacaodata', '25/12/1599'
put 'cidade', '15', 'estatisticas:cidadefundacaoano', '1599'
put 'cidade', '16', 'info:cidadenome', 'Teresina'
put 'cidade', '16', 'info:estadonome', 'Piauí'
put 'cidade', '16', 'estatisticas:moradoresquantidade', '868075'
put 'cidade', '16', 'responsaveis:prefeitonome', 'José Pessoa Leal'
put 'cidade', '16', 'responsaveis:prefeitopartido', 'MDB'
put 'cidade', '16', 'responsaveis:viceprefeitonome', 'Robert Rios'
put 'cidade', '16', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '16', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '16', 'estatisticas:eleitoresquantidade', '531953'
put 'cidade', '16', 'info:cidadefundacaodata', '16/8/1852'
put 'cidade', '16', 'estatisticas:cidadefundacaoano', '1852'
put 'cidade', '17', 'info:cidadenome', 'João Pessoa'
put 'cidade', '17', 'info:estadonome', 'Paraíba'
put 'cidade', '17', 'estatisticas:moradoresquantidade', '817511'
put 'cidade', '17', 'responsaveis:prefeitonome', 'Cícero Lucena'
put 'cidade', '17', 'responsaveis:prefeitopartido', 'PP'
put 'cidade', '17', 'responsaveis:viceprefeitonome', 'Léo Bezerra'
put 'cidade', '17', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '17', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '17', 'estatisticas:eleitoresquantidade', '489028'
put 'cidade', '17', 'info:cidadefundacaodata', '5/8/1585'
put 'cidade', '17', 'estatisticas:cidadefundacaoano', '1585'
put 'cidade', '18', 'info:cidadenome', 'Aracaju'
put 'cidade', '18', 'info:estadonome', 'Sergipe'
put 'cidade', '18', 'estatisticas:moradoresquantidade', '664908'
put 'cidade', '18', 'responsaveis:prefeitonome', 'Edvaldo Nogueira'
put 'cidade', '18', 'responsaveis:prefeitopartido', 'PDT'
put 'cidade', '18', 'responsaveis:viceprefeitonome', 'Katarina Feitosa'
put 'cidade', '18', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '18', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '18', 'estatisticas:eleitoresquantidade', '397228'
put 'cidade', '18', 'info:cidadefundacaodata', '17/3/1855'
put 'cidade', '18', 'estatisticas:cidadefundacaoano', '1855'
put 'cidade', '19', 'info:cidadenome', 'Cuiabá'
put 'cidade', '19', 'info:estadonome', 'Mato Grosso'
put 'cidade', '19', 'estatisticas:moradoresquantidade', '618124'
put 'cidade', '19', 'responsaveis:prefeitonome', 'Emanuel Pinheiro'
put 'cidade', '19', 'responsaveis:prefeitopartido', 'MDB'
put 'cidade', '19', 'responsaveis:viceprefeitonome', 'Roberto Stopa'
put 'cidade', '19', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '19', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '19', 'estatisticas:eleitoresquantidade', '415100'
put 'cidade', '19', 'info:cidadefundacaodata', '8/4/1719'
put 'cidade', '19', 'estatisticas:cidadefundacaoano', '1719'
put 'cidade', '20', 'info:cidadenome', 'Porto Velho'
put 'cidade', '20', 'info:estadonome', 'Rondônia'
put 'cidade', '20', 'estatisticas:moradoresquantidade', '539354'
put 'cidade', '20', 'responsaveis:prefeitonome', 'Hildon Chaves'
put 'cidade', '20', 'responsaveis:prefeitopartido', 'PSDB'
put 'cidade', '20', 'responsaveis:viceprefeitonome', 'Mauricio Carvalho'
put 'cidade', '20', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '20', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '20', 'estatisticas:eleitoresquantidade', '319939'
put 'cidade', '20', 'info:cidadefundacaodata', '2/10/1907'
put 'cidade', '20', 'estatisticas:cidadefundacaoano', '1907'
put 'cidade', '21', 'info:cidadenome', 'Macapá'
put 'cidade', '21', 'info:estadonome', 'Amapá'
put 'cidade', '21', 'estatisticas:moradoresquantidade', '512902'
put 'cidade', '21', 'responsaveis:prefeitonome', 'Antônio Furlan'
put 'cidade', '21', 'responsaveis:prefeitopartido', 'Cidadania'
put 'cidade', '21', 'responsaveis:viceprefeitonome', 'Monica Penha'
put 'cidade', '21', 'estatisticas:ultimaeleicaodata', '20/12/2020'
put 'cidade', '21', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '21', 'estatisticas:eleitoresquantidade', '277688'
put 'cidade', '21', 'info:cidadefundacaodata', '4/2/1758'
put 'cidade', '21', 'estatisticas:cidadefundacaoano', '1758'
put 'cidade', '22', 'info:cidadenome', 'Florianópolis'
put 'cidade', '22', 'info:estadonome', 'Santa Catarina'
put 'cidade', '22', 'estatisticas:moradoresquantidade', '508826'
put 'cidade', '22', 'responsaveis:prefeitonome', 'Gean Loureiro'
put 'cidade', '22', 'responsaveis:prefeitopartido', 'DEM'
put 'cidade', '22', 'responsaveis:viceprefeitonome', 'Topázio Neto'
put 'cidade', '22', 'estatisticas:ultimaeleicaodata', '15/11/2020'
put 'cidade', '22', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '22', 'estatisticas:eleitoresquantidade', '316261'
put 'cidade', '22', 'info:cidadefundacaodata', '23/3/1673'
put 'cidade', '22', 'estatisticas:cidadefundacaoano', '1673'
put 'cidade', '23', 'info:cidadenome', 'Boa Vista'
put 'cidade', '23', 'info:estadonome', 'Roraima'
put 'cidade', '23', 'estatisticas:moradoresquantidade', '419652'
put 'cidade', '23', 'responsaveis:prefeitonome', 'Arthur Henrique'
put 'cidade', '23', 'responsaveis:prefeitopartido', 'MDB'
put 'cidade', '23', 'responsaveis:viceprefeitonome', 'Cássio Gomes'
put 'cidade', '23', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '23', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '23', 'estatisticas:eleitoresquantidade', '203575'
put 'cidade', '23', 'info:cidadefundacaodata', '9/7/1890'
put 'cidade', '23', 'estatisticas:cidadefundacaoano', '1890'
put 'cidade', '24', 'info:cidadenome', 'Rio Branco'
put 'cidade', '24', 'info:estadonome', 'Acre'
put 'cidade', '24', 'estatisticas:moradoresquantidade', '413418'
put 'cidade', '24', 'responsaveis:prefeitonome', 'Tião Bocalom'
put 'cidade', '24', 'responsaveis:prefeitopartido', 'PP'
put 'cidade', '24', 'responsaveis:viceprefeitonome', 'Marfiza Galvão'
put 'cidade', '24', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '24', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '24', 'estatisticas:eleitoresquantidade', '241196'
put 'cidade', '24', 'info:cidadefundacaodata', '28/12/1882'
put 'cidade', '24', 'estatisticas:cidadefundacaoano', '1882'
put 'cidade', '25', 'info:cidadenome', 'Vitória'
put 'cidade', '25', 'info:estadonome', 'Espírito Santo'
put 'cidade', '25', 'estatisticas:moradoresquantidade', '365855'
put 'cidade', '25', 'responsaveis:prefeitonome', 'Lorenzo Pazolini'
put 'cidade', '25', 'responsaveis:prefeitopartido', 'Republicanos'
put 'cidade', '25', 'responsaveis:viceprefeitonome', 'Estéfane da Silva'
put 'cidade', '25', 'estatisticas:ultimaeleicaodata', '29/11/2020'
put 'cidade', '25', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '25', 'estatisticas:eleitoresquantidade', '232829'
put 'cidade', '25', 'info:cidadefundacaodata', '8/9/1551'
put 'cidade', '25', 'estatisticas:cidadefundacaoano', '1551'
put 'cidade', '26', 'info:cidadenome', 'Palmas'
put 'cidade', '26', 'info:estadonome', 'Tocantins'
put 'cidade', '26', 'estatisticas:moradoresquantidade', '306296'
put 'cidade', '26', 'responsaveis:prefeitonome', 'Cinthia Ribeiro'
put 'cidade', '26', 'responsaveis:prefeitopartido', 'PSDB'
put 'cidade', '26', 'responsaveis:viceprefeitonome', 'Lucas Meira'
put 'cidade', '26', 'estatisticas:ultimaeleicaodata', '15/11/2020'
put 'cidade', '26', 'responsaveis:prefeitopossedata', '01/01/2021'
put 'cidade', '26', 'estatisticas:eleitoresquantidade', '172344'
put 'cidade', '26', 'info:cidadefundacaodata', '20/5/1989'
put 'cidade', '26', 'estatisticas:cidadefundacaoano', '1989'
~~~
   
3. Realizar uma contagem de linhas na tabela.
~~~shell
count 'cidade'
~~~
   
4. Consultar só o código e nome da cidade.
~~~shell
get 'cidade', '1', 'info:cidadenome'
~~~
   
5. Escolha uma cidade, consulte os dados dessa cidade em específico antes do próximo passo.
~~~shell
get 'cidade', '1'
~~~
   
6. Altere para a cidade escolhida os dados de Prefeito, Vice Prefeito e nova data de Posse.
~~~shell
put 'cidade', '1', 'responsaveis:prefeitonome', 'Guilherme Boulos'
put 'cidade', '1', 'responsaveis:viceprefeitonome', 'Luiza Erundina'
put 'cidade', '1', 'responsaveis:prefeitopossedata', '01/01/2025'
~~~
   
7. Consulte os dados da cidade alterada.
~~~shell
get 'cidade', '1'
~~~
   
8. Consulte todas as versões dos dados da cidade alterada.
~~~shell
get 'cidade', '1', {COLUMNS=>['responsaveis'],VERSIONS=>5}
~~~

9. Exclua as três cidades com menor quantidade de habitantes e quantidade de eleitores.
~~~shell
#TODO check how do select min inside hbase

g = scan 'cidade', 'estatisticas:eleitoresquantidade', {FILTER => MIN, LIMIT 3}

delete 'cidade', '1', 'estatisticas:eleitoresquantidade'

~~~
   
10. Liste todas as cidades novamente.
~~~shell
scan 'cidade'
~~~
    
11. Adicione na ColumnFamily “estatísticas”, duas novas colunas de “quantidade de partidos políticos” e “Valor em Reais à partidos” para as 2 cidades mais populosas cadastradas.
~~~shell
#TODO check how do select max inside hbase
put 'cidade', '1', 'responsaveis:qtdepartidos', '33'
put 'cidade', '1', 'responsaveis:repassepartidos', '100000000'
~~~
    
12. Liste novamente todas as cidades.
~~~shell
scan 'cidade'
~~~

### 3 - Operações massivas
3.1 Exemplo
~~~shell
#1. Confirmar que o arquivo employees.csv esteja em /home/everis/employee.csv
#2. Criar a tabela employees no Hbase com a column Familty: employee_data
create 'employees', 'employee_data'
#3. Criar uma pasta no HDFS pelo shell do Linux
hadoop fs -mkdir /test
#4. Copia os arquivos exportados para o HDFS pelo shell do Linux
hadoop fs -copyFromLocal /home/everis/employees.csv /test/employees.csv
#5. Executar a importação no shell do Linux
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=';' -Dimporttsv.columns=HBASE_ROW_KEY,employee_data:birth_date,employee_data:first_name,employee_data:last_name,employee_data:gender,employee_data:hire_date employees/test/employees.csv
~~~

3.2 Atividades

### 4 - Integrações NoSQL com ambiente Hadoop
~~~shell
#1. Vamos criar um novo schema no Hive.
CREATE DATABASE tabelas_hbases;

#2. Criar a tabela employee no Hive.
CREATE EXTERNAL TABLE tabelas_hbases.employees (
    emp_no INT,
    birth_date string,
    first_name string,
    last_name string,
    gender string,
    hire_date string)
    STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
    WITH SERDEPROPERTIES("hbase.columns.mapping"=":key, employee_data:birth_date, employee_data:first_name, employee_data:last_name,employee_data:gender,employee_data:hire_date")
    TBLPROPERTIES("hbase.table.name"="employees", "hbase.mapred.output.outputtable"="employees");
~~~

### Material
[Máquina Virtual utilizada](https://hermes.digitalinnovation.one/files/acceleration/Everis_BigData-v3.ova) <br>
[Slide da aula](https://drive.google.com/file/d/1nfzVpbwwB8d99RY8BEvjaQJ-vYx6o56X/view)
