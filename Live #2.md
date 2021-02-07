# Live #2 - Monitoramento de clusters Hadoop de alto nível com HDFS e Yarn

**[HDFS](https://www.cetax.com.br/apache-hadoop-tudo-o-que-voce-precisa-saber/)** Hadoop Distributed File System (HDFS), é responsável por gerenciar o disco das máquinas que compõem o Cluster. HDFS também serve para leitura e gravação dos dados. <br>
**[Yarn](https://pt.wikipedia.org/wiki/Hadoop)** é uma plataforma do ecossistema Hadoop para gerenciamento de recursos responsável pelo gerenciamento dos recursos computacionais em cluster, assim como pelo agendamento dos recursos. <br>
**[MapReduce](https://pt.wikipedia.org/wiki/Hadoop)** Modelo de programação para processamento em larga escala. <br>
**[Hadoop Docs](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html)** Comandos Shell. <br>

* Big Data: petabyte <br>
* distribuído, horizontal <br>
* cluster: grupo computadores <br>
* nó: computador <br>
* daemon: programa rodando em um nó <br>
* Hadoop "SO de Big Data" <br>
* processing: spark, mapreduce <br>
* resource management: yarn <br>
* storage: hdfs, blocos 128MB replicados 3x <br>
* HDFS: Hadoop Distributed File System <br>
* NameNode: index <br>
* DataNode: armazena blocos arquivos <br>
* Secondary NameNode: suporte <br>
* YARN: Yet Another Resource Negotiator <br>
* App: job submetido <br>
* App Master: gerencia tarefas <br>
* Container: unidade alocação <br>
* Node Manager: monitoramento <br>
* Resource Manager: gerenciador global <br>
* MapReduce distribuído com HDFS: processamento nos servidores onde o dado está armazenado. <br>


Profº [**Rodrigo Garcia**](https://www.linkedin.com/in/rodsantosg/) <br>

### 1 - Iniciar serviços do hadoop
~~~shell
# Via script de apoio
sh script_apoio/start_all_service.sh
# Tem o Cassandra, pode demorar

# Individualmente
sudo service hadoop-hdfs-namenode start
sudo service hadoop-hdfs-secondarynamenode start
sudo service hadoop-hdfs-datanode start
sudo service hadoop-yarn-nodemanager start
sudo service hadoop-yarn-resourcemanager start
sudo service hadoop-mapreduce-historyserver start
# sudo service zookeeper-server start
# sudo service hbase-master start
# sudo service hbase-regionserver start
# sudo service hive-metastore start
# sudo service hive-server2 start
# sudo service impala-server start
# sudo service impala-state-store start
# sudo service impala-catalog start
~~~

### 2 - GET/PUT
~~~shell
# Copiar arquivo HDFS para local
hdfs dfs -get /tmp/file_teste.txt

# Ingestão manual
hdfs dfs -put file_teste.txt /user/everis-bigdata/
# file_teste.txt (local file)
# -put -f: overwrites if exists
# /user/everis-bigdata/ (HDFS folder)
~~~

### 3 - Manipulando arquivos no HDFS
~~~shell
# Visualizar informações dos arquivos do HDFS
hdfs dfs -ls -h /

# Visualizando apenas as 10 primeiras linhas do arquivo
hdfs dfs -cat /tmp/file_teste.txt | head -10

# Remover um arquivo
hdfs dfs -rm /tmp/file_teste.txt

# Ingestão arquivo
hdfs dfs -put file_teste.txt /tmp

# Criar um diretório
hdfs dfs -mkdir /tmp/delete

# Copiando arquivo para outro diretório
hdfs dfs -cp /tmp/file_teste.txt /tmp/delete/

# Criando um arquivo vazio
hdfs dfs -touchz /tmp/delete/empty_file

# Remover um diretório
hdfs dfs -rm /tmp/delete
# -rm -r (ou -R): recursivamente (também remove pastas com arquivos)

# Informações sobre uso de disco dos diretórios
sudo -u hdfs hdfs dfs -du -h /user

# Informações sobre os files e blocks
sudo -u hdfs hdfs fsck /tmp/ -files -blocks
~~~

### 4 - Yarn
~~~shell
# Rodando um job
sudo -u hdfs yarn jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar wordcount /tmp/file_teste.txt /tmp/wc_output

# Verificando o que tem no arquivo file_teste.txt
hdfs dfs -cat /tmp/file_teste.txt

# Verificando o diretório do output
hdfs dfs -ls /tmp/wc_output

# Verificando o arquivo de saída
hdfs dfs -cat /tmp/wc_output/part-r-00000

# Verificando o log do job com more
sudo -u hdfs yarn logs -applicationId application_1611297663370_0001 |more

# Verificando o log do job com o tail -f
sudo -u hdfs yarn logs -applicationId application_1611297663370_0001 |tail -f

# Copiando log para o servidor local
sudo -u hdfs yarn logs -applicationId application_1611297663370_0001 > wordcount.log
~~~

### Comandos adicionais
~~~shell
# Verificar status dos serviços
service hadoop-hdfs-namenode status

# Ler arquivos
cat file_teste.txt

# Editar arquivo com um editor preferencial
vim file_teste.txt

# Caso dê erro de permissão quando for realizar o PUT
sudo -u hdfs hdfs dfs -chmod -R 777 /tmp

# Verificar status de utilização de memória
free -mh

# Caso o Namenode entre em Safe Mode
sudo -u hdfs hdfs dfsadmin -safemode leave

# Parar serviços do Hadoop
## Via script de apoio
sh script_apoio/stop_all_service.sh

# Reiniciar serviços do Hadoop
## Via script de apoio
sh script_apoio/restart_all_service.sh

## Individualmente
# sudo service hive-server2 stop
# sudo service hive-metastore stop
# sudo service hbase-master stop
# sudo service hbase-regionserver stop
# sudo service impala-catalog stop
# sudo service impala-state-store stop
# sudo service impala-server stop
sudo service hadoop-mapreduce-historyserver stop
sudo service hadoop-yarn-resourcemanager stop
sudo service hadoop-yarn-nodemanager stop
sudo service hadoop-hdfs-datanode stop
sudo service hadoop-hdfs-namenode stop
sudo service hadoop-hdfs-secondarynamenode stop
# sudo service zookeeper-server stop

# Corrigir problema com o caminho de gravação de logs
sudo sed -i 's|hdfs://|hdfs://bigdata-srv:8020/|g' /etc/hadoop/conf/yarn-site.xml #Corrigido por @jcr0ch4
# Conferir output
cat /etc/hadoop/conf/yarn-site.xml |grep bigdata-srv
#<value>bigdata-srv</value>
#    <value>hdfs://bigdata-srv:8020/var/log/hadoop-yarn/apps</value>
# arrumar manualmente (i insert, ESC :wq write quit, ctrl+z sair sem salvar)
sudo vim /etc/hadoop/conf/yarn-site.xml

# Libera firewall da vm
sudo systemctl disable firewalld && sudo systemctl stop firewalld

# Caso o namenode não inicie, alterar ou comentar o ip da VM em /etc/hosts
sudo vim /etc/hosts

~~~

### Material
[Máquina Virtual utilizada](https://hermes.digitalinnovation.one/files/acceleration/Everis_BigData-v3.ova) <br>
[Slide da aula](https://drive.google.com/file/d/1mSzcFASKCTir5ecdRNA7hHb7DoVfOMM0/view)
