## Project Setup
    ```
    npm install
    ```

## Run
    ```
    npm run dev
    ```

Follow these steps to import the database using phpMyAdmin:

## Prerequisites

Make sure you have XAMPP installed on your system with Apache and MySQL services.

## Question

1. Design a relational database to store all the information contained in the above images such as products, addresses, stores, categories, orders, users, …. And describe the types of database normalization you used.:

   ```
   Start XAMPP Control Panel.
   Enable Apache and MySQL.
   Open a web browser and go to: http://localhost/phpmyadmin

   Navigate to your project directory and run the development server:
   -> npm run dev
   The database tables will be automatically generated based on the defined entities in your project.
   ```

2. User "assessment", with information as shown, purchased the product "KAPPA Women's Sneakers" in yellow, size 36, quantity 1. Please write a query to insert the order that this person purchased database:

   ```
   In phpMyAdmin, click on New to create a new database.
   Name the database "khoaluan".
   ```

3. Write a query to calculate the average order value (total price of items in an order) for each month in the current year.:

   ```
   SELECT 
      DATE_FORMAT(createdAt, '%Y-%m') AS month,
      ROUND(AVG(totalAmount), 2) AS average_order_value,
      COUNT(id) AS order_count
    FROM \`order\`
    WHERE 
      YEAR(createdAt) = YEAR(CURDATE())
    GROUP BY month
    ORDER BY month ASC;
   ```

4. Write a query to calculate the churn rate of customers. The churn rate is defined as the percentage of customers who did not make a purchase in the last 6 months but had made a purchase in the 6 months prior to that:

   ```
   SELECT
        total.total_users AS total_previous_customers,
        COUNT(DISTINCT prev.user_id) AS churned_customers,
        ROUND(
          COUNT(DISTINCT prev.user_id) * 100.0 / NULLIF(total.total_users, 0),
          2
        ) AS churn_rate_percentage
      FROM (
        SELECT DISTINCT user_id
        FROM \`order\`
        WHERE createdAt >= DATE_SUB(CURDATE(), INTERVAL 12 MONTH)
          AND createdAt < DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
      ) AS prev
      LEFT JOIN (
        SELECT DISTINCT user_id
        FROM \`order\`
        WHERE createdAt >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
      ) AS recent
        ON prev.user_id = recent.user_id
      JOIN (
        SELECT COUNT(DISTINCT user_id) AS total_users
        FROM \`order\`
        WHERE createdAt >= DATE_SUB(CURDATE(), INTERVAL 12 MONTH)
          AND createdAt < DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
      ) AS total
      WHERE recent.user_id IS NULL;
    `
   ```

5. Fetches a list of all product categories available in the e-commerce platform

   ```
   Service:
    async getAllCategories(): Promise<Category[]> {
    return await this.categoryRepository.find();
    }

   API: GET-http://localhost:3002/api/category
   OUTPUT: 
   [
      {
         "id": 1,
         "name": "Sneakers",
         "description": "Sneakers"
      },
      {
         "id": 2,
         "name": "Dép",
         "description": "Dép cho nam"
      }
   ]
   ```

6. Fetches a list of products that belong to a specific category.

   ```
   Service:
   async getProductsByCategory(categoryId: number): Promise<Product[]> {
    const category = await this.categoryRepository.findOne({
      where: { id: categoryId },
      relations: ['products'],
    });
    if (!category) {
      throw new Error('Category not found');
    }
    return category.products;
    }

   API: GET-http://localhost:3002/api/product/category/:categoryId
   ```

7. Creates a new order and processes payment.

   ```
   Service:
    async createOrders(data: OrderDTO): Promise<Order> {
    // 1. Kiểm tra người dùng
    const user = await this.userRepository.findOneBy({ id: data.userId });
    if (!user) throw new Error('Người dùng không tồn tại');

    // 2. Lấy các orderItems (giỏ hàng) của user
    const orderItems = await this.orderItemRepository.find({
      where: { user: { id: data.userId } },
      relations: ['product']
    });

    if (orderItems.length === 0) {
      throw new Error('Không có sản phẩm trong giỏ hàng');
    }

    // 3. Tính tổng tiền
    const totalAmount = orderItems.reduce((sum, item) => {
      return sum + item.priceAtPurchase;
    }, 0);

    // 4. Tạo order
    const order = this.orderRepository.create({
      user,
      totalAmount,
      status: 'pending'
    });

    const savedOrder = await this.orderRepository.save(order);
    // Gửi email xác nhận
      await this.emailService.sendOrderConfirmationEmail(user, savedOrder);
    return savedOrder;
    }

   API: POST-http://localhost:3002/api/order/add
   ```

8. Send order confirmation email to user (processed asynchronously with order creation flow).

   ```
    async sendOrderConfirmationEmail(user: User, order: Order) {
    const htmlContent = orderConfirmationTemplate(user, order);

    const mailOptions = {
      from: emailConfig.auth.user,
      to: user.email,
      subject: 'Xác nhận đơn hàng của bạn',
      html: htmlContent,
    };

    await this.transporter.sendMail(mailOptions);
  }
   ```

