# Microservices Architecture Project

## Description

Ce projet met en œuvre une architecture de microservices utilisant Spring Boot, Spring Cloud, et Angular pour gérer les clients, les produits et les factures. Il inclut les composants suivants :

1. **Customer Service** : Gère les clients.
2. **Inventory Service** : Gère les produits.
3. **Gateway Service** : Passerelle Spring Cloud Gateway pour le routage.
4. **Discovery Service** : Annuaire Eureka pour la découverte de services.
5. **Billing Service** : Gère les factures en utilisant OpenFeign.
6. **Angular Frontend** : Interface web pour gérer les clients, les produits et les factures.

## Structure du Projet

### Customer Service

- **Port**: 9081
- **Description**: Ce service gère les informations des clients.
- **Dépendances**: Spring Web, Spring Data JPA, H2 Database, Spring Data Rest, Eureka Client

```java
@SpringBootApplication
public class CustomerServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(CustomerServiceApplication.class, args);
    }

    @Bean
    public CommandLineRunner commandLineRunner(CustomerRepository customerRepository,
                                               RepositoryRestConfiguration restConfiguration) {
        return args -> {
            restConfiguration.exposeIdsFor(Customer.class);
            customerRepository.saveAll(
                    List.of(
                            Customer.builder().name("achraf").email("achraf@gmail.com").build(),
                            Customer.builder().name("mahmoud").email("mahmoud@gmail.com").build(),
                            Customer.builder().name("ahmed").email("ahmed@gmail.com").build()
                    )
            );
            customerRepository.findAll().forEach(c -> {
                System.out.println(c);
            });
        };
    }
}
```


### Inventory Service

- **Port**: 9082
- **Description**: Ce service gère les informations des produits.
- **Dépendances**: Spring Web, Spring Data JPA, H2 Database, Spring Data Rest, Eureka Client
  
```java
- @SpringBootApplication
  public class InventoryServiceApplication {
  public static void main(String[] args) {
  SpringApplication.run(InventoryServiceApplication.class, args);
  }

  @Bean
  CommandLineRunner start(ProductRepository productRepository, RepositoryRestConfiguration restConfiguration) {
  return args -> {
  restConfiguration.exposeIdsFor(Product.class);
  productRepository.saveAll(
  List.of(
  Product.builder().name("computer").quantity(12).price(1200).build(),
  Product.builder().name("Printer").quantity(32).price(120).build(),
  Product.builder().name("Smartphone").quantity(31).price(900).build()
  )
  );
  };
  }
  }

```
###  Gateway Service

- **Port**: 9999
- **Description**: Passerelle de routage pour les microservices.
- **Dépendances**: Spring Cloud Gateway, Eureka Client
```java
@SpringBootApplication
public class GatewaayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewaayApplication.class, args);
    }

    /* Configurer les routes via Java */
    //@Bean
    public RouteLocator routes(RouteLocatorBuilder builder) {
        return builder.routes()
                .route(r -> r.path("/customers/**").uri("lb://customer-service"))
                .route(r -> r.path("/products/**").uri("lb://inventory-service"))
                .build();
    }

    @Bean
    public DiscoveryClientRouteDefinitionLocator dynamicRoutes(ReactiveDiscoveryClient rdc,
                                                               DiscoveryLocatorProperties dlp) {
        return new DiscoveryClientRouteDefinitionLocator(rdc, dlp);
    }
}

```


###  Discovery Service

- **Port**: 8761
- **Description**:  Service Eureka pour la découverte des microservices.
- **Dépendances**:  Spring Cloud Netflix Eureka Server

````java

@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(DiscoveryServiceApplication.class, args);
    }
}


````

