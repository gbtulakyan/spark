#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

FROM spark:3.5.6

USER root

# Install Maven and create temporary app directory
RUN apt-get update && apt-get install -y maven && \
    mkdir -p /app && cd /app && \
    cat <<EOF > pom.xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>spark.minio</groupId>
  <artifactId>deps</artifactId>
  <version>1.0.0</version>
  
  <properties>
    <!-- Latest stable versions for MinIO support -->
    <hadoop.version>3.3.6</hadoop.version>
    <aws.sdk.version>1.12.767</aws.sdk.version>
    <maven.compiler.source>8</maven.compiler.source>
    <maven.compiler.target>8</maven.compiler.target>
  </properties>
  
  <dependencies>
    <!-- MinIO Support via S3A filesystem -->
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-aws</artifactId>
      <version>\${hadoop.version}</version>
      <exclusions>
        <exclusion>
          <groupId>com.amazonaws</groupId>
          <artifactId>aws-java-sdk-bundle</artifactId>
        </exclusion>
        <exclusion>
          <groupId>org.slf4j</groupId>
          <artifactId>slf4j-reload4j</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
    
    <!-- AWS SDK Bundle for S3 operations -->
    <dependency>
      <groupId>com.amazonaws</groupId>
      <artifactId>aws-java-sdk-bundle</artifactId>
      <version>\${aws.sdk.version}</version>
    </dependency>
    
    <!-- Hadoop Common - Required by hadoop-aws -->
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-common</artifactId>
      <version>\${hadoop.version}</version>
      <exclusions>
        <exclusion>
          <groupId>org.slf4j</groupId>
          <artifactId>slf4j-reload4j</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
  </dependencies>
  
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.11.0</version>
        <configuration>
          <source>8</source>
          <target>8</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
EOF

# Download dependencies and clean up
RUN cd /app && \
    mvn dependency:copy-dependencies -DoutputDirectory=/opt/spark/jars && \
    cd / && rm -rf /app /root/.m2 && \
    apt-get purge -y maven && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Set environment variables for MinIO/S3 compatibility
ENV SPARK_HOME=/opt/spark
ENV HADOOP_CONF_DIR=/opt/spark/conf

# Create configuration directory
RUN mkdir -p $HADOOP_CONF_DIR

# Switch back to spark user for security
USER spark

