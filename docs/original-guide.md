# Building a Food Nutrition Library for Your Fitness App

Developing a fitness app that tracks calories consumed (via food intake) and spent (via exercise) is a great idea—it's a core feature in apps like MyFitnessPal or Lose It!. To handle the food side, you'll need a reliable database of foods with nutritional values (e.g., calories, macros like protein/carbs/fats, micronutrients). This can be integrated via APIs for real-time queries or bulk downloads for offline use. For calories spent, you can use standard metabolic formulas or integrate activity-tracking APIs.

This document outlines top options for food databases/APIs, focusing on free and low-cost ones first, based on popularity, coverage, and ease of integration. These support searching by food name, barcode, or ingredients, and return detailed nutrition data. Many are developer-friendly with SDKs for iOS/Android/web.

## Free Food Nutrition Databases and APIs

These are ideal for startups or MVPs since they're cost-free, though they may have usage limits or require attribution.

### USDA FoodData Central

A government-run database with over 380,000 foods, including branded items and raw ingredients. It provides detailed nutrient profiles (calories, macros, vitamins) validated by US standards. Access via REST API—no API key needed, unlimited queries. Great for US/Canada-focused apps, as it includes common North American products. You can query by food name or ID, and it's updated monthly.

*   **Integration Tip:** Use their API endpoint (e.g., `https://api.nal.usda.gov/fdc/v1/foods/search`) to fetch data. Download the full dataset as CSV for offline caching if your app needs it.
*   **Pros:** Accurate, free, no limits.
*   **Cons:** US-centric; lacks advanced features like recipe analysis.

### Open Food Facts

An open-source, crowdsourced database with millions of global products (focus on Europe/North America). Includes nutrition facts, ingredients, allergens, and barcodes. Free API with no signup required.

*   **Integration Tip:** Query via `https://world.openfoodfacts.org/api/v0/product/[barcode].json`. It's collaborative, so data quality varies but improves over time.
*   **Pros:** Global coverage, open-source (you can contribute or fork).
*   **Cons:** Less polished than commercial options; potential inaccuracies in user-submitted data.

### CalorieNinjas (via API Ninjas)

Free API for 100,000+ foods/beverages, extracting nutrition from text (e.g., "1 apple + 2 eggs"). Supports natural language queries for easy user input.

*   **Integration Tip:** Sign up for a free key; endpoints like `/v1/nutrition?query=100g+chicken` return calories, macros, etc. Good for recipe parsing.
*   **Pros:** Simple, fast setup (under 10 minutes).
*   **Cons:** Migrating to API Ninjas platform; free tier has limits.

## Commercial/Paid APIs (Scalable for Production Apps)

If you need more features like AI image recognition (e.g., scan food photos), global coverage, or dedicated support, these start free/low-cost and scale with usage.

| API Name                  | Key Features                                                              | Pricing                                                    | Coverage                      | Best For                                                  |
| ------------------------- | ------------------------------------------------------------------------- | ---------------------------------------------------------- | ----------------------------- | --------------------------------------------------------- |
| **Edamam Food Database API** | 900,000+ foods, 680,000+ UPC codes; recipe analysis, nutrition breakdown. | Free tier (up to 10K calls/month); paid from $0.01/call.      | Global, with strong US/EU data. | Apps needing barcode scanning or meal planning.           |
| **FatSecret Platform API**  | 1.9M+ verified foods across 56 countries; includes allergens, images, recipes. | Free tier; paid for bulk access (contact for quotes).        | Highly global, 24 languages.  | International apps with exercise/diary integration.       |
| **Nutritionix API**         | Largest verified database (millions of items); geolocation for restaurants, barcode support. | Free tier (limited); paid from $99/month.                  | US-focused but expanding; 206K+ restaurants. | Restaurant integration or branded foods.                  |
| **Passio Nutrition-AI**     | 2.5M+ foods; AI for photo recognition, REST API + SDK.                      | Contact for pricing; free trial.                           | Global brands/recipes.        | Apps with camera-based logging.                           |
| **LogMeal Nutritional API** | Extracts nutrition from photos; macro/micro details, no manual input.     | Paid (usage-based); free demo.                             | Focus on meals/images.        | Wellness apps with AI features.                           |

**Other Mentions:** Spoonacular (recipe-focused), Chomp (branded products), or ESHA for premium data. Start with their docs for SDKs—most support JSON responses for easy parsing in your app.

## Handling Calories Consumed and Spent

### Consumed (Food Intake)

Use the above APIs to sum calories from user-logged foods. For example, query "100g chicken breast" to get ~165 calories, then aggregate daily totals.

### Spent (Exercise)

This isn't directly in food databases but can be calculated using:

*   **MET Formulas:** Multiply Metabolic Equivalent of Task (MET) values by weight/time (e.g., running = 7 METs). Free MET lists from sources like the Compendium of Physical Activities.
*   **Integrations:** Use Google Fit REST API for activity data (tracks steps, workouts, and estimates calories burned). Or combine with wearables: Fitbit API for biometrics, Hevy for strength training, Cronometer for nutrition syncing.
*   **Example Calculation:** Daily burn = BMR (Harris-Benedict formula: For men, 88.362 + (13.397 × weight kg) + (4.799 × height cm) - (5.677 × age)) + activity calories.

## Implementation Tips

*   **Start Simple:** Prototype with USDA's free API—fetch data on app load or search.
*   **User Experience:** Add search autocomplete, barcode scanning (via device camera + API), and offline mode (cache popular foods).
*   **Legal/Compliance:** Ensure data attribution (e.g., "Powered by USDA"). For Canada, check Health Canada guidelines on nutrition labeling.
*   **Scaling:** If your app grows, switch to paid APIs for reliability and features like AI (e.g., photo-to-nutrition).
*   **Development Resources:** Check Reddit threads for dev experiences or GitHub lists for open-source trackers.

If this is for a specific platform (e.g., iOS, Android) or you need code examples, let me know for more tailored advice!
