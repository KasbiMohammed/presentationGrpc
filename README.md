# gRPC avec Spring Boot
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
# Description
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

# Création des définitions de service gRPC
.protoPlacez vos définitions/fichiers protobuf dans src/main/resources. Pour écrire des fichiers protobuf, veuillez vous référer à la documentation officielle de protobuf .

Vos .protofichiers ressemblentront à l'exemple ci-dessous :
syntax = "proto3";
option java_package = "com.expose.grpc.stubs";

message Stagere {
  int64 id = 1;
  string nom = 2;
  string prenom = 3;
  Date dateEntree = 4;
}

message Empty {}

service StagereService {
  rpc ListStageres(Empty) returns (ListStageresResponse);
  rpc GetStagere(GetStagereRequest) returns (Stagere);
  rpc ListStageresStream(Empty) returns (stream Stagere);
  rpc CreateStagere(CreateStagereRequest) returns (Stagere);
  rpc UpdateStagere(Stagere) returns (Stagere);
  rpc DeleteStagere(DeleteStagereRequest) returns (DeleteStagereResponse);
}

message ListStageresResponse { repeated Stagere stageres = 1; }
message GetStagereRequest { int64 id = 1; }
message DeleteStagereRequest { int64 id = 1; }
message DeleteStagereResponse { string message = 1; }

message CreateStagereRequest {
   string nom = 1;
  string prenom = 2;
  Date dateEntree = 3;
}

courirmvn clean install

Veuillez noter que le colis grpc.studs a été généré. À l'intérieur de ce package, vous devriez trouver deux fichiers Java : StudentOutClassetStudentServiceGrpc

Créer une entité jpa étudiante
Créons d'abord notre entité jpa 

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class Stagere {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
 int id = 1;
  string nom = 2;
  string prenom = 3;
  Date dateEntree = 4;

}

comme toujours, vous devez créer les référentiels :

@Repository
public interface StagereRepository extends JpaRepository<Stagere, Long> {
}
Cartographie de l'étudiant Grpc avec la Stagere Jpa
Créons maintenant une classe qui mappera les objets de la classe Stagere Grpc aux objets de la classe d'étudiant jpa :


package com.expose.mappers;

import org.springframework.stereotype.Component;

import com.expose.entities.Stagere;

@Component
public class StagereMapper {
    public com.expose.grpc.stubs.StagereOuterClass.Stagere toGrpcStagere(Stagere stagere) {
        return com.expose.grpc.stubs.StagereOuterClass.Stagere.newBuilder().setId(Stagere.getId())
                .setFirstName(Stagere.getNom())
                .setLastName(Stagere.getPrenom())
                .setAge(Stagere.getDateEntree())
                .build();
    }

    public Stagere fromGrpcStagere(com.expose.grpc.stubs.StagereOuterClass.Stagere Stagere) {
        return Stagere.builder()
                .id(Stagere.getId())
                .Nom(Stagere.getNom())
                .Prenom(Stagere.getPrenom())
                .DateEntree(Stagere.getDateEntree())
                .build();
    }
}
Implémentation du service
Le protoc-jar-maven-pluginplugin génère une classe pour chacun de vos services grpc. Par exemple : MyServiceGrpcoù MyService est le nom du service grpc dans le fichier proto. Cette classe contient à la fois les stubs client et le serveur ImplBaseque vous devrez étendre.

Après cela, vous n'avez que quatre tâches à accomplir :

Assurez-vous que vos MyServiceImplextensionsMyServiceGrpc.MyServiceImplBase
Ajoutez l' @GrpcServiceannotation à votre MyServiceImplclasse
Assurez-vous que le MyServiceImplest ajouté au contexte de votre application,
soit en créant @Beanune définition dans un de vos @Configurationcours
ou en le recevoir dans les chemins détectés automatiquement par Spring (par exemple dans le même ou un sous-paquet de votre Mainclasse)
Implémentez réellement les méthodes du service grpc.
Votre classe de service grpc ressemblera alors quelque peu à l'exemple ci-dessous :


package com.expose.grpc.services;

import com.expose.entities.Stagere;
import com.expose.grpc.stubs.StagereOuterClass;
import com.expose.grpc.stubs.StagereOuterClass.*;
import com.expose.grpc.stubs.StagereServiceGrpc;
import com.expose.mappers.StagereMapper;
import com.expose.repositories.StagereRepository;
import io.grpc.Status;
import io.grpc.stub.StreamObserver;
import lombok.RequiredArgsConstructor;
import net.devh.boot.grpc.server.service.GrpcService;

import java.util.List;
import java.util.Stack;
import java.util.Timer;
import java.util.TimerTask;

@GrpcService
@RequiredArgsConstructor
public class GrpcStagereServiceIml extends StagereServiceGrpc.StagereServiceImplBase {

    private final StagereRepository StagereRepository;
    private final StagereMapper StagereMapper;

    @Override
    public void listStageres(Empty request, StreamObserver<ListStageresResponse> responseObserver) {
        List<Stagere> Stageres = StagereRepository.findAll();
        List<StagereOuterClass.Stagere> listStageres = Stageres.stream()
                .map(StagereMapper::toGrpcStagere)
                .toList();
        ListStageresResponse listStageresResponse = ListStageresResponse.newBuilder()
                .addAllStageres(listStageres)
                .build();
        responseObserver.onNext(listStageresResponse);
        responseObserver.onCompleted();
    }

    @Override
    public void listStageresStream(Empty request,
                                   StreamObserver<StagereOuterClass.Stagere> responseObserver) {
        List<Stagere> Stageres = StagereRepository.findAll();
        List<StagereOuterClass.Stagere> listStageres = Stageres.stream()
                .map(StagereMapper::toGrpcStagere)
                .toList();
        if (listStageres.isEmpty()) {
            responseObserver.onError(Status.INTERNAL.withDescription("no Stagere found").asException());
        } else {
            Stack<StagereOuterClass.Stagere> stackStageres = new Stack<>();
            stackStageres.addAll(listStageres);
            Timer timer = new Timer("Stageres timer");
            timer.schedule(new TimerTask() {

                @Override
                public void run() {
                    responseObserver.onNext(stackStageres.pop());
                    if (stackStageres.isEmpty()) {
                        responseObserver.onCompleted();
                        timer.cancel();
                    }
                }

            }, 0, 1000);
        }
    }

    @Override
    public void getStagere(GetStagereRequest request,
                           StreamObserver<StagereOuterClass.Stagere> responseObserver) {
        Stagere Stagere = StagereRepository.findById(request.getId()).orElse(null);
        if (Stagere == null) {
            responseObserver.onError(Status.INTERNAL.withDescription("Stagere not found").asException());
        } else {
            responseObserver.onNext(StagereMapper.toGrpcStagere(Stagere));
            responseObserver.onCompleted();
        }
    }

    @Override
    public void createStagere(CreateStagereRequest request,
                              StreamObserver<StagereOuterClass.Stagere> responseObserver) {
        Stagere Stagere = Stagere.builder().firstName(request.getFirstName()).lastName(request.getLastName()).age(request.getAge()).build();
        responseObserver.onNext(StagereMapper.toGrpcStagere(StagereRepository.save(Stagere)));
        responseObserver.onCompleted();
    }

    @Override
    public void updateStagere(StagereOuterClass.Stagere request,
                              StreamObserver<StagereOuterClass.Stagere> responseObserver) {
        if (StagereRepository.existsById(request.getId())) {
            Stagere Stagere = StagereRepository.save(StagereMapper.fromGrpcStagere(request));
            responseObserver.onNext(StagereMapper.toGrpcStagere(Stagere));
            responseObserver.onCompleted();
        } else {
            responseObserver.onError(Status.INTERNAL.withDescription("Stagere not found").asException());
        }
    }

    @Override
    public void deleteStagere(DeleteStagereRequest request,
                              StreamObserver<DeleteStagereResponse> responseObserver) {
        if (StagereRepository.existsById(request.getId())) {
            StagereRepository.deleteById(request.getId());
            DeleteStagereResponse deleteStagereResponse = DeleteStagereResponse.newBuilder()
                    .setMessage("Stagere Deleted").build();
            responseObserver.onNext(deleteStagereResponse);
            responseObserver.onCompleted();
        } else {
            responseObserver.onError(Status.INTERNAL.withDescription("Stagere not found").asException());
        }
    }
}
Configurer le serveur
Ajouter les propriétés suivantes àapplication.properties

spring.datasource.url=jdbc:mysql://localhost:3306/stagere?createDatabaseIfNotExist=true
spring.jpa.hibernate.ddl-auto=create
spring.datasource.username=root
spring.datasource.password=
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.highlight_sql=true




Configuration du client
Suivez ces étapes pour configurer le client :

Créer un projet Spring Boot
Rendez-vous sur https://start.spring.io et générer une application Spring Boot

initialise_spring_boot_app.png

Ajouter des dépendances maven
Incluez les propriétés suivantes dans votre configuration :

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
    <groupId>net.devh</groupId>
    <artifactId>grpc-client-spring-boot-starter</artifactId>
    <version>2.15.0.RELEASE</version>
</dependency>

<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.2</version>
    <optional>true</optional>
</dependency>

<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.5.11</version>
</dependency>

<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.4.11</version>
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
Générer des serres
Assurez-vous d'avoir l' Student.protointérieur du dossier de ressources.

courirmvn clean install

Veuillez noter que le colis grpc.studsa été généré. À l'intérieur de ce package, vous devriez trouver deux fichiers Java : StudentOutClassetStudentServiceGrpc

Créer un dto
Créez un projet pour Stagere.


