# How to Connect Your App to a Free Nutrition Library (The Easy Way)

Hello Team!

Great news! You **do not** need to build your own food database from scratch. We can connect our app to a massive, free, and professionally maintained nutrition library over the internet. This is done using something called an **API**.

This guide will show you exactly how to do that. We will build a simple feature that lets a user search for a food, and our app will automatically fetch the nutritional information for it.

**Our Goal:** To build a simple food search tool that:
1.  Provides a search box for the user to type a food (e.g., "apple" or "chicken breast").
2.  Connects to a free external nutrition library (the USDA FoodData Central API).
3.  Fetches the nutrition data for that food.
4.  Displays the results to the user in a clean, simple list.

We will provide a complete, working HTML file with all the code you need to make this happen.

---

## What is an API? (And Why It's Awesome for Us)

Think of an API (Application Programming Interface) like a waiter in a restaurant.

*   **Your App** is the customer.
*   The **External Nutrition Library** is the kitchen, which has all the food data.
*   The **API** is the waiter.

Your app (the customer) doesn't go into the kitchen. Instead, it gives an order to the waiter (the API). The waiter takes the order to the kitchen, gets the food (the data), and brings it back to your app.

**Why is this great?**
*   **It's Free:** We will use the official U.S. Department of Agriculture (USDA) food database, which is free and requires no complicated sign-up or API key.
*   **It's Huge:** It has hundreds of thousands of foods, from raw ingredients to branded products.
*   **It's Always Updated:** The USDA manages the data, so we always get accurate, up-to-date information without any work on our end.

---

## The 3 Steps to Integrating the API

Here is the simple process we will follow:

1.  **Step 1: Create the Search Bar** - A simple HTML input box for the user to type in.
2.  **Step 2: Write the JavaScript to Talk to the API** - This is the core part. We'll write a function that sends the user's search query to the USDA API.
3.  **Step 3: Display the Results** - Once the API sends the data back, we'll format it nicely and show it to the user.

---

## Step 1: Create the Search Bar

This is a standard HTML form. The user will type the food they want to look up here.

### The Code: HTML Search Form

```html
<div id="food-search-section">
    <h2>Search for a Food</h2>
    <input type="text" id="food-search-input" placeholder="e.g., banana, grilled cheese...">
    <button id="search-btn">Search</button>
</div>

<!-- This is where we will display the results from the API -->
<div id="results-section">
    <h3>Results</h3>
    <ul id="food-results-list"></ul>
</div>
```

**How it works:**
*   We have an `<input>` with the `id` **`food-search-input`**. This is where the user types.
*   We have a `<button>` with the `id` **`search-btn`**. We will make this button run our search function when clicked.
*   We have an empty list (`<ul>`) with the `id` **`food-results-list`**. This is where we will put the results that the API sends back.

---

## Step 2: Write the JavaScript to Talk to the API

This is where the magic happens. We will use JavaScript's `fetch` function, which is the modern way to make API requests.

### The Code: JavaScript API Function

This function builds the correct URL for the USDA API, sends the request, and gets the data.

```javascript
// This function sends a search request to the USDA API
async function searchFood(foodName) {
    // The base URL for the USDA API search endpoint
    const apiEndpoint = 'https://api.nal.usda.gov/fdc/v1/foods/search';
    
    // The API requires a special key, but for the demo we can use the free DEMO_KEY
    const apiKey = 'DEMO_KEY'; 

    // We build the full URL with the foodName and our API key
    // The `encodeURIComponent` part makes sure the food name is safe for a URL
    const url = `${apiEndpoint}?query=${encodeURIComponent(foodName)}&api_key=${apiKey}`;

    try {
        // We use `fetch` to make the API call
        const response = await fetch(url);
        
        // We convert the response into a format JavaScript can easily use (JSON)
        const data = await response.json();
        
        // The actual food items are inside a property called 'foods'
        return data.foods;

    } catch (error) {
        // If something goes wrong (like no internet), we log an error
        console.error("Error fetching food data:", error);
        return []; // Return an empty list on error
    }
}
```

**How it works:**
1.  We define the `url` for the USDA API.
2.  We include the `foodName` the user searched for as a `query` parameter.
3.  We use `await fetch(url)` to send the request and wait for the response.
4.  `await response.json()` converts the raw response into a usable JavaScript object.
5.  We return `data.foods`, which is a list of all the foods that matched the search.

---

## Step 3: Display the Results

Once we have the list of foods from the API, we need to show them to the user. We'll create a function that takes this list and turns it into simple HTML.

### The Code: JavaScript Display Function

```javascript
// This function takes a list of foods and displays them on the page
function displayResults(foods) {
    const resultsList = document.getElementById('food-results-list');
    
    // First, we clear any old results
    resultsList.innerHTML = '';

    if (!foods || foods.length === 0) {
        resultsList.innerHTML = '<li>No results found. Try another search.</li>';
        return;
    }

    // We loop through each food item in the list
    foods.forEach(food => {
        // For each food, we find the calorie information
        // The nutrient data is in a list called 'foodNutrients'
        const calories = food.foodNutrients.find(n => n.nutrientName === 'Energy')?.value || 'N/A';

        // We create a new list item (<li>) to display the food's info
        const listItem = document.createElement('li');
        listItem.textContent = `${food.description} - ${calories} kcal`;
        
        // We add the new list item to our results list on the page
        resultsList.appendChild(listItem);
    });
}
```

**How it works:**
1.  It finds the `food-results-list` on our HTML page.
2.  It clears out any old results.
3.  It loops through the `foods` array we got from the API.
4.  For each `food`, it finds the 'Energy' nutrient (which is calories).
5.  It creates a new HTML list item (`<li>`) with the food's description and calories.
6.  It adds this new item to the list on the page.

---

## Putting It All Together: A Complete, Working Example

Talk is cheap! I have created a **single, complete HTML file** that contains all the HTML and JavaScript needed for a working prototype. Your team can open this file, see it work, and copy the code directly into your project.

This file is the best way to learn because you can experiment with it.

**You can find this file in the repository here:**
*   **[api-integration-example.html](./api-integration-example.html)**

### How to Use the Example File:

1.  **Download it** from the repository.
2.  **Open it in a web browser** (like Chrome, Firefox, or Safari). No server needed!
3.  You will see a simple page with a search bar.
4.  **Type a food** (e.g., "scrambled eggs") and click "Search".
5.  Watch as the results from the USDA database appear on the page instantly!

This example is the blueprint. Your team can now take this proven code and integrate it into your app's design. They can expand it to show more nutrients (like protein, fat, and carbs) by following the same pattern used for calories.

This is the standard way modern apps handle large datasets, and it's a powerful skill for your team to have.

Good luck!
