# ProyectoFinal--Programacion2
//Codigo
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

// Clase para conexión a la base de datos
class DBConnection {
    private static final String URL = "jdbc:mysql://127.0.0.1:3306/sakila2";
    private static final String USER = "root";
    private static final String PASSWORD = "1234";

    // Método para obtener conexión a la base de datos
    public static Connection getConnection() throws SQLException {
        try {
            // Intentar cargar el driver
            Class.forName("com.mysql.cj.jdbc.Driver");
            System.out.println("Driver cargado exitosamente.");
            // Conectar a la base de datos
            return DriverManager.getConnection(URL, USER, PASSWORD);
        } catch (ClassNotFoundException e) {
            System.out.println("Error cargando el driver: " + e.getMessage());
            throw new SQLException("No se encontró el driver MySQL. Asegúrate de que mysql-connector-java.jar esté en el classpath.");
        }
    }
}

// Interfaz para operaciones CRUD
interface IDataPost<T> {
    void post(T entity) throws SQLException;
    T get(int id) throws SQLException;
    List<T> getAll() throws SQLException;
    void put(T entity) throws SQLException;
    void delete(int id) throws SQLException;
}

// Clase abstracta base para el controlador de datos
abstract class DataContext<T> implements IDataPost<T> {}

// Modelo para la entidad Customer
class Customer {
    private int customerId;
    private String firstName;
    private String lastName;
    private City city;

    public Customer(int customerId, String firstName, String lastName, City city) {
        this.customerId = customerId;
        this.firstName = firstName;
        this.lastName = lastName;
        this.city = city;
    }

    public int getCustomerId() { return customerId; }
    public void setCustomerId(int customerId) { this.customerId = customerId; }
    public String getFirstName() { return firstName; }
    public void setFirstName(String firstName) { this.firstName = firstName; }
    public String getLastName() { return lastName; }
    public void setLastName(String lastName) { this.lastName = lastName; }
    public City getCity() { return city; }
    public void setCity(City city) { this.city = city; }

    @Override
    public String toString() {
        return "Customer ID: " + customerId + ", Nombre: " + firstName + " " + lastName + ", Ciudad: " + city.getCityName();
    }
}

// Modelo para la entidad City
class City {
    private int cityId;
    private String cityName;
    private Country country;

    public City(int cityId, String cityName, Country country) {
        this.cityId = cityId;
        this.cityName = cityName;
        this.country = country;
    }

    public int getCityId() { return cityId; }
    public String getCityName() { return cityName; }
    public Country getCountry() { return country; }
}

// Modelo para la entidad Country
class Country {
    private int countryId;
    private String countryName;

    public Country(int countryId, String countryName) {
        this.countryId = countryId;
        this.countryName = countryName;
    }

    public int getCountryId() { return countryId; }
    public String getCountryName() { return countryName; }
}

// Controlador para manejar las operaciones de Customer en MySQL
class CustomerController extends DataContext<Customer> {
    public CustomerController() { super(); }

    @Override
    public void post(Customer customer) throws SQLException {
        String query = "INSERT INTO customer (first_name, last_name, city_id, store_id , address_id ) VALUES (?, ?, ?, ?,?)";
        try (Connection conn = DBConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {
            stmt.setString(1, customer.getFirstName());
            stmt.setString(2, customer.getLastName());
            stmt.setInt(3, customer.getCity().getCityId());
            stmt.setInt(4, 1);
            stmt.setInt(5, 1);
            stmt.executeUpdate();
            System.out.println("Customer creado exitosamente.");
        }
    }

    @Override
    public Customer get(int id) throws SQLException {
        String query = "SELECT customer_id, first_name, last_name, city_id FROM customer WHERE customer_id = ?";
        try (Connection conn = DBConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {
            stmt.setInt(1, id);
            ResultSet rs = stmt.executeQuery();
            if (rs.next()) {
                int customerId = rs.getInt("customer_id");
                String firstName = rs.getString("first_name");
                String lastName = rs.getString("last_name");
                City city = new City(rs.getInt("city_id"), "SampleCity", new Country(1, "SampleCountry"));
                return new Customer(customerId, firstName, lastName, city);
            }
        }
        return null;
    }

    @Override
    public List<Customer> getAll() throws SQLException {
        List<Customer> customers = new ArrayList<>();
        String query = "SELECT customer_id, first_name, last_name, city_id FROM customer";
        try (Connection conn = DBConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(query);
             ResultSet rs = stmt.executeQuery()) {
            while (rs.next()) {
                int customerId = rs.getInt("customer_id");
                String firstName = rs.getString("first_name");
                String lastName = rs.getString("last_name");
                City city = new City(rs.getInt("city_id"), "SampleCity", new Country(1, "SampleCountry"));
                customers.add(new Customer(customerId, firstName, lastName, city));
            }
        }
        return customers;
    }


    @Override
    public void put(Customer customer) throws SQLException {
        String query = "UPDATE customer SET first_name = ?, last_name = ? WHERE customer_id = ?";
        try (Connection conn = DBConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {
            stmt.setString(1, customer.getFirstName());
            stmt.setString(2, customer.getLastName());
            stmt.setInt(3, customer.getCustomerId());
            stmt.executeUpdate();
            System.out.println("Customer actualizado exitosamente.");
        }
    }

    @Override
    public void delete(int id) throws SQLException {
        String query = "DELETE FROM customer WHERE customer_id = ?";
        try (Connection conn = DBConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(query)) {
            stmt.setInt(1, id);
            stmt.executeUpdate();
            System.out.println("Customer eliminado exitosamente.");
        }
    }
}

// Aplicación principal con interacción en consola
public class Main {
    private static final CustomerController customerController = new CustomerController();
    private static final Scanner scanner = new Scanner(System.in);

    public static void main(String[] args) {
        int option;
        do {
            System.out.println("1. Crear Customer");
            System.out.println("2. Buscar Customer");
            System.out.println("3. Actualizar Customer");
            System.out.println("4. Eliminar Customer");
            System.out.println("5. Salir");
            option = scanner.nextInt();
            scanner.nextLine();

            try {
                switch (option) {
                    case 1:
                        createCustomer();
                        break;
                    case 2:
                        searchCustomer();
                        break;
                    case 3:
                        updateCustomer();
                        break;
                    case 4:
                        deleteCustomer();
                        break;
                    case 5:
                        System.out.println("Saliendo...");
                        break;
                    default:
                        System.out.println("Opción no válida");
                        break;
                }
            } catch (SQLException e) {
                System.out.println("Error de base de datos: " + e.getMessage());
            }
        } while (option != 5);
    }

    private static void createCustomer() throws SQLException {
        System.out.print("Nombre: ");
        String firstName = scanner.nextLine();
        System.out.print("Apellido: ");
        String lastName = scanner.nextLine();
        Customer customer = new Customer(0, firstName, lastName, new City(1, "SampleCity", new Country(1, "SampleCountry")));
        customerController.post(customer);
    }

    private static void searchCustomer() throws SQLException {
        System.out.print("ID de Customer: ");
        int id = scanner.nextInt();
        scanner.nextLine();
        Customer customer = customerController.get(id);
        System.out.println(customer != null ? customer : "Customer no encontrado.");
    }

    private static void updateCustomer() throws SQLException {
        System.out.print("ID de Customer a actualizar: ");
        int id = scanner.nextInt();
        scanner.nextLine();
        Customer customer = customerController.get(id);
        if (customer != null) {
            System.out.print("Nuevo Nombre: ");
            customer.setFirstName(scanner.nextLine());
            System.out.print("Nuevo Apellido: ");
            customer.setLastName(scanner.nextLine());
            customerController.put(customer);
        } else {
            System.out.println("Customer no encontrado.");
        }
    }

    private static void deleteCustomer() throws SQLException {
        System.out.print("ID de Customer a eliminar: ");
        int id = scanner.nextInt();
        scanner.nextLine();
        customerController.delete(id);
    }
}


 //base de dato mysqlworbench
CREATE DATABASE  IF NOT EXISTS `sakila2` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */;
USE `sakila2`;
-- MySQL dump 10.13  Distrib 8.0.40, for Win64 (x86_64)
--
-- Host: localhost    Database: sakila2
-- ------------------------------------------------------
-- Server version	8.3.0

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!50503 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table ` actor`
--

DROP TABLE IF EXISTS ` actor`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE ` actor` (
  `actor_id` int NOT NULL AUTO_INCREMENT,
  `first_name` varchar(45) NOT NULL,
  `last_name` varchar(45) NOT NULL,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`actor_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table ` actor`
--

LOCK TABLES ` actor` WRITE;
/*!40000 ALTER TABLE ` actor` DISABLE KEYS */;
/*!40000 ALTER TABLE ` actor` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `address`
--

DROP TABLE IF EXISTS `address`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `address` (
  `address_id` int NOT NULL AUTO_INCREMENT,
  `address` varchar(255) NOT NULL,
  `address2` varchar(255) DEFAULT NULL,
  `district` varchar(50) DEFAULT NULL,
  `city_id` int NOT NULL,
  `postal_code` varchar(10) DEFAULT NULL,
  `phone` varchar(20) DEFAULT NULL,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`address_id`),
  KEY `city_id` (`city_id`),
  CONSTRAINT `address_ibfk_1` FOREIGN KEY (`city_id`) REFERENCES `city` (`city_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `address`
--

LOCK TABLES `address` WRITE;
/*!40000 ALTER TABLE `address` DISABLE KEYS */;
INSERT INTO `address` VALUES (1,'Santo Domingo','Santo Domingo','Santo Domingo',1,'123','809','2024-11-10 02:51:39');
/*!40000 ALTER TABLE `address` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `category`
--

DROP TABLE IF EXISTS `category`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `category` (
  `category_id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(25) NOT NULL,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`category_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `category`
--

LOCK TABLES `category` WRITE;
/*!40000 ALTER TABLE `category` DISABLE KEYS */;
/*!40000 ALTER TABLE `category` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `city`
--

DROP TABLE IF EXISTS `city`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `city` (
  `city_id` int NOT NULL AUTO_INCREMENT,
  `city` varchar(50) NOT NULL,
  `country_id` int NOT NULL,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`city_id`),
  KEY `country_id` (`country_id`),
  CONSTRAINT `city_ibfk_1` FOREIGN KEY (`country_id`) REFERENCES `country` (`country_id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `city`
--

LOCK TABLES `city` WRITE;
/*!40000 ALTER TABLE `city` DISABLE KEYS */;
INSERT INTO `city` VALUES (1,'San Cristobal',1,'2024-11-10 02:49:22'),(2,'Santo Domingo',1,'2024-11-10 04:02:16'),(3,'Azua',1,'2024-11-10 04:02:16'),(4,'Bani',1,'2024-11-10 04:02:16'),(5,'Cotui',1,'2024-11-10 04:04:17'),(6,'Romana',1,'2024-11-10 04:04:17');
/*!40000 ALTER TABLE `city` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `country`
--

DROP TABLE IF EXISTS `country`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `country` (
  `country_id` int NOT NULL AUTO_INCREMENT,
  `country` varchar(50) NOT NULL,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`country_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `country`
--

LOCK TABLES `country` WRITE;
/*!40000 ALTER TABLE `country` DISABLE KEYS */;
INSERT INTO `country` VALUES (1,'RD','2024-11-10 02:46:08');
/*!40000 ALTER TABLE `country` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `customer`
--

DROP TABLE IF EXISTS `customer`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `customer` (
  `customer_id` int NOT NULL AUTO_INCREMENT,
  `store_id` int NOT NULL,
  `first_name` varchar(45) NOT NULL,
  `last_name` varchar(45) NOT NULL,
  `email` varchar(50) DEFAULT NULL,
  `address_id` int NOT NULL,
  `active` tinyint(1) NOT NULL DEFAULT '1',
  `create_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `city_id` int DEFAULT NULL,
  PRIMARY KEY (`customer_id`),
  KEY `store_id` (`store_id`),
  KEY `address_id` (`address_id`),
  CONSTRAINT `customer_ibfk_1` FOREIGN KEY (`store_id`) REFERENCES `store` (`store_id`),
  CONSTRAINT `customer_ibfk_2` FOREIGN KEY (`address_id`) REFERENCES `address` (`address_id`)
) ENGINE=InnoDB AUTO_INCREMENT=24 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='ALTER TABLE customer ADD city_id CONSTRAINT FK_city_id FOREIGN KEY (city_id) REFERENCES city(city_id);\\n';
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `customer`
--

LOCK TABLES `customer` WRITE;
/*!40000 ALTER TABLE `customer` DISABLE KEYS */;
INSERT INTO `customer` VALUES (5,1,'Eriberto','Perdomo',NULL,1,1,'2024-11-10 03:08:35','2024-11-10 03:08:35',1),(6,1,'Heriberto','Perdomo',NULL,1,1,'2024-11-10 03:16:44','2024-11-10 03:16:44',1),(7,1,'Heriberto','Perdomo',NULL,1,1,'2024-11-10 03:23:13','2024-11-10 03:23:13',1),(8,1,'Miguel ','perez',NULL,1,1,'2024-11-10 03:28:21','2024-11-10 03:28:21',1),(9,1,'2','3',NULL,1,1,'2024-11-10 04:15:34','2024-11-10 04:15:34',1),(10,1,'s','d',NULL,1,1,'2024-11-10 12:15:41','2024-11-10 12:15:41',1),(11,1,'d','d',NULL,1,1,'2024-11-10 13:12:05','2024-11-10 13:12:05',1),(12,1,'Heriberto ','Perdomo',NULL,1,1,'2024-11-10 13:15:40','2024-11-10 13:15:40',2),(13,1,'w','s',NULL,1,1,'2024-11-10 13:19:20','2024-11-10 13:19:20',1),(14,1,'Heriberto','Perdomo',NULL,1,1,'2024-11-10 13:22:40','2024-11-10 13:22:40',2),(15,1,'Heriberto','Perdomo',NULL,1,1,'2024-11-10 13:26:37','2024-11-10 13:26:37',2),(16,1,'Heriberto','Perdomo0',NULL,1,1,'2024-11-10 13:31:06','2024-11-10 13:31:06',2),(17,1,'Heriberto','Perdomo',NULL,1,1,'2024-11-10 13:34:35','2024-11-10 13:34:35',1),(18,1,'Heriberto','Perdomo',NULL,1,1,'2024-11-10 14:16:26','2024-11-10 14:16:26',1),(19,1,'He.','Per',NULL,1,1,'2024-11-10 14:18:01','2024-11-10 14:18:01',1),(21,1,'Heriberto','Perdomo',NULL,1,1,'2024-11-10 20:28:40','2024-11-10 20:28:40',1),(22,1,'Heriberto ','Perdomo',NULL,1,1,'2024-11-10 21:16:26','2024-11-10 21:16:26',1);
/*!40000 ALTER TABLE `customer` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `film`
--

DROP TABLE IF EXISTS `film`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `film` (
  `film_id` int NOT NULL AUTO_INCREMENT,
  `title` varchar(255) NOT NULL,
  `description` text,
  `release_year` year DEFAULT NULL,
  `language_id` int NOT NULL,
  `rental_duration` int DEFAULT '3',
  `rental_rate` decimal(4,2) DEFAULT '4.99',
  `length` int DEFAULT NULL,
  `replacement_cost` decimal(5,2) DEFAULT '19.99',
  `rating` varchar(10) DEFAULT NULL,
  `special_features` varchar(255) DEFAULT NULL,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`film_id`),
  KEY `language_id` (`language_id`),
  CONSTRAINT `film_ibfk_1` FOREIGN KEY (`language_id`) REFERENCES `language` (`language_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `film`
--

LOCK TABLES `film` WRITE;
/*!40000 ALTER TABLE `film` DISABLE KEYS */;
/*!40000 ALTER TABLE `film` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `inventory`
--

DROP TABLE IF EXISTS `inventory`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `inventory` (
  `inventory_id` int NOT NULL AUTO_INCREMENT,
  `film_id` int NOT NULL,
  `store_id` int NOT NULL,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`inventory_id`),
  KEY `film_id` (`film_id`),
  KEY `store_id` (`store_id`),
  CONSTRAINT `inventory_ibfk_1` FOREIGN KEY (`film_id`) REFERENCES `film` (`film_id`),
  CONSTRAINT `inventory_ibfk_2` FOREIGN KEY (`store_id`) REFERENCES `store` (`store_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `inventory`
--

LOCK TABLES `inventory` WRITE;
/*!40000 ALTER TABLE `inventory` DISABLE KEYS */;
/*!40000 ALTER TABLE `inventory` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `language`
--

DROP TABLE IF EXISTS `language`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `language` (
  `language_id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`language_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `language`
--

LOCK TABLES `language` WRITE;
/*!40000 ALTER TABLE `language` DISABLE KEYS */;
/*!40000 ALTER TABLE `language` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `payment`
--

DROP TABLE IF EXISTS `payment`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `payment` (
  `payment_id` int NOT NULL AUTO_INCREMENT,
  `customer_id` int NOT NULL,
  `staff_id` int NOT NULL,
  `rental_id` int NOT NULL,
  `amount` decimal(5,2) NOT NULL,
  `payment_date` datetime NOT NULL,
  PRIMARY KEY (`payment_id`),
  KEY `customer_id` (`customer_id`),
  KEY `staff_id` (`staff_id`),
  KEY `rental_id` (`rental_id`),
  CONSTRAINT `payment_ibfk_1` FOREIGN KEY (`customer_id`) REFERENCES `customer` (`customer_id`),
  CONSTRAINT `payment_ibfk_2` FOREIGN KEY (`staff_id`) REFERENCES `staff` (`staff_id`),
  CONSTRAINT `payment_ibfk_3` FOREIGN KEY (`rental_id`) REFERENCES `rental` (`rental_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `payment`
--

LOCK TABLES `payment` WRITE;
/*!40000 ALTER TABLE `payment` DISABLE KEYS */;
/*!40000 ALTER TABLE `payment` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `rental`
--

DROP TABLE IF EXISTS `rental`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `rental` (
  `rental_id` int NOT NULL AUTO_INCREMENT,
  `rental_date` datetime NOT NULL,
  `inventory_id` int NOT NULL,
  `customer_id` int NOT NULL,
  `return_date` datetime DEFAULT NULL,
  `staff_id` int NOT NULL,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`rental_id`),
  KEY `inventory_id` (`inventory_id`),
  KEY `customer_id` (`customer_id`),
  KEY `staff_id` (`staff_id`),
  CONSTRAINT `rental_ibfk_1` FOREIGN KEY (`inventory_id`) REFERENCES `inventory` (`inventory_id`),
  CONSTRAINT `rental_ibfk_2` FOREIGN KEY (`customer_id`) REFERENCES `customer` (`customer_id`),
  CONSTRAINT `rental_ibfk_3` FOREIGN KEY (`staff_id`) REFERENCES `staff` (`staff_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `rental`
--

LOCK TABLES `rental` WRITE;
/*!40000 ALTER TABLE `rental` DISABLE KEYS */;
/*!40000 ALTER TABLE `rental` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `staff`
--

DROP TABLE IF EXISTS `staff`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `staff` (
  `staff_id` int NOT NULL AUTO_INCREMENT,
  `first_name` varchar(45) NOT NULL,
  `last_name` varchar(45) NOT NULL,
  `address_id` int NOT NULL,
  `email` varchar(50) DEFAULT NULL,
  `active` tinyint(1) NOT NULL DEFAULT '1',
  `username` varchar(16) NOT NULL,
  `password` varchar(16) DEFAULT NULL,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`staff_id`),
  KEY `address_id` (`address_id`),
  CONSTRAINT `staff_ibfk_1` FOREIGN KEY (`address_id`) REFERENCES `address` (`address_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='ALTER TABLE staff\nADD COLUMN store_id INT NOT NULL,\nADD FOREIGN KEY (store_id) REFERENCES store(store_id);';
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `staff`
--

LOCK TABLES `staff` WRITE;
/*!40000 ALTER TABLE `staff` DISABLE KEYS */;
INSERT INTO `staff` VALUES (1,'Eriberto','Perdomo',1,'EribertoPerdomo@gmail.com',1,'Eriberto','1','2024-11-10 02:56:39');
/*!40000 ALTER TABLE `staff` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `store`
--

DROP TABLE IF EXISTS `store`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `store` (
  `store_id` int NOT NULL AUTO_INCREMENT,
  `manager_staff_id` int NOT NULL,
  `address_id` int NOT NULL,
  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`store_id`),
  KEY `manager_staff_id` (`manager_staff_id`),
  KEY `address_id` (`address_id`),
  CONSTRAINT `store_ibfk_1` FOREIGN KEY (`manager_staff_id`) REFERENCES `staff` (`staff_id`),
  CONSTRAINT `store_ibfk_2` FOREIGN KEY (`address_id`) REFERENCES `address` (`address_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `store`
--

LOCK TABLES `store` WRITE;
/*!40000 ALTER TABLE `store` DISABLE KEYS */;
INSERT INTO `store` VALUES (1,1,1,'2024-11-10 03:04:20');
/*!40000 ALTER TABLE `store` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2024-11-12 19:58:29





 




