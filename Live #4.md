# Live #4 - Como realizar consultas de maneira simples no ambiente complexo de Big Data com HIVE e Impala

**[Apache Hive](https://www.dezyre.com/article/impala-vs-hive-difference-between-sql-on-hadoop-components/180)** é uma infraestrutura de data warehouse construída sobre a plataforma Hadoop para realizar tarefas intensivas de dados, como consulta, análise, processamento e visualização. <br>
**[Apache Impala](https://medium.com/@markuvinicius/deep-diving-into-hadoop-world-hive-e-impala-11b1dca716ba)** traz uma tecnologia de banco de dados paralela e escalável para o Hadoop, permitindo que os usuários emitam consultas SQL de baixa latência a dados armazenados no HDFS ou HBase sem necessidade de movimentar ou transformar os dados. <br>
**[HDFS Shell](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html#copyToLocal)** <br>

* Hive, Impala ~ SQL Big Data (MapReduce)
* Hive (Facebook, intermediate queries, less memory), Impala (Cloudera, fast)
* HDFS: folder=table (managed (default): drop HDFS, removes, só ali; external (metadata~pointer): drop HDFS, keeps)
* Parquet (Cloudera,Twitter): compressed,column: Impala, Spark; Avro: Kafka; Orc: Hive
* Snappy: compress codec


Profº [**Vinicius Bueno**](https://www.linkedin.com/in/vinicius-m-bueno-br/) <br>

### 1 - Iniciando os serviços que serão utilizados
~~~shell

# Download and move scripts to proper folder
wget https://raw.githubusercontent.com/viniciusriosfuck/Aceleracao_Global_Dev4_Everis/master/script_apoio/restart_all_service_hive.sh
mv restart_all_service_hive.sh script_apoio/
wget https://raw.githubusercontent.com/viniciusriosfuck/Aceleracao_Global_Dev4_Everis/master/script_apoio/sqoop_import.sh
mv sqoop_import.sh script_apoio/

# Download used data files, first layer, raw data as strings, market proxy
wget https://raw.githubusercontent.com/viniciusriosfuck/Aceleracao_Global_Dev4_Everis/master/arquivos_apoio/employees.csv
wget https://raw.githubusercontent.com/viniciusriosfuck/Aceleracao_Global_Dev4_Everis/master/arquivos_apoio/base_localidade.csv

#Utilizando script (restart_all_service_hive.sh)
sh script_apoio/restart_all_service_hive.sh
# Leave safe_mode
sudo -u hdfs hdfs dfsadmin -safemode leave

~~~

### 2 - Acessando Hive e Impala
~~~shell
#Impala
impala-shell

# to quit from the both
quit;
ctrl+c

#Hive
hive

#!: comando bash dentro do Hive/Impala (; at end of line)

~~~

### 3 - Manipulando bases de dados no Hive
~~~shell

#Exibindo as bases de dados
show databases;

#Personalizando terminal para exibir banco que está sendo usado
set hive.cli.print.current.db=true;

#Exibindo as tabelas
show tables;

#Criando base de dados
create database teste01;

#Criar base de dados caso já exista com o mesmo nome
create database if not exists teste01;

#Acessando uma base de dados
use teste01;

#Exibindo as tabelas
show tables;

#Criando uma tabela
##Fora do banco de dados
create table teste01.teste01 (id int);
create table if not exists teste01.teste01 (id int);

##Usando o banco de dados
create table teste02 (id int);

#Exibindo informações de criação das tabelas
show create table teste01;
 
#Personalizando terminal para exibir o header
set hive.cli.print.header=true;

#Inserindo dados na tabela
insert into table teste01 values(1);

#Diretório default do Hive
!hdfs dfs -ls /user/hive/warehouse;

#Acessando arquivos no diretório default do hive.
##Base de dados
!hdfs dfs -ls /user/hive/warehouse/teste01.db;

##Tabela
!hdfs dfs -ls /user/hive/warehouse/teste01.db/teste01;

#Tabelas externas
!hdfs dfs -ls /user/hive/warehouse/external/tabelas;

#Criando tabela do tipo externa
create external table teste03 (id int);
~~~

4 - Ingestão de base de dados
~~~shell
#Criando tabela no Hive
CREATE EXTERNAL TABLE TB_EXT_EMPLOYEE (
      id STRING,
      groups STRING,
      age STRING,
      active_lifestyle STRING,
      salary STRING)
      ROW FORMAT DELIMITED FIELDS
      TERMINATED BY '\;'
      STORED AS TEXTFILE
      LOCATION '/user/hive/warehouse/external/tabelas/employee'
      tblproperties ("skip.header.line.count"="1");
      
#Carregando base de dados em arquivo para o HDFS usando -PUT
!hdfs dfs -put /home/everis/employees.csv /user/hive/warehouse/external/tabelas/employee;
!hdfs dfs -put -f /home/everis/employees.csv /user/hive/warehouse/external/tabelas/employee;
#-f: if not exists (overwrite)

#Criando tabela formatada
CREATE TABLE TB_EMPLOYEE(
      id INT,
      groups STRING,
      age INT,
      active_lifestyle STRING,
      salary DOUBLE)
      PARTITIONED BY (dt_processamento STRING)
      ROW FORMAT DELIMITED FIELDS TERMINATED BY '|'
      STORED AS PARQUET TBLPROPERTIES ("parquet.compression"="SNAPPY");
#Coluna do tipo partição.
#Armazenando em formato parquet Snappy
      
#Inserindo dados de outra tabela na tabela formatada
INSERT INTO TABLE TB_EMPLOYEE partition (dt_processamento='20201118')
      select 
      id,
      groups,
      age,
      active_lifestyle,
      salary
      FROM TB_EXT_EMPLOYEE;
~~~

5 - Visualizando arquivo do tipo parquet
~~~shell
#Copiar arquivo para o disco local
!rm 000000_0;
!hdfs dfs -copyToLocal /user/hive/warehouse/teste01.db/tb_employee/dt_processamento=20201118/000000_0 .;

#Visualizar info do schema utilizando o parquet-tools
!parquet-tools schema 000000_0;
~~~

6 - Ingestão de base de dados II
~~~shell
#Criando tabela
CREATE EXTERNAL TABLE localidade(
      street string,
      city string,
      zip string,
      state string,
      beds string,
      baths string,
      sq__ft string,
      type string,
      sale_date string,
      price string,
      latitude string,
      longitude string)
      PARTITIONED BY (particao STRING)
      ROW FORMAT DELIMITED FIELDS TERMINATED BY ","
      STORED AS TEXTFILE
      location '/user/hive/warehouse/external/tabelas/localidade'
      tblproperties ("skip.header.line.count"="1");
      
#Carga na tabela pelo Hive
load data local inpath '/home/everis/base_localidade.csv'
      INTO TABLE teste01.localidade partition (particao='2021-01-21');

#Visualizando dados da tabela no Hive
SELECT * FROM localidade LIMIT 10;

#Visualizando arquivo no HDFS
!hdfs dfs -ls /user/hive/warehouse/external/tabelas/localidade/particao=2021-01-21;

#Visualizando conteúdo do arquivo no HDFS
!hdfs dfs -cat /user/hive/warehouse/external/tabelas/localidade/particao=2021-01-21/base_localidade.csv;
!clear;
ctrl+L (clears terminal)

#Criando tabela formatada
CREATE TABLE tb_localidade_parquet(
      street string,
      city string,
      zip string,
      state string,
      beds string,
      baths string,
      sq__ft string,
      type string,
      sale_date string,
      price string,
      latitude string,
      longitude string)
      PARTITIONED BY (particao STRING)
      STORED AS PARQUET;

#Inserindo dados via partição na tabela formatada
INSERT INTO TABLE tb_localidade_parquet
      PARTITION(PARTICAO='01')
      SELECT
      street,
      city,
      zip,
      state,
      beds,
      baths,
      sq__ft,
      type,
      sale_date,
      price,
      latitude,
      longitude
      FROM localidade;  
	  
#Exemplo sem relação com as tabelas disponíveis durante a aula
SELECT
      tab01.id,
      tab02.zip
      FROM tb_ext_employee tab01
      FULL OUTER JOIN tb_localidade_parquet tab02
      ON tab01.id = tab02.zip;

#Select com coluna com agrupamento de informações
SELECT
      tab01.id,
      tab02.zip,
      "teste" col_fixa,
      concat(tab01.id, tab02.zip) AS col_concatenada
      FROM tb_ext_employee tab01
      FULL OUTER JOIN tb_localidade_parquet tab02
      ON tab01.id = tab02.zip;

# Leave hive
ctrl+c or quit;
~~~

7 - Impala
~~~shell
#Acessando Apache Impala
impala-shell

#Exibindo bases de dados;
SHOW databases;

#Exibindo bases de dados;
SHOW tables;

#Atualizando bases de dados. Necessário atualizar o metastore com a base de dados e tabela que deseja invalidar.
INVALIDATE METADATA teste01.tb_localidade_parquet;
INVALIDATE METADATA teste01.tb_ext_employee;

# Query
SELECT * FROM teste01.tb_ext_employee LIMIT 10;
~~~

### Exemplo de Script
(TODO): see how to adapt this script
~~~shell
#!/bin/bash

dt_processamento=$(date '+%Y-%m-%d')
path_file='/home/cloudera/hive/datasets/employee.txt'
table=beca.ext_p_employee
load=/home/cloudera/hive/load.hql

hive -hiveconf dt_processamento=${dt_processamento} -hiveconf table=${table} -hiveconf path_file=${path_file} -f $load 2>> log.txt

hive_status=$?

if [ ${hive_status} -eq 0 ];
then
        echo -e "\nScript executado com sucesso"
else
        echo -e "\nHouve um erro na ingestao do arquivo"
impala-shell -q 'INVALIDATE METADATA beca.ext_p_employee;'

fi 

LOAD DATA LOCAL INPATH '${hiveconf:path_file}' INTO TABLE ${hiveconf:table} PARTITION(dt_processamento='${hiveconf:dt_processamento}');
~~~

### Comandos adicionais
~~~shell
#Acessar informações de comandos do Hadoop
hadoop -h

#Acessar manual do Hadoop
man hadoop

#Acessar manual do HDFS
man hdfs

#Acessar informações de comandos do HDFS
hdfs -h

#Acessar informações de comandos do HIVE
hive -h

#Exemplo de execução do Hive sem acessá-lo
hive -S -e "SELECT COUNT(*) FROM teste01.localidade;"
# -S: silent mode
# -e: SQL

# fix backspace key issue (when press backspace key print ^H in the terminal)
stty erase ^H
# https://unix.stackexchange.com/questions/43103/backspace-tab-not-working-in-terminal-using-ssh

~~~

### Material
[Máquina Virtual utilizada](https://hermes.digitalinnovation.one/files/acceleration/Everis_BigData-v3.ova) <br>
[Slide da aula](https://drive.google.com/file/d/1duwf7g9lfAsIOJWRL26tx8RBfV8ySPon/view?usp=sharing) <br>
