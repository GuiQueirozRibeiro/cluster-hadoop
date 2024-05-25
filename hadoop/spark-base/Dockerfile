FROM openjdk:8-jdk-slim

# Copia o arquivo de requisitos do Jupyter
COPY /config/jupyter/requirements.txt /

# Instalações e configurações
RUN apt-get update \
    && apt-get install -y wget vim ssh openssh-server curl iputils-ping nano sudo \
    python3 python3-pip python3-dev \
    build-essential libssl-dev libffi-dev libpq-dev mariadb-server libmariadb-java \
    texlive-xetex texlive-fonts-recommended texlive-plain-generic \
    && python3 -m pip install -r requirements.txt \
    && python3 -m pip install dask[bag] --upgrade \
    && python3 -m pip install --upgrade toree \
    && python3 -m pip install seaborn \
    && python3 -m bash_kernel.install \
    && python3 -c "import nltk; nltk.download('stopwords')" \
    && mkdir datasets \
    && wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-8.0.28.tar.gz \
    && tar -xzf mysql-connector-java-8.0.28.tar.gz \
    && cp mysql-connector-java-8.0.28/mysql-connector-java-8.0.28.jar /usr/spark-3.5.1/jars/ \
    && rm -rf mysql-connector-java-8.0.28* \
    && ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa \
    && cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys \
    && chmod 600 ~/.ssh/authorized_keys

# Copia a configuração SSH
COPY /config/config /root/.ssh
RUN chmod 600 /root/.ssh/config

# Variáveis de ambiente do Hadoop
ENV HADOOP_VERSION 3.4.0
ENV HADOOP_MINOR_VERSION 3
ENV HADOOP_HOME /usr/hadoop-$HADOOP_VERSION
ENV HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
ENV HDFS_NAMENODE_USER root
ENV HDFS_SECONDARYNAMENODE_USER root
ENV HDFS_DATANODE_USER root
ENV YARN_RESOURCEMANAGER_USER root
ENV YARN_NODEMANAGER_USER root

# Variáveis de ambiente do Spark
ENV SPARK_VERSION 3.5.1
ENV SPARK_HOME /usr/spark-${SPARK_VERSION}
ENV SPARK_DIST_CLASSPATH="$HADOOP_HOME/etc/hadoop/*:$HADOOP_HOME/share/hadoop/common/lib/*:$HADOOP_HOME/share/hadoop/common/*:$HADOOP_HOME/share/hadoop/hdfs/*:$HADOOP_HOME/share/hadoop/hdfs/lib/*:$HADOOP_HOME/share/hadoop/hdfs/*:$HADOOP_HOME/share/hadoop/yarn/lib/*:$HADOOP_HOME/share/hadoop/yarn/*:$HADOOP_HOME/share/hadoop/mapreduce/lib/*:$HADOOP_HOME/share/hadoop/mapreduce/*:$HADOOP_HOME/share/hadoop/tools/lib/*"

# Configurações do PySpark
ENV PYSPARK_PYTHON python3
ENV PYSPARK_DRIVER_PYTHON=python3
ENV PYTHONPATH $SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.7-src.zip:/usr/bin/python3
ENV PATH $PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$SPARK_HOME/bin:$SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.7-src.zip:$PYTHONPATH

# Copia e instala o Hadoop
COPY bin/hadoop-${HADOOP_VERSION}.tar.gz .
RUN tar zvxf hadoop-${HADOOP_VERSION}.tar.gz -C /usr/ \
    && rm hadoop-${HADOOP_VERSION}.tar.gz \
    && rm -rf ${HADOOP_HOME}/share/doc \
    && chown -R root:root ${HADOOP_HOME} \
    && echo "export JAVA_HOME=${JAVA_HOME}" >> ${HADOOP_HOME}/etc/hadoop/hadoop-env.sh

# Copia e instala o Spark
COPY bin/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_MINOR_VERSION}.tgz .
RUN mkdir ${SPARK_HOME} \
    && tar zvxf spark-${SPARK_VERSION}-bin-hadoop${HADOOP_MINOR_VERSION}.tgz \
    && mv spark-${SPARK_VERSION}-bin-hadoop${HADOOP_MINOR_VERSION}/* ${SPARK_HOME} \
    && rm -r spark-${SPARK_VERSION}-bin-hadoop${HADOOP_MINOR_VERSION} \
    && rm spark-${SPARK_VERSION}-bin-hadoop${HADOOP_MINOR_VERSION}.tgz \
    && chown -R root:root ${SPARK_HOME}

# Copia as configurações e scripts
COPY config/hadoop/*.xml /usr/hadoop-${HADOOP_VERSION}/etc/hadoop/
COPY config/spark ${SPARK_HOME}/conf/
COPY config/scripts /

EXPOSE 9000 4040 8020 22 9083 9870

ENTRYPOINT ["/bin/bash", "bootstrap.sh"]
