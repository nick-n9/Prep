Step 1: Create a Maven Project in IntelliJ
Open IntelliJ IDEA.
Click on File → New Project.
Select Maven.
Enter the following details:
Group ID: com.order.management
Artifact ID: customer-order-management
Version: 1.0-SNAPSHOT
Project Name: customer-order-management
JDK: Select 1.8.0_301
Click Next → Finish.
Step 2: Update pom.xml Dependencies
Open pom.xml and replace the content with:

xml
Copy
Edit
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.order.management</groupId>
    <artifactId>customer-order-management</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <dependencies>
        <!-- SQL Server JDBC Driver -->
        <dependency>
            <groupId>com.microsoft.sqlserver</groupId>
            <artifactId>mssql-jdbc</artifactId>
            <version>9.4.0.jre8</version>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.28</version>
            <scope>provided</scope>
        </dependency>

        <!-- JAX-RS (Jersey) for REST API -->
        <dependency>
            <groupId>org.glassfish.jersey.containers</groupId>
            <artifactId>jersey-container-servlet</artifactId>
            <version>2.36</version>
        </dependency>

        <!-- Swagger for API Documentation -->
        <dependency>
            <groupId>io.swagger.core.v3</groupId>
            <artifactId>swagger-jaxrs2</artifactId>
            <version>2.2.10</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
Reload Maven Dependencies
Open the Maven tab in IntelliJ.
Click Reload All Maven Projects (🔄).
Step 3: Set Up the Project Structure
Manually create the following folder structure inside src/main/java/com/order/management/:

bash
Copy
Edit
com/order/management/
├── config/            # Configuration classes (Swagger, Jersey)
├── controller/        # REST API Controllers
├── dao/               # Data Access Objects (DB operations)
├── db/                # Database connection class
├── model/             # Entity classes (Customer, Order, Supplier, etc.)
Step 4: Configure JAX-RS for REST APIs
Create JerseyConfig.java inside src/main/java/com/order/management/config/:

java
Copy
Edit
package com.order.management.config;

import org.glassfish.jersey.server.ResourceConfig;
import jakarta.ws.rs.ApplicationPath;

@ApplicationPath("/api")
public class JerseyConfig extends ResourceConfig {
    public JerseyConfig() {
        packages("com.order.management.controller");
    }
}
Explanation
@ApplicationPath("/api"): All APIs will be accessible under /api/*.
Registers API controllers inside com.order.management.controller.
Step 5: Set Up Database Connection
Create DatabaseConnection.java inside src/main/java/com/order/management/db/:

java
Copy
Edit
package com.order.management.db;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DatabaseConnection {
    private static final String URL = "jdbc:sqlserver://localhost:1433;databaseName=CustomerOrderDB;encrypt=true;trustServerCertificate=true";
    private static final String USER = "your_username";
    private static final String PASSWORD = "your_password";

    public static Connection getConnection() {
        try {
            return DriverManager.getConnection(URL, USER, PASSWORD);
        } catch (SQLException e) {
            throw new RuntimeException("Database connection failed!", e);
        }
    }
}
Step 6: Create Entity Class for Customer
Create Customer.java inside src/main/java/com/order/management/model/:

java
Copy
Edit
package com.order.management.model;

import lombok.*;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class Customer {
    private int customerId;
    private String name;
    private String email;
}
Step 7: Implement DAO for Customer
Create CustomerDAO.java inside src/main/java/com/order/management/dao/:

java
Copy
Edit
package com.order.management.dao;

import com.order.management.db.DatabaseConnection;
import com.order.management.model.Customer;
import lombok.extern.slf4j.Slf4j;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

@Slf4j
public class CustomerDAO {

    public void addCustomer(Customer customer) {
        String sql = "INSERT INTO Customer (name, email) VALUES (?, ?)";
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setString(1, customer.getName());
            stmt.setString(2, customer.getEmail());
            stmt.executeUpdate();
            log.info("Customer added: {}", customer);
        } catch (SQLException e) {
            log.error("Error adding customer", e);
        }
    }

    public List<Customer> getAllCustomers() {
        List<Customer> customers = new ArrayList<>();
        String sql = "SELECT * FROM Customer";
        try (Connection conn = DatabaseConnection.getConnection();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            while (rs.next()) {
                customers.add(new Customer(rs.getInt("customer_id"),
                                           rs.getString("name"),
                                           rs.getString("email")));
            }
        } catch (SQLException e) {
            log.error("Error retrieving customers", e);
        }
        return customers;
    }
}
Step 8: Create REST API Controller
Create CustomerController.java inside src/main/java/com/order/management/controller/:

java
Copy
Edit
package com.order.management.controller;

import com.order.management.dao.CustomerDAO;
import com.order.management.model.Customer;
import jakarta.ws.rs.*;
import jakarta.ws.rs.core.MediaType;
import java.util.List;

@Path("/customers")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class CustomerController {
    private final CustomerDAO customerDAO = new CustomerDAO();

    @POST
    public String addCustomer(Customer customer) {
        customerDAO.addCustomer(customer);
        return "Customer added successfully!";
    }

    @GET
    public List<Customer> getAllCustomers() {
        return customerDAO.getAllCustomers();
    }
}

Step 9: Create Supplier Entity Class
Create Supplier.java inside src/main/java/com/order/management/model/:

java
Copy
Edit
package com.order.management.model;

import lombok.*;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class Supplier {
    private int supplierId;
    private String name;
    private String email;
}
Step 10: Implement DAO for Supplier
Create SupplierDAO.java inside src/main/java/com/order/management/dao/:

java
Copy
Edit
package com.order.management.dao;

import com.order.management.db.DatabaseConnection;
import com.order.management.model.Supplier;
import lombok.extern.slf4j.Slf4j;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

@Slf4j
public class SupplierDAO {

    public void addSupplier(Supplier supplier) {
        String sql = "INSERT INTO Supplier (name, email) VALUES (?, ?)";
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setString(1, supplier.getName());
            stmt.setString(2, supplier.getEmail());
            stmt.executeUpdate();
            log.info("Supplier added: {}", supplier);
        } catch (SQLException e) {
            log.error("Error adding supplier", e);
        }
    }

    public List<Supplier> getAllSuppliers() {
        List<Supplier> suppliers = new ArrayList<>();
        String sql = "SELECT * FROM Supplier";
        try (Connection conn = DatabaseConnection.getConnection();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            while (rs.next()) {
                suppliers.add(new Supplier(rs.getInt("supplier_id"),
                                           rs.getString("name"),
                                           rs.getString("email")));
            }
        } catch (SQLException e) {
            log.error("Error retrieving suppliers", e);
        }
        return suppliers;
    }
}
Step 11: Create REST API for Supplier
Create SupplierController.java inside src/main/java/com/order/management/controller/:

java
Copy
Edit
package com.order.management.controller;

import com.order.management.dao.SupplierDAO;
import com.order.management.model.Supplier;
import jakarta.ws.rs.*;
import jakarta.ws.rs.core.MediaType;
import java.util.List;

@Path("/suppliers")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class SupplierController {
    private final SupplierDAO supplierDAO = new SupplierDAO();

    @POST
    public String addSupplier(Supplier supplier) {
        supplierDAO.addSupplier(supplier);
        return "Supplier added successfully!";
    }

    @GET
    public List<Supplier> getAllSuppliers() {
        return supplierDAO.getAllSuppliers();
    }
}
Step 12: Create Order Entity Class
Create Order.java inside src/main/java/com/order/management/model/:

java
Copy
Edit
package com.order.management.model;

import lombok.*;

import java.sql.Timestamp;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class Order {
    private int orderId;
    private int customerId;
    private Timestamp orderDate;
    private Timestamp deliveryDate;
    private String status;
}
Step 13: Implement DAO for Order
Create OrderDAO.java inside src/main/java/com/order/management/dao/:

java
Copy
Edit
package com.order.management.dao;

import com.order.management.db.DatabaseConnection;
import com.order.management.model.Order;
import lombok.extern.slf4j.Slf4j;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

@Slf4j
public class OrderDAO {

    public void addOrder(Order order) {
        String sql = "INSERT INTO Orders (customer_id, order_date, delivery_date, status) VALUES (?, ?, ?, ?)";
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, order.getCustomerId());
            stmt.setTimestamp(2, order.getOrderDate());
            stmt.setTimestamp(3, order.getDeliveryDate());
            stmt.setString(4, order.getStatus());
            stmt.executeUpdate();
            log.info("Order added: {}", order);
        } catch (SQLException e) {
            log.error("Error adding order", e);
        }
    }

    public List<Order> getOrdersByCustomer(int customerId) {
        List<Order> orders = new ArrayList<>();
        String sql = "SELECT * FROM Orders WHERE customer_id = ? ORDER BY order_date DESC";
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, customerId);
            try (ResultSet rs = stmt.executeQuery()) {
                while (rs.next()) {
                    orders.add(new Order(rs.getInt("order_id"),
                                         rs.getInt("customer_id"),
                                         rs.getTimestamp("order_date"),
                                         rs.getTimestamp("delivery_date"),
                                         rs.getString("status")));
                }
            }
        } catch (SQLException e) {
            log.error("Error retrieving orders", e);
        }
        return orders;
    }
}
Step 14: Create REST API for Orders
Create OrderController.java inside src/main/java/com/order/management/controller/:

java
Copy
Edit
package com.order.management.controller;

import com.order.management.dao.OrderDAO;
import com.order.management.model.Order;
import jakarta.ws.rs.*;
import jakarta.ws.rs.core.MediaType;
import java.util.List;

@Path("/orders")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class OrderController {
    private final OrderDAO orderDAO = new OrderDAO();

    @POST
    public String addOrder(Order order) {
        orderDAO.addOrder(order);
        return "Order placed successfully!";
    }

    @GET
    @Path("/{customerId}")
    public List<Order> getOrdersByCustomer(@PathParam("customerId") int customerId) {
        return orderDAO.getOrdersByCustomer(customerId);
    }
}
Step 15: Run and Test APIs in Postman
Start your Java application and test the following APIs.

Base URL: http://localhost:8080/api/
Customer APIs
HTTP Method	Endpoint	Description
POST	/customers	Add a new customer
GET	/customers	Get all customers
Supplier APIs
HTTP Method	Endpoint	Description
POST	/suppliers	Add a new supplier
GET	/suppliers	Get all suppliers
Order APIs
HTTP Method	Endpoint	Description
POST	/orders	Place a new order
GET	/orders/{customerId}	Get orders by customer ID

Step 16: Create Item Entity Class
Create Item.java inside src/main/java/com/order/management/model/:

java
Copy
Edit
package com.order.management.model;

import lombok.*;

import java.sql.Timestamp;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class Item {
    private int itemId;
    private String name;
    private String description;
    private Timestamp creationTime;
    private Timestamp expirationTime;
}
Step 17: Implement DAO for Item
Create ItemDAO.java inside src/main/java/com/order/management/dao/:

java
Copy
Edit
package com.order.management.dao;

import com.order.management.db.DatabaseConnection;
import com.order.management.model.Item;
import lombok.extern.slf4j.Slf4j;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

@Slf4j
public class ItemDAO {

    public void addItem(Item item) {
        String sql = "INSERT INTO Item (name, description, creation_time, expiration_time) VALUES (?, ?, ?, ?)";
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setString(1, item.getName());
            stmt.setString(2, item.getDescription());
            stmt.setTimestamp(3, item.getCreationTime());
            stmt.setTimestamp(4, item.getExpirationTime());
            stmt.executeUpdate();
            log.info("Item added: {}", item);
        } catch (SQLException e) {
            log.error("Error adding item", e);
        }
    }

    public List<Item> getAllItems() {
        List<Item> items = new ArrayList<>();
        String sql = "SELECT * FROM Item";
        try (Connection conn = DatabaseConnection.getConnection();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            while (rs.next()) {
                items.add(new Item(rs.getInt("item_id"),
                                   rs.getString("name"),
                                   rs.getString("description"),
                                   rs.getTimestamp("creation_time"),
                                   rs.getTimestamp("expiration_time")));
            }
        } catch (SQLException e) {
            log.error("Error retrieving items", e);
        }
        return items;
    }
}
Step 18: Create REST API for Item Management
Create ItemController.java inside src/main/java/com/order/management/controller/:

java
Copy
Edit
package com.order.management.controller;

import com.order.management.dao.ItemDAO;
import com.order.management.model.Item;
import jakarta.ws.rs.*;
import jakarta.ws.rs.core.MediaType;
import java.util.List;

@Path("/items")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class ItemController {
    private final ItemDAO itemDAO = new ItemDAO();

    @POST
    public String addItem(Item item) {
        itemDAO.addItem(item);
        return "Item added successfully!";
    }

    @GET
    public List<Item> getAllItems() {
        return itemDAO.getAllItems();
    }
}
Step 19: Implement Supplier-Item Mapping Table (Stock Management)
Entity Class for Supplier-Item Mapping
Create SupplierItem.java inside src/main/java/com/order/management/model/:

java
Copy
Edit
package com.order.management.model;

import lombok.*;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class SupplierItem {
    private int supplierId;
    private int itemId;
    private int quantity;
}
DAO for Supplier-Item Mapping
Create SupplierItemDAO.java inside src/main/java/com/order/management/dao/:

java
Copy
Edit
package com.order.management.dao;

import com.order.management.db.DatabaseConnection;
import com.order.management.model.SupplierItem;
import lombok.extern.slf4j.Slf4j;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

@Slf4j
public class SupplierItemDAO {

    public void addSupplierItem(SupplierItem supplierItem) {
        String sql = "INSERT INTO Supplier_Items (supplier_id, item_id, quantity) VALUES (?, ?, ?)";
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, supplierItem.getSupplierId());
            stmt.setInt(2, supplierItem.getItemId());
            stmt.setInt(3, supplierItem.getQuantity());
            stmt.executeUpdate();
            log.info("Supplier-Item mapping added: {}", supplierItem);
        } catch (SQLException e) {
            log.error("Error adding supplier-item mapping", e);
        }
    }

    public int getItemStock(int itemId) {
        String sql = "SELECT SUM(quantity) FROM Supplier_Items WHERE item_id = ?";
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, itemId);
            try (ResultSet rs = stmt.executeQuery()) {
                if (rs.next()) {
                    return rs.getInt(1);
                }
            }
        } catch (SQLException e) {
            log.error("Error retrieving stock quantity", e);
        }
        return 0;
    }
}
Step 20: Implement Order Validation (Stock Availability)
Modify OrderDAO.java to check stock before placing an order:

java
Copy
Edit
public boolean isStockAvailable(int itemId, int requiredQuantity) {
    int availableStock = new SupplierItemDAO().getItemStock(itemId);
    return availableStock >= requiredQuantity;
}

public boolean placeOrderWithValidation(Order order, int itemId, int quantity) {
    if (!isStockAvailable(itemId, quantity)) {
        log.error("Order cannot be placed. Item {} is out of stock.", itemId);
        return false;
    }
    addOrder(order);
    log.info("Order placed successfully.");
    return true;
}
Step 21: Create REST API for Order Placement with Validation
Modify OrderController.java:

java
Copy
Edit
@POST
@Path("/validate")
public String placeOrderWithValidation(Order order, @QueryParam("itemId") int itemId, @QueryParam("quantity") int quantity) {
    boolean success = orderDAO.placeOrderWithValidation(order, itemId, quantity);
    return success ? "Order placed successfully!" : "Order cannot be placed. Item is out of stock.";
}
Step 22: Run and Test APIs in Postman
Start your Java application and test the following APIs.

Item APIs
HTTP Method	Endpoint	Description
POST	/items	Add a new item
GET	/items	Get all items
Supplier-Item Stock APIs
HTTP Method	Endpoint	Description
POST	/suppliers/items	Map supplier with item stock
GET	/items/stock/{itemId}	Get stock for an item
Order APIs (with validation)
HTTP Method	Endpoint	Description
POST	/orders/validate	Place an order with stock check

Step 23: Implement Order Modification
We need an API to allow customers to update their order details, such as changing the delivery date or modifying the item quantity.

Modify OrderDAO.java to Add an Update Order Method
Add this method inside OrderDAO.java:

java
Copy
Edit
public boolean updateOrder(int orderId, Timestamp newDeliveryDate, String newStatus) {
    String sql = "UPDATE Orders SET delivery_date = ?, status = ? WHERE order_id = ?";
    try (Connection conn = DatabaseConnection.getConnection();
         PreparedStatement stmt = conn.prepareStatement(sql)) {
        stmt.setTimestamp(1, newDeliveryDate);
        stmt.setString(2, newStatus);
        stmt.setInt(3, orderId);
        int rowsUpdated = stmt.executeUpdate();
        return rowsUpdated > 0;
    } catch (SQLException e) {
        log.error("Error updating order", e);
        return false;
    }
}
Modify OrderController.java to Add an Update Order API
Add this method inside OrderController.java:

java
Copy
Edit
@PUT
@Path("/{orderId}")
public String updateOrder(
        @PathParam("orderId") int orderId,
        @QueryParam("deliveryDate") String deliveryDate,
        @QueryParam("status") String status) {

    Timestamp newDeliveryDate = Timestamp.valueOf(deliveryDate + " 00:00:00");

    boolean success = orderDAO.updateOrder(orderId, newDeliveryDate, status);
    return success ? "Order updated successfully!" : "Failed to update order.";
}
Step 24: Implement Order Cancellation
To cancel an order, we will delete it from the database.

Modify OrderDAO.java to Add a Cancel Order Method
Add this method inside OrderDAO.java:

java
Copy
Edit
public boolean cancelOrder(int orderId) {
    String sql = "DELETE FROM Orders WHERE order_id = ?";
    try (Connection conn = DatabaseConnection.getConnection();
         PreparedStatement stmt = conn.prepareStatement(sql)) {
        stmt.setInt(1, orderId);
        int rowsDeleted = stmt.executeUpdate();
        return rowsDeleted > 0;
    } catch (SQLException e) {
        log.error("Error canceling order", e);
        return false;
    }
}
Modify OrderController.java to Add a Cancel Order API
Add this method inside OrderController.java:

java
Copy
Edit
@DELETE
@Path("/{orderId}")
public String cancelOrder(@PathParam("orderId") int orderId) {
    boolean success = orderDAO.cancelOrder(orderId);
    return success ? "Order canceled successfully!" : "Failed to cancel order.";
}
Step 25: Run and Test APIs in Postman
Start your Java application and test the following APIs.

Order Modification & Cancellation APIs
HTTP Method	Endpoint	Description
PUT	/orders/{orderId}	Modify an existing order
DELETE	/orders/{orderId}	Cancel an order
Example Test Cases
Modify an order:

URL: PUT /orders/1?deliveryDate=2025-02-10&status=Processing
Response: "Order updated successfully!"
Cancel an order:

URL: DELETE /orders/1
Response: "Order canceled successfully!"

Step 26: Implement Supplier Item Removal
A supplier should be able to remove an item they own. We will create an API to delete an item from the database.

Modify ItemDAO.java to Add a Remove Item Method
Add this method inside ItemDAO.java:

java
Copy
Edit
public boolean removeItem(int itemId) {
    String sql = "DELETE FROM Item WHERE item_id = ?";
    try (Connection conn = DatabaseConnection.getConnection();
         PreparedStatement stmt = conn.prepareStatement(sql)) {
        stmt.setInt(1, itemId);
        int rowsDeleted = stmt.executeUpdate();
        return rowsDeleted > 0;
    } catch (SQLException e) {
        log.error("Error removing item", e);
        return false;
    }
}
Modify ItemController.java to Add a Remove Item API
Add this method inside ItemController.java:

java
Copy
Edit
@DELETE
@Path("/{itemId}")
public String removeItem(@PathParam("itemId") int itemId) {
    boolean success = itemDAO.removeItem(itemId);
    return success ? "Item removed successfully!" : "Failed to remove item.";
}
Step 27: Implement Supplier Order Cancellation
A supplier should be able to cancel an order if necessary.

Modify OrderDAO.java to Add a Supplier Order Cancellation Method
Add this method inside OrderDAO.java:

java
Copy
Edit
public boolean cancelOrderBySupplier(int orderId) {
    String sql = "DELETE FROM Orders WHERE order_id = ?";
    try (Connection conn = DatabaseConnection.getConnection();
         PreparedStatement stmt = conn.prepareStatement(sql)) {
        stmt.setInt(1, orderId);
        int rowsDeleted = stmt.executeUpdate();
        return rowsDeleted > 0;
    } catch (SQLException e) {
        log.error("Error canceling order by supplier", e);
        return false;
    }
}
Modify OrderController.java to Add a Supplier Order Cancellation API
Add this method inside OrderController.java:

java
Copy
Edit
@DELETE
@Path("/supplier/{orderId}")
public String cancelOrderBySupplier(@PathParam("orderId") int orderId) {
    boolean success = orderDAO.cancelOrderBySupplier(orderId);
    return success ? "Order canceled by supplier successfully!" : "Failed to cancel order.";
}
Step 28: Run and Test APIs in Postman
Start your Java application and test the following APIs.

Supplier Item Removal & Order Cancellation APIs
HTTP Method	Endpoint	Description
DELETE	/items/{itemId}	Remove an item
DELETE	/orders/supplier/{orderId}	Cancel an order as a supplier
Example Test Cases
Remove an item:

URL: DELETE /items/2
Response: "Item removed successfully!"
Cancel an order as a supplier:

URL: DELETE /orders/supplier/3
Response: "Order canceled by supplier successfully!"

Step 29: Implement Order History Retrieval for Customers
We will create an API that allows customers to view their order history. The orders will be sorted in descending order by order date, ensuring the new orders are displayed on top.

Modify OrderDAO.java to Add a Method for Fetching Customer Orders
Add this method inside OrderDAO.java:

java
Copy
Edit
public List<Order> getCustomerOrders(int customerId) {
    List<Order> orders = new ArrayList<>();
    String sql = "SELECT * FROM Orders WHERE customer_id = ? ORDER BY order_date DESC";
    try (Connection conn = DatabaseConnection.getConnection();
         PreparedStatement stmt = conn.prepareStatement(sql)) {
        stmt.setInt(1, customerId);
        try (ResultSet rs = stmt.executeQuery()) {
            while (rs.next()) {
                orders.add(new Order(rs.getInt("order_id"),
                                     rs.getInt("customer_id"),
                                     rs.getTimestamp("order_date"),
                                     rs.getTimestamp("delivery_date"),
                                     rs.getString("status")));
            }
        }
    } catch (SQLException e) {
        log.error("Error retrieving customer orders", e);
    }
    return orders;
}
Step 30: Create REST API for Retrieving Customer Order History
Create the following method in OrderController.java to fetch the customer's orders:

java
Copy
Edit
@GET
@Path("/customer/{customerId}")
public List<Order> getCustomerOrders(@PathParam("customerId") int customerId) {
    return orderDAO.getCustomerOrders(customerId);
}
This endpoint will return the list of orders for the customer, ordered by the most recent ones at the top.

Step 31: Run and Test APIs in Postman
Start your Java application and test the following API.

Customer Order History API
HTTP Method	Endpoint	Description
GET	/orders/customer/{customerId}	Get customer order history
Example Test Case
Get customer order history:
URL: GET /orders/customer/1
Response:
json
Copy
Edit
[
  {
    "orderId": 5,
    "customerId": 1,
    "orderDate": "2025-01-29 10:00:00",
    "deliveryDate": "2025-02-05 00:00:00",
    "status": "Processing"
  },
  {
    "orderId": 3,
    "customerId": 1,
    "orderDate": "2025-01-15 14:00:00",
    "deliveryDate": "2025-01-22 00:00:00",
    "status": "Delivered"
  }
]

Step 35: Implement Stock Availability Validation
To ensure that an order is only placed if items are in stock, we will create a function to validate the stock before an order is placed.

Step 35.1: Modify ItemDAO.java to Check Stock Availability
We need to check if enough quantity of an item is available in stock before an order can be placed. Here's the method to validate stock availability:

java
Copy
Edit
public boolean isItemInStock(int itemId, int quantity) {
    String sql = "SELECT qty FROM Supplier_Items WHERE item_id = ?";
    try (Connection conn = DatabaseConnection.getConnection();
         PreparedStatement stmt = conn.prepareStatement(sql)) {
        stmt.setInt(1, itemId);
        try (ResultSet rs = stmt.executeQuery()) {
            if (rs.next()) {
                int stockQty = rs.getInt("qty");
                return stockQty >= quantity;
            }
        }
    } catch (SQLException e) {
        log.error("Error checking stock availability", e);
    }
    return false;
}
Step 35.2: Modify OrderController.java to Include Stock Validation
Before placing an order, we'll check if the items are available in stock. Here's how we can modify the order creation process:

java
Copy
Edit
@POST
@Path("/order")
@Consumes(MediaType.APPLICATION_JSON)
public String createOrder(OrderRequest orderRequest) {
    for (OrderItem item : orderRequest.getItems()) {
        boolean isInStock = itemDAO.isItemInStock(item.getItemId(), item.getQuantity());
        if (!isInStock) {
            return "Item " + item.getItemId() + " is out of stock!";
        }
    }

    // Proceed with creating the order (assuming no stock issues)
    boolean success = orderDAO.createOrder(orderRequest);
    return success ? "Order placed successfully!" : "Failed to place order.";
}
Step 36: Implement Customer Order Modification
We already implemented order modification in earlier steps, but let's make sure we handle stock updates and any modifications made to the order.

Step 36.1: Modify OrderDAO.java to Update Stock when Modifying Orders
For order modifications, we must update the stock quantity if the item quantity changes. Here's how you can implement that in OrderDAO.java:

java
Copy
Edit
public boolean updateOrder(int orderId, List<OrderItem> updatedItems) {
    try (Connection conn = DatabaseConnection.getConnection()) {
        conn.setAutoCommit(false);

        // Update each item in the order
        for (OrderItem item : updatedItems) {
            boolean isInStock = itemDAO.isItemInStock(item.getItemId(), item.getQuantity());
            if (!isInStock) {
                return false; // Item is out of stock
            }

            String updateItemSql = "UPDATE Orders_Items SET quantity = ? WHERE order_id = ? AND item_id = ?";
            try (PreparedStatement stmt = conn.prepareStatement(updateItemSql)) {
                stmt.setInt(1, item.getQuantity());
                stmt.setInt(2, orderId);
                stmt.setInt(3, item.getItemId());
                stmt.executeUpdate();
            }
        }

        conn.commit();
        return true;
    } catch (SQLException e) {
        log.error("Error updating order", e);
        return false;
    }
}
Step 37: Implement Logging and Error Handling
Proper logging and error handling are essential for maintaining the project and troubleshooting issues.

Step 37.1: Add Log4j for Logging
First, include the Log4j dependency in your pom.xml file:

xml
Copy
Edit
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.14.1</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.14.1</version>
</dependency>
Then, create a log4j2.xml configuration file in src/main/resources:

xml
Copy
Edit
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Console" />
        </Root>
    </Loggers>
</Configuration>
You can now use Logger for logging:

java
Copy
Edit
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class SomeClass {
    private static final Logger log = LogManager.getLogger(SomeClass.class);

    public void someMethod() {
        log.info("This is an info message.");
        log.error("This is an error message.");
    }
}
Step 37.2: Handle Errors Gracefully
Modify the controllers to catch and handle errors:

java
Copy
Edit
@GET
@Path("/orders/{orderId}")
public Response getOrder(@PathParam("orderId") int orderId) {
    try {
        Order order = orderDAO.getOrder(orderId);
        if (order == null) {
            return Response.status(Response.Status.NOT_FOUND).entity("Order not found").build();
        }
        return Response.ok(order).build();
    } catch (Exception e) {
        log.error("Error fetching order", e);
        return Response.status(Response.Status.INTERNAL_SERVER_ERROR).entity("Internal server error").build();
    }
}
