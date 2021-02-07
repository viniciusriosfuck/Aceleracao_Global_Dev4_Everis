# Live #3 - Orquestrando ambientes de big data distribuídos com Zookeeper, Yarn e Sqoop

**[Zookeeper](https://www.cetax.com.br/apache-hadoop-tudo-o-que-voce-precisa-saber/)** é um serviço de coordenação distribuída para gerenciar grandes conjuntos de Clusters. <br>
**[Yarn](https://pt.wikipedia.org/wiki/Hadoop)** é uma plataforma do ecossistema Hadoop para gerenciamento de recursos responsável pelo gerenciamento dos recursos computacionais em cluster, assim como pelo agendamento dos recursos. <br>
**[Sqoop](https://www.cetax.com.br/apache-hadoop-tudo-o-que-voce-precisa-saber/)** é um projeto do ecossistema Hadoop, cuja responsabilidade é importar e exportar dados do banco de dados relacionais. <br>

* Zookeeper: coordenação distribuído, nós (hosts), rotas, ~DNS; Leader, Followers <br>
* Sqoop: RDBS <-> HDFS (csv) <br>

Profº [**Rodrigo Garcia**](https://www.linkedin.com/in/rodsantosg/) <br>

### 1 - Alguns exemplos:
~~~shell
# Exemplo 1
sqoop import \
--connect jdbc:mysql://mysql.example.com/sqoop \
--username sqoop \
--password sqoop \
--table cities
--warehouse-dir /etl/input/ # Pemite especificar um diretório no HDFS como destino
--where "country = 'Brazil'" # Para importar apenas um subconjunto de registros de uma tabela
-P ou --password-file my-sqoop-password
--as-sequencefile ou --as-avrodatafile # Para escrever o arquivo no HDFS em formato binário (Sequence ou Avro)
--compress # Comprime os blocos antes de gravar no HDFS em formato gzip por padrão
--compression-codec # Utilizar outros codecs de compressão, exemplo: org.apache.hadoop.io.compress.BZip2codec
--direct # Realiza import direto por meio das funcionalidades nativas do BD para melhorar a performance, exemplo: mysqldump ou pg_dump
--map-column-java c1=String # Especificar o tipo do campo
--num-mappers 10 # Especificar a quantidade de paralelismo para controlar o workload
--null-string '\\N'\
--null-non-string '\\N'
--incremental append ou lastmodified # Funcionalidade para incrementar os dados
			--check-column id ou last_update_date # Identifica a coluna que será verificada para incrementar novos dados
			--last-value 1 ou "2013-05-22 01:01:01" # Para especificar o último valor importado no Hadoop
			
# Exemplo 2 - Import da tabela accounts
sqoop import --table accounts \
--connect jdbc:mysql://dbhost/loudacre \
--username dbuser --password pw

# Exemplo 3 - Importa da tabela accounts utilizando um delimitador
sqoop import --table accounts \
--connect jdbc:mysql://dbhost/loudacre \
--username dbuser --password pw \
--fields-terminated-by "\t"

# Exemplo 4 - Import da tabela accounts limitando os resultados
sqoop import --table accounts \
--connect jdbc:mysql://dbhost/loudacre \
--username dbuser --password pw \
--where "state='CA'"

# Exemplo 5 - Import incremental baseado em um timestamp. Deve certificar-se de que esta coluna é atualizada quando os registros são atualizados ou adicionados
sqoop import --table invoices \
--connect jdbc:mysql://dbhost/loudacre \
--username dbuser --password pw \
--incremental lasmodified \
--check-column mod_dt \
--last-value '2015-09-30 16:00:00'

# Exemplo 6 - Import baseado no último valor de uma coluna específica
sqoop import --table invoices \
--connect jdbc:mysql://dbhost/loudacre \
--username dbuser --password pw \
--incremental append \
--check-column id \
--last-value 9878306
~~~

### 2 - Instalando o Sqoop
~~~shell
sudo yum install --assumeyes sqoop # Instalando
cd /tmp # Abre a pasta tmp
wget http://www.java2s.com/Code/JarDownload/java-json/java-json.jar.zip # Pega o arquivo zipado
unzip /tmp/java-json.jar.zip # Descompacta o arquivo
sudo mv /tmp/java-json.jar /usr/lib/sqoop/lib/ # Move o json para a lib do sqoop
sudo chown root: /usr/lib/sqoop/lib/java-json.jar # Altera o dono para o root
sqoop-version
~~~

### 3 - Criando base de dados de exemplo no Mysql
~~~shell
# Foi utilizado uma base do Pokemon. Base disponível no arquivo pokemon.sql
wget https://raw.githubusercontent.com/viniciusriosfuck/Aceleracao_Global_Dev4_Everis/master/arquivos_apoio/pokemon.sql
mysql -u root -h localhost -pEveris@2021 < pokemon.sql
~~~

### 4 - Iniciando serviços utilizados
~~~shell
sudo service hadoop-hdfs-namenode start
sudo service hadoop-hdfs-secondarynamenode start
sudo service hadoop-hdfs-datanode start
sudo service hadoop-yarn-nodemanager start
sudo service hadoop-yarn-resourcemanager start
sudo service hadoop-mapreduce-historyserver start
sudo service zookeeper-server start
~~~

### 5 - Importando base de dados Mysql para o HDFS
~~~shell
sh sqoop_import.sh
~~~

### 6 - Manipulando arquivos no HDFS
~~~shell
# Listando arquivos
hdfs dfs -ls /user/everis-bigdata/pokemon/

# Lendo um arquivo
hdfs dfs -text /user/everis-bigdata/pokemon/part-m-00000.gz | more
~~~

### 7 - Atividades
~~~shell

# mysql
# Abre mysql
mysql -u root -h localhost -pEveris@2021
SELECT Number, Name, Legendary FROM trainning.pokemon WHERE Legendary IS true; #65
SELECT Number, Name, Type1, Type2 FROM trainning.pokemon WHERE Type2 = ""; #386
SELECT Number, Name, Speed FROM trainning.pokemon ORDER BY speed DESC LIMIT 10;
SELECT Number, Name, hp FROM trainning.pokemon ORDER BY hp LIMIT 50;
SELECT Number, Name, SUM(HP+Attack+Defense+SpAtk+SpDef+Speed) AS Total FROM trainning.pokemon GROUP BY Number ORDER BY Total DESC LIMIT 100;
quit

#1 - Todos os Pokemons lendários;
sudo -u hdfs sqoop import \ # Utilizando o usuário HDFS
--connect jdbc:mysql://localhost/trainning \ # String de conexão
--username root --password "Everis@2021" \ # Login
--direct \ # Executar no BD
--table pokemon \ # Terminando tabela
--target-dir /user/everis-bigdata/pokemon/1 \ # Onde será salvo
--where "Legendary=1" # Condição

# mysql
SELECT Number, Name, Legendary FROM trainning.pokemon WHERE Legendary IS true; #65
+--------+---------------------------+-----------+
| Number | Name                      | Legendary |
+--------+---------------------------+-----------+
|    157 | Articuno                  |         1 |
|    158 | Zapdos                    |         1 |
|    159 | Moltres                   |         1 |
|    163 | Mewtwo                    |         1 |
|    164 | Mega Mewtwo X             |         1 |
|    165 | Mega Mewtwo Y             |         1 |
|    263 | Raikou                    |         1 |
65 rows in set (0.00 sec)

# hdfs
sudo -u hdfs sqoop import \
--connect jdbc:mysql://localhost/trainning \
--username root --password "Everis@2021" \
--target-dir /user/everis-bigdata/pokemon/1 \
--direct \
--table pokemon \
--where "Legendary=1"

hdfs dfs -text /user/everis-bigdata/pokemon/1/*
# 157,Articuno,Ice,Flying,90,85,100,95,125,85,1,1
# 158,Zapdos,Electric,Flying,90,90,85,125,90,100,1,1
# 159,Moltres,Fire,Flying,90,100,90,125,85,90,1,1
# 163,Mewtwo,Psychic,,106,110,90,154,90,130,1,1

# número de linhas: 65
hdfs dfs -cat /user/everis-bigdata/pokemon/1/* |wc -l

#2 - Todos os Pokemons de apenas um tipo;
# mysql
SELECT Number, Name, Type1, Type2 FROM trainning.pokemon WHERE Type1 = "";
# Empty set
SELECT Number, Name, Type1, Type2 FROM trainning.pokemon WHERE Type2 = ""; #386
+--------+--------------------------+----------+-------+
| Number | Name                     | Type1    | Type2 |
+--------+--------------------------+----------+-------+
|      5 | Charmander               | Fire     |       |
|      6 | Charmeleon               | Fire     |       |
|     10 | Squirtle                 | Water    |       |
|     11 | Wartortle                | Water    |       |
|     12 | Blastoise                | Water    |       |
|     13 | Mega Blastoise           | Water    |       |
|     14 | Caterpie                 | Bug      |       |
|     15 | Metapod                  | Bug      |       |
|     25 | Rattata                  | Normal   |       |
386 rows in set (0.00 sec)

#hdfs
sudo -u hdfs sqoop import \
--connect jdbc:mysql://localhost/trainning \
--username root --password "Everis@2021" \
--direct \
--table pokemon \
--target-dir /user/everis-bigdata/pokemon/2 \
--where "Type2=''"

hdfs dfs -text /user/everis-bigdata/pokemon/2/*
5,Charmander,Fire,,39,52,43,60,50,65,1,0
6,Charmeleon,Fire,,58,64,58,80,65,80,1,0
10,Squirtle,Water,,44,48,65,50,64,43,1,0
11,Wartortle,Water,,59,63,80,65,80,58,1,0
12,Blastoise,Water,,79,83,100,85,105,78,1,0
13,Mega Blastoise,Water,,79,103,120,135,115,78,1,0
14,Caterpie,Bug,,45,30,35,20,20,45,1,0
15,Metapod,Bug,,50,20,55,25,25,30,1,0
25,Rattata,Normal,,30,56,35,25,35,72,1,0

# número de linhas: 386
hdfs dfs -cat /user/everis-bigdata/pokemon/2/* |wc -l

#3 - Os top 10 Pokemons mais rápidos;
# mysql
SELECT Number, Name, Speed FROM trainning.pokemon ORDER BY speed DESC LIMIT 10;
+--------+---------------------+-------+
| Number | Name                | speed |
+--------+---------------------+-------+
|    432 | Deoxys Speed Forme  |   180 |
|    316 | Ninjask             |   160 |
|    430 | DeoxysAttack Forme  |   150 |
|    155 | Mega Aerodactyl     |   150 |
|    429 | Deoxys Normal Forme |   150 |
|     72 | Mega Alakazam       |   150 |
|    276 | Mega Sceptile       |   145 |
|     20 | Mega Beedrill       |   145 |
|    679 | Accelgor            |   145 |
|    110 | Electrode           |   140 |
+--------+---------------------+-------+

#hdfs
sudo -u hdfs sqoop import \
--connect jdbc:mysql://localhost/trainning \
--username root --password "Everis@2021" \
--fields-terminated-by "|" \
--split-by 1 \
--target-dir /user/everis-bigdata/pokemon/3 \
--query 'SELECT Number, Name, Speed FROM pokemon WHERE $CONDITIONS ORDER BY SPEED DESC LIMIT 10' 

hdfs dfs -text /user/everis-bigdata/pokemon/3/*
#432|Deoxys Speed Forme|180
#316|Ninjask|160
#430|DeoxysAttack Forme|150
#155|Mega Aerodactyl|150
#429|Deoxys Normal Forme|150
#72|Mega Alakazam|150
#276|Mega Sceptile|145
#20|Mega Beedrill|145
#679|Accelgor|145
#110|Electrode|140

#4 - Os top 50 Pokemons com menos HP;
# mysql
SELECT Number, Name, hp FROM trainning.pokemon ORDER BY hp LIMIT 50;
+--------+------------+------+
| Number | Name       | hp   |
+--------+------------+------+
|    317 | Shedinja   |    1 |
|     56 | Diglett    |   10 |
|    187 | Pichu      |   20 |
|    389 | Duskull    |   20 |
|    140 | Magikarp   |   20 |
|    488 | Mime Jr.   |   20 |
|    382 | Feebas     |   20 |
|    231 | Shuckle    |   20 |
|     69 | Abra       |   25 |
|     89 | Magnemite  |   25 |
|    304 | Ralts      |   28 |

# hdfs
sudo -u hdfs sqoop import \
--connect jdbc:mysql://localhost/trainning \
--username root --password "Everis@2021" \
--fields-terminated-by "|" \
--split-by 1 \
--target-dir /user/everis-bigdata/pokemon/4 \
--query 'SELECT Number, Name, hp FROM pokemon WHERE $CONDITIONS ORDER BY HP ASC LIMIT 50' 

hdfs dfs -text /user/everis-bigdata/pokemon/4/*
#317|Shedinja|1
#56|Diglett|10
#187|Pichu|20
#389|Duskull|20
#140|Magikarp|20
#488|Mime Jr.|20
#382|Feebas|20
#231|Shuckle|20
#69|Abra|25
#89|Magnemite|25
#304|Ralts|28

#5 - Os top 100 Pokemon com maiores atributos
sudo -u hdfs sqoop import \
--connect jdbc:mysql://localhost/trainning \
--username root --password "Everis@2021" \
--fields-terminated-by "|" \
--split-by 1 \
--target-dir /user/everis-bigdata/pokemon/5 \
--query 'SELECT Number, Name, SUM(HP+Attack+Defense+SpAtk+SpDef+Speed) AS Total FROM pokemon WHERE $CONDITIONS GROUP BY Number ORDER BY Total DESC LIMIT 100' 

SELECT Number, Name, SUM(HP+Attack+Defense+SpAtk+SpDef+Speed) AS Total FROM trainning.pokemon GROUP BY Number ORDER BY Total DESC LIMIT 100;
+--------+---------------------------+-------+
| Number | Name                      | Total |
+--------+---------------------------+-------+
|    165 | Mega Mewtwo Y             |   780 |
|    164 | Mega Mewtwo X             |   780 |
|    427 | Mega Rayquaza             |   780 |
|    423 | Primal Kyogre             |   770 |
|    425 | Primal Groudon            |   770 |
|    553 | Arceus                    |   720 |

hdfs dfs -text /user/everis-bigdata/pokemon/5/*
#165,Mega Mewtwo Y,780
#164,Mega Mewtwo X,780
#427,Mega Rayquaza,780
#425,Primal Groudon,770
#423,Primal Kyogre,770
#553,Arceus,720

# cria txt local com query
hdfs dfs -text /user/everis-bigdata/pokemon/1/* > lendarios.txt
hdfs dfs -text /user/everis-bigdata/pokemon/2/* > tipoUnico.txt
hdfs dfs -text /user/everis-bigdata/pokemon/3/* > maisRapido10.txt
hdfs dfs -text /user/everis-bigdata/pokemon/4/* > menosHP50.txt
hdfs dfs -text /user/everis-bigdata/pokemon/5/* > maiorAtributo100.txt

~~~

### Adicionais
~~~shell
#Arquivo sh com os comandos para importar tabela para o HDFS(sqoop_import.sh)
#Arquivo utilizado para criar base de dados Pokemon(pokemon.sql)

#Conectar com o Mysql
mysql -u root -h localhost -pEveris@2021

#Contar linhas
hdfs dfs -cat /user/everis-bigdata/pokemon/1/* |wc -l
~~~

### Material
[Máquina Virtual utilizada](https://hermes.digitalinnovation.one/files/acceleration/Everis_BigData-v3.ova) <br>
[Slide da aula](https://drive.google.com/file/d/1ZN53soEHPYiRS1hCtEN3L3lYSnuCZH9-/view) <br>
[Dataset Kaggle](https://www.kaggle.com/abcsds/pokemon)
