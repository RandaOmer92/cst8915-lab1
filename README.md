# CST8915 Lab 1: Algonquin Pet Store on Azure VM

**Student Name**: Randa Omer

**Student ID**: 041079985

**Course**: CST8915 Full-stack Cloud-native Development

**Semester**: Fall 2026

---

## Demo Video

🎥 [Watch Demo Video](https://youtu.be/Uhpy3TSc5Q0)

---

## Technical Explanations

### Order Service (Node.js)

The Order Service receives customer orders and puts them in a queue. It is written in Node.js using the Express framework, which makes it simple to build a small web API. It has one endpoint, `POST /orders`, and runs on port 3000. When the Store Front sends an order, the service takes the order details (product, quantity, and total price) as JSON and sends them to RabbitMQ. It uses the `amqplib` library to talk to RabbitMQ and the `cors` package so the browser is allowed to call it from the Store Front's address.

In the microservices architecture, the Order Service is the link between the website and the message broker. For each order, it connects to RabbitMQ on the same VM (`amqp://localhost`), makes sure a durable queue called `order_queue` exists, and sends the order as a persistent message, so the queue and message are kept even if RabbitMQ restarts. It uses a confirm channel, which means it waits for RabbitMQ to confirm it received the message before replying. If RabbitMQ confirms, the service replies "Order received". If something goes wrong, it replies with an error (HTTP 500). This lab does not include a service that processes the orders, so they stay in the queue.

### Product Service (Rust)

The Product Service provides the product catalog for the store. It is written in Rust, a fast and memory-safe language that works well for APIs that need to handle many requests. It uses the Warp web framework with the Tokio runtime, which lets it handle requests asynchronously. It has one endpoint, `GET /products`, which returns a JSON list of three products, each with an ID, name, and price: Dog Food ($19.99), Cat Food ($34.99), and Bird Seeds ($10.99). The products are written directly in the code, so there is no database.

In the architecture, the Product Service works on its own and does not depend on any other service. It listens on port 3030 on all network interfaces (`0.0.0.0`), so it can be reached from outside the VM. The Store Front calls it when the page loads to get the list of products to show. It uses CORS settings that allow GET requests from any origin, which is needed because the website and the API run on different ports.


### Store Front (Vue.js)

The Store Front is the website that customers use. It is built with Vue.js, a JavaScript framework for building interactive pages, and runs on port 8080 using the Vue development server (`npm run serve`). The main component, `OrderForm.vue`, shows the store name, animal images, and a list of products with radio buttons. When a customer selects a product and enters a quantity, it calculates the total price automatically (for example, 2 × Dog Food = $39.98). The Place Order button stays disabled until a product is selected and the quantity is valid.

The Store Front talks to both backend services using the browser's `fetch()` function. When the page loads, it sends a GET request to the Product Service (`/products` on port 3030) to get the products. When the customer clicks Place Order, it sends a POST request to the Order Service (`/orders` on port 3000) with the product, quantity, and total price, then shows a success or error message. Because these requests come from the customer's browser and not from the VM, the URLs in the code had to be changed from `localhost` to the VM's public IP, and ports 3000 and 3030 had to be opened in the Azure Network Security Group.

---

## Challenges and Learnings 

- **SSH setup in VS Code:** My first connection failed with the error "SSH user name cannot include the character \". I found a duplicate host entry in my SSH config file. After I cleaned up the file so it had only one entry with `HostName`, `User`, and `IdentityFile`, the connection worked.
- **What I learned:** How the services run separately but work together, and how RabbitMQ holds orders in a queue between services.

---

## Acknowledgments

- Lab instructions and source code provided by the course instructor.
-  Used ChatGPT (AI assistant) for help with setup troubleshooting.

