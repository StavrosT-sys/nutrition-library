# Integration Guide

This guide provides step-by-step instructions and code examples for integrating popular nutrition APIs into your fitness app.

## Table of Contents

1. [USDA FoodData Central](#usda-fooddata-central)
2. [Open Food Facts](#open-food-facts)
3. [Edamam Food Database](#edamam-food-database)
4. [CalorieNinjas](#calorieninjas)
5. [Multi-API Strategy](#multi-api-strategy)
6. [Caching and Optimization](#caching-and-optimization)

---

## USDA FoodData Central

### Setup

No API key required! This is the simplest API to get started with.

### JavaScript Example

```javascript
const axios = require('axios');

class USDANutritionAPI {
  constructor() {
    this.baseURL = 'https://api.nal.usda.gov/fdc/v1';
  }

  // Search for foods by name
  async searchFood(foodName, pageSize = 10) {
    try {
      const response = await axios.get(`${this.baseURL}/foods/search`, {
        params: {
          query: foodName,
          pageSize: pageSize
        }
      });
      return response.data.foods;
    } catch (error) {
      console.error('USDA API Error:', error.message);
      throw error;
    }
  }

  // Get detailed nutrition info for a specific food
  async getFoodDetails(fdcId) {
    try {
      const response = await axios.get(`${this.baseURL}/foods/${fdcId}`);
      return response.data;
    } catch (error) {
      console.error('USDA API Error:', error.message);
      throw error;
    }
  }

  // Extract nutrition data from food item
  extractNutrition(foodItem) {
    const nutrients = {};
    
    if (foodItem.foodNutrients) {
      foodItem.foodNutrients.forEach(nutrient => {
        nutrients[nutrient.nutrientName] = {
          value: nutrient.value,
          unitName: nutrient.unitName
        };
      });
    }
    
    return {
      fdcId: foodItem.fdcId,
      description: foodItem.description,
      calories: nutrients['Energy'] || nutrients['Energy (kcal)'],
      protein: nutrients['Protein'],
      carbs: nutrients['Carbohydrate, by difference'],
      fat: nutrients['Total lipid (fat)'],
      fiber: nutrients['Fiber, total dietary'],
      allNutrients: nutrients
    };
  }
}

// Usage
const usda = new USDANutritionAPI();

(async () => {
  try {
    // Search for chicken breast
    const results = await usda.searchFood('chicken breast', 5);
    console.log('Search Results:', results);

    // Get details for first result
    if (results.length > 0) {
      const details = await usda.getFoodDetails(results[0].fdcId);
      const nutrition = usda.extractNutrition(details);
      console.log('Nutrition Info:', nutrition);
    }
  } catch (error) {
    console.error('Error:', error);
  }
})();
```

### Python Example

```python
import requests

class USDANutritionAPI:
    def __init__(self):
        self.base_url = 'https://api.nal.usda.gov/fdc/v1'

    def search_food(self, food_name, page_size=10):
        """Search for foods by name"""
        params = {
            'query': food_name,
            'pageSize': page_size
        }
        response = requests.get(f'{self.base_url}/foods/search', params=params)
        response.raise_for_status()
        return response.json().get('foods', [])

    def get_food_details(self, fdc_id):
        """Get detailed nutrition info for a specific food"""
        response = requests.get(f'{self.base_url}/foods/{fdc_id}')
        response.raise_for_status()
        return response.json()

    def extract_nutrition(self, food_item):
        """Extract key nutrition data from food item"""
        nutrients = {}
        
        if 'foodNutrients' in food_item:
            for nutrient in food_item['foodNutrients']:
                nutrients[nutrient['nutrientName']] = {
                    'value': nutrient.get('value'),
                    'unitName': nutrient.get('unitName')
                }
        
        return {
            'fdc_id': food_item.get('fdcId'),
            'description': food_item.get('description'),
            'calories': nutrients.get('Energy (kcal)', {}).get('value'),
            'protein': nutrients.get('Protein', {}).get('value'),
            'carbs': nutrients.get('Carbohydrate, by difference', {}).get('value'),
            'fat': nutrients.get('Total lipid (fat)', {}).get('value'),
            'fiber': nutrients.get('Fiber, total dietary', {}).get('value'),
            'all_nutrients': nutrients
        }

# Usage
usda = USDANutritionAPI()

# Search for chicken breast
results = usda.search_food('chicken breast', 5)
print('Search Results:', results)

# Get details for first result
if results:
    details = usda.get_food_details(results[0]['fdcId'])
    nutrition = usda.extract_nutrition(details)
    print('Nutrition Info:', nutrition)
```

---

## Open Food Facts

### Setup

No signup required! Free API with no rate limits.

### JavaScript Example

```javascript
const axios = require('axios');

class OpenFoodFactsAPI {
  constructor() {
    this.baseURL = 'https://world.openfoodfacts.org/api/v0';
  }

  // Get product by barcode
  async getProductByBarcode(barcode) {
    try {
      const response = await axios.get(
        `${this.baseURL}/product/${barcode}.json`
      );
      
      if (response.data.status === 1) {
        return response.data.product;
      } else {
        throw new Error('Product not found');
      }
    } catch (error) {
      console.error('Open Food Facts Error:', error.message);
      throw error;
    }
  }

  // Search for products by name
  async searchProducts(query, pageSize = 20) {
    try {
      const response = await axios.get(`${this.baseURL}/cgi/search.pl`, {
        params: {
          search_terms: query,
          page_size: pageSize,
          json: 1
        }
      });
      return response.data.products;
    } catch (error) {
      console.error('Open Food Facts Error:', error.message);
      throw error;
    }
  }

  // Extract nutrition data
  extractNutrition(product) {
    const nutriments = product.nutriments || {};
    
    return {
      barcode: product.code,
      name: product.product_name,
      brand: product.brands,
      calories: nutriments.energy_kcal_100g,
      protein: nutriments.proteins_100g,
      carbs: nutriments.carbohydrates_100g,
      fat: nutriments.fat_100g,
      fiber: nutriments.fiber_100g,
      sugar: nutriments.sugars_100g,
      salt: nutriments.salt_100g,
      ingredients: product.ingredients_text,
      allergens: product.allergens,
      imageUrl: product.image_front_url
    };
  }
}

// Usage
const off = new OpenFoodFactsAPI();

(async () => {
  try {
    // Search by barcode (example barcode for Coca-Cola)
    const product = await off.getProductByBarcode('5000112126619');
    const nutrition = off.extractNutrition(product);
    console.log('Product Info:', nutrition);

    // Search by name
    const results = await off.searchProducts('apple', 10);
    console.log('Search Results:', results);
  } catch (error) {
    console.error('Error:', error);
  }
})();
```

### Python Example

```python
import requests

class OpenFoodFactsAPI:
    def __init__(self):
        self.base_url = 'https://world.openfoodfacts.org/api/v0'

    def get_product_by_barcode(self, barcode):
        """Get product by barcode"""
        response = requests.get(f'{self.base_url}/product/{barcode}.json')
        response.raise_for_status()
        data = response.json()
        
        if data.get('status') == 1:
            return data.get('product')
        else:
            raise ValueError('Product not found')

    def search_products(self, query, page_size=20):
        """Search for products by name"""
        params = {
            'search_terms': query,
            'page_size': page_size,
            'json': 1
        }
        response = requests.get(f'{self.base_url}/cgi/search.pl', params=params)
        response.raise_for_status()
        return response.json().get('products', [])

    def extract_nutrition(self, product):
        """Extract nutrition data from product"""
        nutriments = product.get('nutriments', {})
        
        return {
            'barcode': product.get('code'),
            'name': product.get('product_name'),
            'brand': product.get('brands'),
            'calories': nutriments.get('energy_kcal_100g'),
            'protein': nutriments.get('proteins_100g'),
            'carbs': nutriments.get('carbohydrates_100g'),
            'fat': nutriments.get('fat_100g'),
            'fiber': nutriments.get('fiber_100g'),
            'sugar': nutriments.get('sugars_100g'),
            'salt': nutriments.get('salt_100g'),
            'ingredients': product.get('ingredients_text'),
            'allergens': product.get('allergens'),
            'image_url': product.get('image_front_url')
        }

# Usage
off = OpenFoodFactsAPI()

# Get by barcode
product = off.get_product_by_barcode('5000112126619')
nutrition = off.extract_nutrition(product)
print('Product Info:', nutrition)

# Search by name
results = off.search_products('apple', 10)
print('Search Results:', results)
```

---

## Edamam Food Database

### Setup

1. Sign up at [Edamam Developer Dashboard](https://developer.edamam.com/)
2. Create a new application
3. Get your `app_id` and `app_key`

### JavaScript Example

```javascript
const axios = require('axios');

class EdamamAPI {
  constructor(appId, appKey) {
    this.baseURL = 'https://api.edamam.com/api/food-database/v2';
    this.appId = appId;
    this.appKey = appKey;
  }

  // Parse food text and get nutrition
  async parseFood(foodText) {
    try {
      const response = await axios.post(
        `${this.baseURL}/parser`,
        { ingr: foodText },
        {
          params: {
            app_id: this.appId,
            app_key: this.appKey
          }
        }
      );
      return response.data;
    } catch (error) {
      console.error('Edamam API Error:', error.message);
      throw error;
    }
  }

  // Get nutrition for specific foods
  async getNutrition(foods) {
    try {
      const response = await axios.post(
        `${this.baseURL}/nutrients`,
        { ingredients: foods },
        {
          params: {
            app_id: this.appId,
            app_key: this.appKey
          }
        }
      );
      return response.data;
    } catch (error) {
      console.error('Edamam API Error:', error.message);
      throw error;
    }
  }

  // Extract nutrition summary
  extractNutritionSummary(nutritionData) {
    const totalNutrients = nutritionData.totalNutrients || {};
    
    return {
      calories: Math.round(nutritionData.calories),
      protein: Math.round(totalNutrients.PROCNT?.quantity || 0),
      carbs: Math.round(totalNutrients.CHOCDF?.quantity || 0),
      fat: Math.round(totalNutrients.FAT?.quantity || 0),
      fiber: Math.round(totalNutrients.FIBTG?.quantity || 0),
      sugar: Math.round(totalNutrients.SUGAR?.quantity || 0),
      sodium: Math.round(totalNutrients.NA?.quantity || 0),
      cholesterol: Math.round(totalNutrients.CHOLE?.quantity || 0)
    };
  }
}

// Usage
const edamam = new EdamamAPI('YOUR_APP_ID', 'YOUR_APP_KEY');

(async () => {
  try {
    // Parse food text
    const parsed = await edamam.parseFood('1 cup cooked chicken breast');
    console.log('Parsed Food:', parsed);

    // Get nutrition for multiple foods
    const nutrition = await edamam.getNutrition([
      { quantity: 100, measureURI: 'http://www.edamam.com/ontologies/edamam.owl#Measure_gram', foodURI: 'food_...' }
    ]);
    const summary = edamam.extractNutritionSummary(nutrition);
    console.log('Nutrition Summary:', summary);
  } catch (error) {
    console.error('Error:', error);
  }
})();
```

### Python Example

```python
import requests

class EdamamAPI:
    def __init__(self, app_id, app_key):
        self.base_url = 'https://api.edamam.com/api/food-database/v2'
        self.app_id = app_id
        self.app_key = app_key

    def parse_food(self, food_text):
        """Parse food text and get nutrition"""
        params = {
            'app_id': self.app_id,
            'app_key': self.app_key
        }
        data = {'ingr': food_text}
        response = requests.post(f'{self.base_url}/parser', json=data, params=params)
        response.raise_for_status()
        return response.json()

    def get_nutrition(self, foods):
        """Get nutrition for specific foods"""
        params = {
            'app_id': self.app_id,
            'app_key': self.app_key
        }
        data = {'ingredients': foods}
        response = requests.post(f'{self.base_url}/nutrients', json=data, params=params)
        response.raise_for_status()
        return response.json()

    def extract_nutrition_summary(self, nutrition_data):
        """Extract nutrition summary"""
        total_nutrients = nutrition_data.get('totalNutrients', {})
        
        return {
            'calories': int(nutrition_data.get('calories', 0)),
            'protein': int(total_nutrients.get('PROCNT', {}).get('quantity', 0)),
            'carbs': int(total_nutrients.get('CHOCDF', {}).get('quantity', 0)),
            'fat': int(total_nutrients.get('FAT', {}).get('quantity', 0)),
            'fiber': int(total_nutrients.get('FIBTG', {}).get('quantity', 0)),
            'sugar': int(total_nutrients.get('SUGAR', {}).get('quantity', 0)),
            'sodium': int(total_nutrients.get('NA', {}).get('quantity', 0)),
            'cholesterol': int(total_nutrients.get('CHOLE', {}).get('quantity', 0))
        }

# Usage
edamam = EdamamAPI('YOUR_APP_ID', 'YOUR_APP_KEY')

# Parse food text
parsed = edamam.parse_food('1 cup cooked chicken breast')
print('Parsed Food:', parsed)

# Get nutrition
nutrition = edamam.get_nutrition([...])
summary = edamam.extract_nutrition_summary(nutrition)
print('Nutrition Summary:', summary)
```

---

## CalorieNinjas

### Setup

1. Sign up at [API Ninjas](https://api-ninjas.com/)
2. Get your free API key
3. Free tier: 10 requests/day

### JavaScript Example

```javascript
const axios = require('axios');

class CalorieNinjasAPI {
  constructor(apiKey) {
    this.baseURL = 'https://api.api-ninjas.com/v1';
    this.apiKey = apiKey;
  }

  // Get nutrition for food description
  async getNutrition(query) {
    try {
      const response = await axios.get(
        `${this.baseURL}/nutrition`,
        {
          params: { query },
          headers: { 'X-Api-Key': this.apiKey }
        }
      );
      return response.data;
    } catch (error) {
      console.error('CalorieNinjas Error:', error.message);
      throw error;
    }
  }

  // Parse multiple foods
  async parseRecipe(foods) {
    const results = [];
    for (const food of foods) {
      const nutrition = await this.getNutrition(food);
      results.push(nutrition);
    }
    return results;
  }

  // Calculate totals
  calculateTotals(nutritionArray) {
    return nutritionArray.reduce(
      (totals, item) => ({
        calories: totals.calories + (item.calories || 0),
        protein: totals.protein + (item.protein_g || 0),
        carbs: totals.carbs + (item.carbohydrates_g || 0),
        fat: totals.fat + (item.fat_total_g || 0),
        fiber: totals.fiber + (item.fiber_g || 0),
        sugar: totals.sugar + (item.sugar_g || 0)
      }),
      { calories: 0, protein: 0, carbs: 0, fat: 0, fiber: 0, sugar: 0 }
    );
  }
}

// Usage
const ninjas = new CalorieNinjasAPI('YOUR_API_KEY');

(async () => {
  try {
    // Get nutrition for single food
    const nutrition = await ninjas.getNutrition('100g chicken breast');
    console.log('Nutrition:', nutrition);

    // Parse recipe
    const recipe = await ninjas.parseRecipe([
      '2 eggs',
      '1 slice whole wheat bread',
      '1 banana'
    ]);
    const totals = ninjas.calculateTotals(recipe);
    console.log('Recipe Totals:', totals);
  } catch (error) {
    console.error('Error:', error);
  }
})();
```

### Python Example

```python
import requests

class CalorieNinjasAPI:
    def __init__(self, api_key):
        self.base_url = 'https://api.api-ninjas.com/v1'
        self.api_key = api_key

    def get_nutrition(self, query):
        """Get nutrition for food description"""
        headers = {'X-Api-Key': self.api_key}
        params = {'query': query}
        response = requests.get(f'{self.base_url}/nutrition', params=params, headers=headers)
        response.raise_for_status()
        return response.json()

    def parse_recipe(self, foods):
        """Parse multiple foods"""
        results = []
        for food in foods:
            nutrition = self.get_nutrition(food)
            results.append(nutrition)
        return results

    def calculate_totals(self, nutrition_array):
        """Calculate totals for recipe"""
        totals = {
            'calories': 0,
            'protein': 0,
            'carbs': 0,
            'fat': 0,
            'fiber': 0,
            'sugar': 0
        }
        
        for item in nutrition_array:
            totals['calories'] += item.get('calories', 0)
            totals['protein'] += item.get('protein_g', 0)
            totals['carbs'] += item.get('carbohydrates_g', 0)
            totals['fat'] += item.get('fat_total_g', 0)
            totals['fiber'] += item.get('fiber_g', 0)
            totals['sugar'] += item.get('sugar_g', 0)
        
        return totals

# Usage
ninjas = CalorieNinjasAPI('YOUR_API_KEY')

# Get nutrition
nutrition = ninjas.get_nutrition('100g chicken breast')
print('Nutrition:', nutrition)

# Parse recipe
recipe = ninjas.parse_recipe(['2 eggs', '1 slice whole wheat bread', '1 banana'])
totals = ninjas.calculate_totals(recipe)
print('Recipe Totals:', totals)
```

---

## Multi-API Strategy

### Fallback Pattern

Use multiple APIs with automatic fallback:

```javascript
class NutritionAPIManager {
  constructor(apis) {
    this.apis = apis; // Array of API instances
  }

  async searchFood(query) {
    for (const api of this.apis) {
      try {
        const result = await api.search(query);
        if (result && result.length > 0) {
          return { result, source: api.name };
        }
      } catch (error) {
        console.warn(`${api.name} failed, trying next...`);
        continue;
      }
    }
    throw new Error('All APIs failed');
  }
}

// Usage
const manager = new NutritionAPIManager([
  new USDANutritionAPI(),
  new OpenFoodFactsAPI(),
  new EdamamAPI(appId, appKey)
]);

const result = await manager.searchFood('chicken breast');
```

---

## Caching and Optimization

### Local Cache Pattern

```javascript
class CachedNutritionAPI {
  constructor(api, cacheDir = './cache') {
    this.api = api;
    this.cacheDir = cacheDir;
    this.cache = new Map();
  }

  getCacheKey(query) {
    return query.toLowerCase().replace(/\s+/g, '_');
  }

  async searchFood(query) {
    const cacheKey = this.getCacheKey(query);
    
    // Check memory cache
    if (this.cache.has(cacheKey)) {
      console.log('Cache hit:', cacheKey);
      return this.cache.get(cacheKey);
    }

    // Query API
    const result = await this.api.searchFood(query);
    
    // Store in cache
    this.cache.set(cacheKey, result);
    
    return result;
  }

  clearCache() {
    this.cache.clear();
  }
}
```

---

**Next Steps:**
- See [best-practices.md](./best-practices.md) for optimization tips
- Check [calculations.md](./calculations.md) for calorie burn formulas
- Review [data schemas](../schemas/) for standardized formats

---

**Last Updated:** January 2026  
**Maintained By:** Nutrition Library Team
