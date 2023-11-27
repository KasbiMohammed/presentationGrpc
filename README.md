gRPC avec Spring Boot
Table des matières
•	Description
•	Configuration du serveur
o	Créer un projet Spring Boot
o	Ajouter des dépendances maven
o	Création des définitions de service gRPC
o	Créer une entité jpa stagere
o	Cartographie de la stagere Grpc avec la stagere Jpa
o	Implémentation du service
o	Configurer le serveur
•	Configuration du client
o	Créer un projet Spring Boot
o	Ajouter des dépendances maven
o	Générer des serres
o	Créer un dto
o	Créer un service client Stagere
o	Créer un contrôleur Stagere 
o	Configurer l'application
o	Test avec facteur
•	Sécurité
o	Mise à jour côté serveur
	Installation d'OpenSSL
	Générer des clés et des certificats
	Ajout d'une clé privée et d'un certificat au serveur
	Création d'une classe d'intercepteur
	Mise à jour du fichier de propriétés
o	Mise à jour côté client
	Ajout du certificat au client
	Création d'une classe d'intercepteur
	Mise à jour du fichier de propriétés
Description
Dans ce projet, nous développons une application gRPC Spring Boot avec des composants serveur et client. L'objectif principal est de mettre en œuvre des opérations CRUD (Créer, Lire, Mettre à jour, Supprimer) pour la gestion des stageres. L'application permettra des fonctionnalités telles que l'ajout de nouveaux stageres, la suppression, la mise à jour des informations sur les stageres et la récupération d'une liste des stageres. Nous exploiterons également la programmation réactive pour récupérer et afficher efficacement une liste d'étudiants utilisant des flux.
Configuration du serveur
pour configurer le serveur il faut suivez les étapes suivantes
Créer un projet Spring Boot
Utilisez spring initializr (ou tout ce que vous voulez) et créez un simple projet Spring Boot:

![Spring Initializr et 2 pages de plus - Profil 1 – Microsoft​ Edge 27_11_2023 22_49_22](https://github.com/KasbiMohammed/presentationGrpc/assets/147922729/c97cc56d-500f-4c93-8dc2-21296bac0dab)

Ajouter des dépendances maven
Incluez les propriétés suivantes dans votre configuration 
<protobuf.version>3.23.4</protobuf.version>
<protobuf-plugin.version>0.6.1</protobuf-plugin.version>
<grpc.version>1.58.0</grpc.version>
Ajout des dépendances suivantes :

<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-stub</artifactId>
    <version>${grpc.version}</version>
</dependency>
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-protobuf</artifactId>
    <version>${grpc.version}</version>
</dependency>
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>1.3.5</version>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>net.devh</groupId>
    <artifactId>grpc-spring-boot-starter</artifactId>
    <version>2.15.0.RELEASE</version>
</dependency>
Ajoutez le plugin suivant :

<plugin>
    <groupId>com.github.os72</groupId>
    <artifactId>protoc-jar-maven-plugin</artifactId>
    <version>3.11.4</version>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>run</goal>
            </goals>
            <configuration>
                <includeMavenTypes>direct</includeMavenTypes>
                <inputDirectories>
                    <include>src/main/resources</include>
                </inputDirectories>
                <outputTargets>
                    <outputTarget>
                        <type>java</type>
                        <outputDirectory>src/main/java</outputDirectory>
                    </outputTarget>
                    <outputTarget>
                        <type>grpc-java</type>
                        <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.15.0</pluginArtifact>
                        <outputDirectory>src/main/java</outputDirectory>
                    </outputTarget>
                </outputTargets>
            </configuration>
        </execution>
    </executions>
</plugin>
Création des définitions de service gRPC
.protoPlacez vos définitions/fichiers protobuf dans src/main/resources. Pour écrire des fichiers protobuf, veuillez vous référer à la documentation officielle de protobuf .

Vos .protofichiers ressemblentront à l'exemple ci-dessous :
