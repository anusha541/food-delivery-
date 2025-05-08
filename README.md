Create a data.json file that will hold information about restaurants and their menus. This will simulate a database.
{
  "restaurants": [
    {
      "id": 1,
      "name": "Pizza Palace",
      "menu": [
        { "id": 1, "name": "Margherita", "price": 9.99 },
        { "id": 2, "name": "Pepperoni", "price": 12.99 },
        { "id": 3, "name": "BBQ Chicken", "price": 14.99 }
      ]
    },
    {
      "id": 2,
      "name": "Sushi Spot",
      "menu": [
        { "id": 1, "name": "California Roll", "price": 7.99 },
        { "id": 2, "name": "Spicy Tuna", "price": 9.99 },
        { "id": 3, "name": "Tempura Roll", "price": 11.99 }
      ]
    }
  ]
}
Step 3: Create HTML Files
index.html
This will display the list of restaurants on the homepage.
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Food Delivery</title>
</head>
<body>
  <h1>Food Delivery Service</h1>
  <h2>Available Restaurants</h2>
  <ul id="restaurant-list">
    <!-- Restaurant list will be dynamically inserted here -->
  </ul>

  <script>
    fetch('/restaurants')
      .then(response => response.json())
      .then(data => {
        const restaurantList = document.getElementById('restaurant-list');
        data.restaurants.forEach(restaurant => {
          const li = document.createElement('li');
          li.innerHTML = `<a href="/order?id=${restaurant.id}">${restaurant.name}</a>`;
          restaurantList.appendChild(li);
        });
      });
  </script>
</body>
</html>
order.html
This page will display the menu for a selected restaurant.
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Order Food</title>
</head>
<body>
  <h1>Restaurant Menu</h1>
  <h2 id="restaurant-name"></h2>
  <ul id="menu-list">
    <!-- Menu items will be dynamically inserted here -->
  </ul>

  <h3>Place your order</h3>
  <form id="order-form">
    <label for="food-id">Select Food:</label>
    <select id="food-id" name="food-id">
      <!-- Food options will be dynamically inserted here -->
    </select>
    <br><br>
    <button type="submit">Place Order</button>
  </form>

  <div id="order-summary"></div>

  <script>
    const urlParams = new URLSearchParams(window.location.search);
    const restaurantId = urlParams.get('id');

    fetch(`/restaurant/${restaurantId}`)
      .then(response => response.json())
      .then(data => {
        document.getElementById('restaurant-name').textContent = data.name;

        const menuList = document.getElementById('menu-list');
        const foodSelect = document.getElementById('food-id');

        data.menu.forEach(food => {
          const li = document.createElement('li');
          li.textContent = `${food.name} - $${food.price}`;
          menuList.appendChild(li);

          const option = document.createElement('option');
          option.value = food.id;
          option.textContent = `${food.name} - $${food.price}`;
          foodSelect.appendChild(option);
        });

        const form = document.getElementById('order-form');
        form.addEventListener('submit', (event) => {
          event.preventDefault();
          const foodId = foodSelect.value;

          fetch('/order', {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify({ restaurantId, foodId })
          })
            .then(response => response.json())
            .then(order => {
              document.getElementById('order-summary').textContent = `Your order for ${order.food.name} has been placed! Total: $${order.food.price}`;
            });
        });
      });
  </script>
</body>
</html>
Step 4: Set Up the Server (app.js)
Now, let's set up the server using Express to handle the routing and HTTP requests.
const express = require('express');
const bodyParser = require('body-parser');
const path = require('path');
const fs = require('fs');

const app = express();
const PORT = 3000;

// Parse incoming JSON requests
app.use(bodyParser.json());

// Serve static files (HTML, CSS, etc.)
app.use(express.static(path.join(__dirname, 'public')));

// Load restaurant data
const data = JSON.parse(fs.readFileSync('data.json', 'utf8'));

// Route to get the list of restaurants
app.get('/restaurants', (req, res) => {
  res.json(data);
});

// Route to get a single restaurant with menu items
app.get('/restaurant/:id', (req, res) => {
  const restaurant = data.restaurants.find(r => r.id == req.params.id);
  if (restaurant) {
    res.json(restaurant);
  } else {
    res.status(404).send('Restaurant not found');
  }
});

// Route to handle food order
app.post('/order', (req, res) => {
  const { restaurantId, foodId } = req.body;
  const restaurant = data.restaurants.find(r => r.id == restaurantId);
  if (restaurant) {
    const food = restaurant.menu.find(f => f.id == foodId);
    if (food) {
      // For simplicity, we just send a basic confirmation message.
      res.json({ food });
    } else {
      res.status(404).send('Food item not found');
    }
  } else {
    res.status(404).send('Restaurant not found');
  }
});

// Start the server
app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
