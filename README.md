Step 1: Create a Spring Boot Project in IntelliJ IDEA
We will use Spring Boot with Maven and include the required dependencies.

1.1 Open IntelliJ IDEA and Create a New Project
Open IntelliJ IDEA.
Click on File → New → Project.
Select Spring Initializr (Ensure you have Spring Boot plugin installed).
Enter the following details:
Group: com.example
Artifact: customer-order-management
Name: customer-order-management
Type: Maven
Java Version: 1.8
Packaging: Jar
Spring Boot Version: 2.7.x (Latest version that supports JDK 1.8)
Click Next.
Step 2: Add Required Dependencies
In the dependencies selection section, add:

Spring Web (For REST APIs)
Spring Boot DevTools (For hot reloads)
Spring Data JPA (For ORM and JPA)
SQL Server Driver (For database connection)
Hibernate Validator (For input validation)
Lombok (To reduce boilerplate code)
Once selected, click Finish.

Step 3: Configure application.properties for SQL Server

Inside src/main/resources/application.properties,
 add:

# Server Configuration
server.port=8080

# Database Configuration
spring.datasource.url=jdbc:sqlserver://localhost:1433;databaseName=customer_order_db;encrypt=false
spring.datasource.username=your_username
spring.datasource.password=your_password
spring.datasource.driver-class-name=com.microsoft.sqlserver.jdbc.SQLServerDriver

# JPA & Hibernate Configurations
spring.jpa.database-platform=org.hibernate.dialect.SQLServerDialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

Step 4: Define Project Structure
We will use Controller-Service-Repository pattern.

customer-order-management/
│── src/main/java/com/example/customerordermanagement
│   ├── controller/         // REST Controllers
│   ├── service/            // Business Logic Layer
│   ├── repository/         // Data Access Layer (JPA Repositories)
│   ├── entity/             // JPA Entities (Database Tables)
│   ├── dto/                // Data Transfer Objects
│   ├── CustomerOrderManagementApplication.java  // Main class
│── src/main/resources/
│   ├── application.properties
│── pom.xml

Step 5: Create JPA Entities
Let's define the database structure using JPA annotations.

5.1 Customer Entity

package com.example.customerordermanagement.entity;

import lombok.*;
import javax.persistence.*;
import java.util.List;

@Entity
@Table(name = "customer")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Customer {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String email;

    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL)
    private List<Order> orders;
}


5.2 Supplier Entity

package com.example.customerordermanagement.entity;

import lombok.*;
import javax.persistence.*;
import java.util.List;

@Entity
@Table(name = "supplier")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Supplier {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    @OneToMany(mappedBy = "supplier", cascade = CascadeType.ALL)
    private List<Item> items;
}

5.3 Item Entity

package com.example.customerordermanagement.entity;

import lombok.*;
import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.List;

@Entity
@Table(name = "item")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Item {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String description;
    private LocalDateTime creationTime;
    private LocalDateTime expirationTime;

    @ManyToOne
    @JoinColumn(name = "supplier_id")
    private Supplier supplier;

    @ManyToMany(mappedBy = "items")
    private List<Order> orders;
}

5.4 Order Entity

package com.example.customerordermanagement.entity;

import lombok.*;
import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.List;

@Entity
@Table(name = "orders")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private LocalDateTime orderDate;
    private LocalDateTime deliveryDate;
    private String status;

    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;

    @ManyToMany
    @JoinTable(
        name = "orders_items",
        joinColumns = @JoinColumn(name = "order_id"),
        inverseJoinColumns = @JoinColumn(name = "item_id")
    )
    private List<Item> items;
}

5.5 Supplier_Items Mapping (For Stock Quantity)

package com.example.customerordermanagement.entity;

import lombok.*;
import javax.persistence.*;

@Entity
@Table(name = "supplier_items")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class SupplierItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "supplier_id")
    private Supplier supplier;

    @ManyToOne
    @JoinColumn(name = "item_id")
    private Item item;

    private int quantity;
}

5.6 Orders_Items Mapping (For Order Quantity)

package com.example.customerordermanagement.entity;

import lombok.*;
import javax.persistence.*;

@Entity
@Table(name = "orders_items")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class OrderItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;

    @ManyToOne
    @JoinColumn(name = "item_id")
    private Item item;

    private int quantity;
}

Step 6: Create Repository Layer
We will define JPA Repositories to interact with the database.

6.1 Customer Repository
package com.example.customerordermanagement.repository;

import com.example.customerordermanagement.entity.Customer;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface CustomerRepository extends JpaRepository<Customer, Long> {
}

6.2 Supplier Repository

package com.example.customerordermanagement.repository;

import com.example.customerordermanagement.entity.Supplier;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface SupplierRepository extends JpaRepository<Supplier, Long> {
}

6.3 Order Repository
package com.example.customerordermanagement.repository;

import com.example.customerordermanagement.entity.Order;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {
}

Step 7: Implement Service Layer
The Service Layer contains business logic and interacts with the Repository Layer.

7.1 Create Customer Service
This service will handle customer-related operations.

CustomerService Interface

package com.example.customerordermanagement.service;

import com.example.customerordermanagement.entity.Customer;
import java.util.List;

public interface CustomerService {
    Customer createCustomer(Customer customer);
    Customer getCustomerById(Long id);
    List<Customer> getAllCustomers();
    Customer updateCustomer(Long id, Customer customer);
    void deleteCustomer(Long id);
}

CustomerService Implementation,

package com.example.customerordermanagement.service.impl;

import com.example.customerordermanagement.entity.Customer;
import com.example.customerordermanagement.repository.CustomerRepository;
import com.example.customerordermanagement.service.CustomerService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class CustomerServiceImpl implements CustomerService {

    @Autowired
    private CustomerRepository customerRepository;

    @Override
    public Customer createCustomer(Customer customer) {
        return customerRepository.save(customer);
    }

    @Override
    public Customer getCustomerById(Long id) {
        return customerRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Customer not found with id: " + id));
    }

    @Override
    public List<Customer> getAllCustomers() {
        return customerRepository.findAll();
    }

    @Override
    public Customer updateCustomer(Long id, Customer customer) {
        Customer existingCustomer = getCustomerById(id);
        existingCustomer.setName(customer.getName());
        existingCustomer.setEmail(customer.getEmail());
        return customerRepository.save(existingCustomer);
    }

    @Override
    public void deleteCustomer(Long id) {
        Customer customer = getCustomerById(id);
        customerRepository.delete(customer);
    }
}

7.2 Create Supplier Service
This service will handle supplier-related operations.

SupplierService Interface,
package com.example.customerordermanagement.service;

import com.example.customerordermanagement.entity.Supplier;
import java.util.List;

public interface SupplierService {
    Supplier createSupplier(Supplier supplier);
    Supplier getSupplierById(Long id);
    List<Supplier> getAllSuppliers();
    Supplier updateSupplier(Long id, Supplier supplier);
    void deleteSupplier(Long id);
}

SupplierService Implementation,
package com.example.customerordermanagement.service.impl;

import com.example.customerordermanagement.entity.Supplier;
import com.example.customerordermanagement.repository.SupplierRepository;
import com.example.customerordermanagement.service.SupplierService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class SupplierServiceImpl implements SupplierService {

    @Autowired
    private SupplierRepository supplierRepository;

    @Override
    public Supplier createSupplier(Supplier supplier) {
        return supplierRepository.save(supplier);
    }

    @Override
    public Supplier getSupplierById(Long id) {
        return supplierRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Supplier not found with id: " + id));
    }

    @Override
    public List<Supplier> getAllSuppliers() {
        return supplierRepository.findAll();
    }

    @Override
    public Supplier updateSupplier(Long id, Supplier supplier) {
        Supplier existingSupplier = getSupplierById(id);
        existingSupplier.setName(supplier.getName());
        existingSupplier.setEmail(supplier.getEmail());
        return supplierRepository.save(existingSupplier);
    }

    @Override
    public void deleteSupplier(Long id) {
        Supplier supplier = getSupplierById(id);
        supplierRepository.delete(supplier);
    }
}

7.3 Create Item Service
This service will handle item-related operations.

ItemService Interface,
package com.example.customerordermanagement.service;

import com.example.customerordermanagement.entity.Item;
import java.util.List;

public interface ItemService {
    Item createItem(Item item);
    Item getItemById(Long id);
    List<Item> getAllItems();
    Item updateItem(Long id, Item item);
    void deleteItem(Long id);
}

ItemService Implementation,
package com.example.customerordermanagement.service.impl;

import com.example.customerordermanagement.entity.Item;
import com.example.customerordermanagement.repository.ItemRepository;
import com.example.customerordermanagement.service.ItemService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class ItemServiceImpl implements ItemService {

    @Autowired
    private ItemRepository itemRepository;

    @Override
    public Item createItem(Item item) {
        return itemRepository.save(item);
    }

    @Override
    public Item getItemById(Long id) {
        return itemRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Item not found with id: " + id));
    }

    @Override
    public List<Item> getAllItems() {
        return itemRepository.findAll();
    }

    @Override
    public Item updateItem(Long id, Item item) {
        Item existingItem = getItemById(id);
        existingItem.setName(item.getName());
        existingItem.setDescription(item.getDescription());
        existingItem.setCreationTime(item.getCreationTime());
        existingItem.setExpirationTime(item.getExpirationTime());
        return itemRepository.save(existingItem);
    }

    @Override
    public void deleteItem(Long id) {
        Item item = getItemById(id);
        itemRepository.delete(item);
    }
}

7.4 Create Order Service
This service will handle order-related operations.

OrderService Interface,
package com.example.customerordermanagement.service;

import com.example.customerordermanagement.entity.Order;
import java.util.List;

public interface OrderService {
    Order placeOrder(Order order);
    Order getOrderById(Long id);
    List<Order> getAllOrders();
    Order updateOrder(Long id, Order order);
    void cancelOrder(Long id);
}

OrderService Implementation,

package com.example.customerordermanagement.service.impl;

import com.example.customerordermanagement.entity.Order;
import com.example.customerordermanagement.repository.OrderRepository;
import com.example.customerordermanagement.service.OrderService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class OrderServiceImpl implements OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Override
    public Order placeOrder(Order order) {
        return orderRepository.save(order);
    }

    @Override
    public Order getOrderById(Long id) {
        return orderRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Order not found with id: " + id));
    }

    @Override
    public List<Order> getAllOrders() {
        return orderRepository.findAll();
    }

    @Override
    public Order updateOrder(Long id, Order order) {
        Order existingOrder = getOrderById(id);
        existingOrder.setDeliveryDate(order.getDeliveryDate());
        existingOrder.setStatus(order.getStatus());
        return orderRepository.save(existingOrder);
    }

    @Override
    public void cancelOrder(Long id) {
        Order order = getOrderById(id);
        orderRepository.delete(order);
    }
}

Step 8: Implement Controller Layer
The Controller Layer will handle HTTP requests and call the Service Layer.

8.1 Customer Controller,
package com.example.customerordermanagement.controller;

import com.example.customerordermanagement.entity.Customer;
import com.example.customerordermanagement.service.CustomerService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/customers")
public class CustomerController {

    @Autowired
    private CustomerService customerService;

    @PostMapping
    public ResponseEntity<Customer> createCustomer(@RequestBody Customer customer) {
        return ResponseEntity.ok(customerService.createCustomer(customer));
    }

    @GetMapping("/{id}")
    public ResponseEntity<Customer> getCustomerById(@PathVariable Long id) {
        return ResponseEntity.ok(customerService.getCustomerById(id));
    }

    @GetMapping
    public ResponseEntity<List<Customer>> getAllCustomers() {
        return ResponseEntity.ok(customerService.getAllCustomers());
    }

    @PutMapping("/{id}")
    public ResponseEntity<Customer> updateCustomer(@PathVariable Long id, @RequestBody Customer customer) {
        return ResponseEntity.ok(customerService.updateCustomer(id, customer));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteCustomer(@PathVariable Long id) {
        customerService.deleteCustomer(id);
        return ResponseEntity.noContent().build();
    }
}

8.2 Supplier Controller,
package com.example.customerordermanagement.controller;

import com.example.customerordermanagement.entity.Supplier;
import com.example.customerordermanagement.service.SupplierService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/suppliers")
public class SupplierController {

    @Autowired
    private SupplierService supplierService;

    @PostMapping
    public ResponseEntity<Supplier> createSupplier(@RequestBody Supplier supplier) {
        return ResponseEntity.ok(supplierService.createSupplier(supplier));
    }

    @GetMapping("/{id}")
    public ResponseEntity<Supplier> getSupplierById(@PathVariable Long id) {
        return ResponseEntity.ok(supplierService.getSupplierById(id));
    }

    @GetMapping
    public ResponseEntity<List<Supplier>> getAllSuppliers() {
        return ResponseEntity.ok(supplierService.getAllSuppliers());
    }

    @PutMapping("/{id}")
    public ResponseEntity<Supplier> updateSupplier(@PathVariable Long id, @RequestBody Supplier supplier) {
        return ResponseEntity.ok(supplierService.updateSupplier(id, supplier));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteSupplier(@PathVariable Long id) {
        supplierService.deleteSupplier(id);
        return ResponseEntity.noContent().build();
    }
}

8.3 Item Controller,
package com.example.customerordermanagement.controller;

import com.example.customerordermanagement.entity.Item;
import com.example.customerordermanagement.service.ItemService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/items")
public class ItemController {

    @Autowired
    private ItemService itemService;

    @PostMapping
    public ResponseEntity<Item> createItem(@RequestBody Item item) {
        return ResponseEntity.ok(itemService.createItem(item));
    }

    @GetMapping("/{id}")
    public ResponseEntity<Item> getItemById(@PathVariable Long id) {
        return ResponseEntity.ok(itemService.getItemById(id));
    }

    @GetMapping
    public ResponseEntity<List<Item>> getAllItems() {
        return ResponseEntity.ok(itemService.getAllItems());
    }

    @PutMapping("/{id}")
    public ResponseEntity<Item> updateItem(@PathVariable Long id, @RequestBody Item item) {
        return ResponseEntity.ok(itemService.updateItem(id, item));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteItem(@PathVariable Long id) {
        itemService.deleteItem(id);
        return ResponseEntity.noContent().build();
    }
}


8.4 Order Controller,
package com.example.customerordermanagement.controller;

import com.example.customerordermanagement.entity.Order;
import com.example.customerordermanagement.service.OrderService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @Autowired
    private OrderService orderService;

    @PostMapping
    public ResponseEntity<Order> placeOrder(@RequestBody Order order) {
        return ResponseEntity.ok(orderService.placeOrder(order));
    }

    @GetMapping("/{id}")
    public ResponseEntity<Order> getOrderById(@PathVariable Long id) {
        return ResponseEntity.ok(orderService.getOrderById(id));
    }

    @GetMapping
    public ResponseEntity<List<Order>> getAllOrders() {
        return ResponseEntity.ok(orderService.getAllOrders());
    }

    @PutMapping("/{id}")
    public ResponseEntity<Order> updateOrder(@PathVariable Long id, @RequestBody Order order) {
        return ResponseEntity.ok(orderService.updateOrder(id, order));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> cancelOrder(@PathVariable Long id) {
        orderService.cancelOrder(id);
        return ResponseEntity.noContent().build();
    }
}

Step 9: Enable Swagger API Documentation
To test our APIs easily, let's integrate Swagger UI.

9.1 Add Dependency in pom.xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.0.2</version>
</dependency>

9.2 Access Swagger UI
Start your Spring Boot application and go to:,
http://localhost:8080/swagger-ui/index.html
Here, you can see and test all the API endpoints.

Step 10: Implement Business Logic for Order Validation

10.1 Modify OrderService for Order Validation
Update OrderService Interface,
package com.example.customerordermanagement.service;

import com.example.customerordermanagement.entity.Order;

import java.util.List;

public interface OrderService {
    Order placeOrder(Long customerId, List<Long> itemIds, List<Integer> quantities);
    Order getOrderById(Long id);
    List<Order> getAllOrders();
    Order updateOrder(Long orderId, Long customerId, List<Long> itemIds, List<Integer> quantities);
    void cancelOrder(Long orderId, Long customerId);
}

Update OrderServiceImpl with Order Validation Logic,

package com.example.customerordermanagement.service.impl;

import com.example.customerordermanagement.entity.*;
import com.example.customerordermanagement.repository.*;
import com.example.customerordermanagement.service.OrderService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.List;
import java.util.Optional;

@Service
public class OrderServiceImpl implements OrderService {

    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private CustomerRepository customerRepository;
    
    @Autowired
    private ItemRepository itemRepository;

    @Override
    public Order placeOrder(Long customerId, List<Long> itemIds, List<Integer> quantities) {
        Customer customer = customerRepository.findById(customerId)
                .orElseThrow(() -> new RuntimeException("Customer not found with id: " + customerId));

        Order order = new Order();
        order.setCustomer(customer);
        order.setOrderDate(LocalDateTime.now());
        order.setStatus("NEW");

        for (int i = 0; i < itemIds.size(); i++) {
            Long itemId = itemIds.get(i);
            int quantity = quantities.get(i);
            Item item = itemRepository.findById(itemId)
                    .orElseThrow(() -> new RuntimeException("Item not found with id: " + itemId));

            // Validate stock availability
            if (item.getStock() < quantity) {
                throw new RuntimeException("Item " + item.getName() + " is out of stock.");
            }

            item.setStock(item.getStock() - quantity); // Reduce stock
            itemRepository.save(item);
        }

        return orderRepository.save(order);
    }

    @Override
    public Order getOrderById(Long id) {
        return orderRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Order not found with id: " + id));
    }

    @Override
    public List<Order> getAllOrders() {
        return orderRepository.findAll();
    }

    @Override
    public Order updateOrder(Long orderId, Long customerId, List<Long> itemIds, List<Integer> quantities) {
        Order order = getOrderById(orderId);

        if (!order.getCustomer().getId().equals(customerId)) {
            throw new RuntimeException("Customer is not allowed to modify this order.");
        }

        for (int i = 0; i < itemIds.size(); i++) {
            Long itemId = itemIds.get(i);
            int quantity = quantities.get(i);
            Item item = itemRepository.findById(itemId)
                    .orElseThrow(() -> new RuntimeException("Item not found with id: " + itemId));

            // Validate stock availability
            if (item.getStock() < quantity) {
                throw new RuntimeException("Item " + item.getName() + " is out of stock.");
            }

            item.setStock(item.getStock() - quantity);
            itemRepository.save(item);
        }

        return orderRepository.save(order);
    }

    @Override
    public void cancelOrder(Long orderId, Long customerId) {
        Order order = getOrderById(orderId);

        if (!order.getCustomer().getId().equals(customerId)) {
            throw new RuntimeException("Customer is not allowed to cancel this order.");
        }

        orderRepository.delete(order);
    }
}

10.2 Update OrderController,
package com.example.customerordermanagement.controller;

import com.example.customerordermanagement.entity.Order;
import com.example.customerordermanagement.service.OrderService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @Autowired
    private OrderService orderService;

    @PostMapping
    public ResponseEntity<Order> placeOrder(@RequestParam Long customerId, @RequestParam List<Long> itemIds, @RequestParam List<Integer> quantities) {
        return ResponseEntity.ok(orderService.placeOrder(customerId, itemIds, quantities));
    }

    @GetMapping("/{id}")
    public ResponseEntity<Order> getOrderById(@PathVariable Long id) {
        return ResponseEntity.ok(orderService.getOrderById(id));
    }

    @GetMapping
    public ResponseEntity<List<Order>> getAllOrders() {
        return ResponseEntity.ok(orderService.getAllOrders());
    }

    @PutMapping("/{id}")
    public ResponseEntity<Order> updateOrder(@PathVariable Long id, @RequestParam Long customerId, @RequestParam List<Long> itemIds, @RequestParam List<Integer> quantities) {
        return ResponseEntity.ok(orderService.updateOrder(id, customerId, itemIds, quantities));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> cancelOrder(@PathVariable Long id, @RequestParam Long customerId) {
        orderService.cancelOrder(id, customerId);
        return ResponseEntity.noContent().build();
    }
}

10.3 Supplier Can Remove Only Their Own Items
Modify ItemServiceImpl to ensure that a Supplier can delete only their own items.

@Override
public void deleteItem(Long itemId, Long supplierId) {
    Item item = itemRepository.findById(itemId)
            .orElseThrow(() -> new RuntimeException("Item not found with id: " + itemId));

    if (!item.getSupplier().getId().equals(supplierId)) {
        throw new RuntimeException("Supplier is not allowed to delete this item.");
    }

    itemRepository.delete(item);
}



