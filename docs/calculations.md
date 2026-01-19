# Nutrition Calculations and Formulas

This document provides formulas and implementations for calculating calories burned, BMR, macros, and other nutrition metrics.

## Table of Contents

1. [Basal Metabolic Rate (BMR)](#basal-metabolic-rate-bmr)
2. [Total Daily Energy Expenditure (TDEE)](#total-daily-energy-expenditure-tdee)
3. [Activity Calories (MET-Based)](#activity-calories-met-based)
4. [Macro Calculations](#macro-calculations)
5. [Calorie Deficit/Surplus](#calorie-deficitsurplus)

---

## Basal Metabolic Rate (BMR)

BMR is the number of calories your body burns at rest. This is the foundation for calculating total daily calorie burn.

### Harris-Benedict Formula (Revised)

This is the most commonly used formula for fitness apps.

#### For Men

```
BMR = 88.362 + (13.397 × weight_kg) + (4.799 × height_cm) - (5.677 × age_years)
```

#### For Women

```
BMR = 447.593 + (9.247 × weight_kg) + (3.098 × height_cm) - (4.330 × age_years)
```

### Implementation

#### JavaScript

```javascript
class BMRCalculator {
  static calculateHarrisBenedict(gender, weight_kg, height_cm, age_years) {
    if (gender.toLowerCase() === 'male') {
      return 88.362 + 
             (13.397 * weight_kg) + 
             (4.799 * height_cm) - 
             (5.677 * age_years);
    } else {
      return 447.593 + 
             (9.247 * weight_kg) + 
             (3.098 * height_cm) - 
             (4.330 * age_years);
    }
  }

  static calculateMifflinStJeor(gender, weight_kg, height_cm, age_years) {
    // Alternative formula (slightly more accurate)
    const baseBMR = (10 * weight_kg) + (6.25 * height_cm) - (5 * age_years);
    
    if (gender.toLowerCase() === 'male') {
      return baseBMR + 5;
    } else {
      return baseBMR - 161;
    }
  }
}

// Usage
const bmr = BMRCalculator.calculateHarrisBenedict('male', 80, 180, 30);
console.log(`BMR: ${bmr.toFixed(0)} calories/day`);
```

#### Python

```python
class BMRCalculator:
    @staticmethod
    def calculate_harris_benedict(gender, weight_kg, height_cm, age_years):
        """Calculate BMR using Harris-Benedict formula"""
        if gender.lower() == 'male':
            return (88.362 + 
                   (13.397 * weight_kg) + 
                   (4.799 * height_cm) - 
                   (5.677 * age_years))
        else:
            return (447.593 + 
                   (9.247 * weight_kg) + 
                   (3.098 * height_cm) - 
                   (4.330 * age_years))

    @staticmethod
    def calculate_mifflin_st_jeor(gender, weight_kg, height_cm, age_years):
        """Calculate BMR using Mifflin-St Jeor formula (more accurate)"""
        base_bmr = (10 * weight_kg) + (6.25 * height_cm) - (5 * age_years)
        
        if gender.lower() == 'male':
            return base_bmr + 5
        else:
            return base_bmr - 161

# Usage
bmr = BMRCalculator.calculate_harris_benedict('male', 80, 180, 30)
print(f'BMR: {bmr:.0f} calories/day')
```

---

## Total Daily Energy Expenditure (TDEE)

TDEE is the total calories burned in a day, including BMR plus activity.

### Activity Multiplier Method

```
TDEE = BMR × Activity Factor
```

| Activity Level | Factor | Description |
|---|---|---|
| Sedentary | 1.2 | Little or no exercise |
| Lightly Active | 1.375 | Exercise 1-3 days/week |
| Moderately Active | 1.55 | Exercise 3-5 days/week |
| Very Active | 1.725 | Exercise 6-7 days/week |
| Extremely Active | 1.9 | Physical job or training twice/day |

### Implementation

#### JavaScript

```javascript
class TDEECalculator {
  static calculateTDEE(bmr, activityLevel) {
    const activityFactors = {
      'sedentary': 1.2,
      'lightly_active': 1.375,
      'moderately_active': 1.55,
      'very_active': 1.725,
      'extremely_active': 1.9
    };

    const factor = activityFactors[activityLevel] || 1.55;
    return bmr * factor;
  }

  static getActivityLevel(exerciseDaysPerWeek) {
    if (exerciseDaysPerWeek === 0) return 'sedentary';
    if (exerciseDaysPerWeek <= 1) return 'lightly_active';
    if (exerciseDaysPerWeek <= 3) return 'moderately_active';
    if (exerciseDaysPerWeek <= 5) return 'very_active';
    return 'extremely_active';
  }
}

// Usage
const bmr = 1700;
const tdee = TDEECalculator.calculateTDEE(bmr, 'moderately_active');
console.log(`TDEE: ${tdee.toFixed(0)} calories/day`);
```

#### Python

```python
class TDEECalculator:
    ACTIVITY_FACTORS = {
        'sedentary': 1.2,
        'lightly_active': 1.375,
        'moderately_active': 1.55,
        'very_active': 1.725,
        'extremely_active': 1.9
    }

    @staticmethod
    def calculate_tdee(bmr, activity_level):
        """Calculate TDEE based on BMR and activity level"""
        factor = TDEECalculator.ACTIVITY_FACTORS.get(activity_level, 1.55)
        return bmr * factor

    @staticmethod
    def get_activity_level(exercise_days_per_week):
        """Determine activity level from exercise frequency"""
        if exercise_days_per_week == 0:
            return 'sedentary'
        elif exercise_days_per_week <= 1:
            return 'lightly_active'
        elif exercise_days_per_week <= 3:
            return 'moderately_active'
        elif exercise_days_per_week <= 5:
            return 'very_active'
        else:
            return 'extremely_active'

# Usage
bmr = 1700
tdee = TDEECalculator.calculate_tdee(bmr, 'moderately_active')
print(f'TDEE: {tdee:.0f} calories/day')
```

---

## Activity Calories (MET-Based)

MET (Metabolic Equivalent of Task) represents the intensity of physical activity.

```
Calories Burned = MET × Weight (kg) × Duration (hours)
```

### Common MET Values

| Activity | MET | Intensity |
|---|---|---|
| Walking (3 mph) | 3.5 | Light |
| Walking (4 mph) | 5.0 | Moderate |
| Jogging (5 mph) | 8.3 | Vigorous |
| Running (6 mph) | 9.8 | Very Vigorous |
| Running (8 mph) | 11.0 | Very Vigorous |
| Cycling (10 mph) | 6.0 | Moderate |
| Cycling (16 mph) | 12.0 | Vigorous |
| Swimming (leisurely) | 6.0 | Moderate |
| Swimming (vigorous) | 11.0 | Vigorous |
| Weight Training | 6.0 | Moderate |
| HIIT Training | 12.0 | Very Vigorous |
| Yoga | 2.5 | Light |
| Pilates | 3.0 | Light |

### Implementation

#### JavaScript

```javascript
class METCalculator {
  static metValues = {
    'walking_3mph': 3.5,
    'walking_4mph': 5.0,
    'jogging_5mph': 8.3,
    'running_6mph': 9.8,
    'running_8mph': 11.0,
    'cycling_10mph': 6.0,
    'cycling_16mph': 12.0,
    'swimming_leisurely': 6.0,
    'swimming_vigorous': 11.0,
    'weight_training': 6.0,
    'hiit': 12.0,
    'yoga': 2.5,
    'pilates': 3.0,
    'basketball': 8.0,
    'tennis': 8.0,
    'soccer': 10.0
  };

  static calculateCaloriesBurned(activity, weight_kg, duration_minutes) {
    const met = this.metValues[activity];
    if (!met) {
      throw new Error(`Unknown activity: ${activity}`);
    }

    const duration_hours = duration_minutes / 60;
    return met * weight_kg * duration_hours;
  }

  static calculateDuration(activity, weight_kg, targetCalories) {
    const met = this.metValues[activity];
    if (!met) {
      throw new Error(`Unknown activity: ${activity}`);
    }

    const duration_hours = targetCalories / (met * weight_kg);
    return duration_hours * 60; // Convert to minutes
  }
}

// Usage
const caloriesBurned = METCalculator.calculateCaloriesBurned('running_6mph', 80, 30);
console.log(`Calories burned: ${caloriesBurned.toFixed(0)}`);

const durationNeeded = METCalculator.calculateDuration('walking_4mph', 80, 300);
console.log(`Duration needed: ${durationNeeded.toFixed(0)} minutes`);
```

#### Python

```python
class METCalculator:
    MET_VALUES = {
        'walking_3mph': 3.5,
        'walking_4mph': 5.0,
        'jogging_5mph': 8.3,
        'running_6mph': 9.8,
        'running_8mph': 11.0,
        'cycling_10mph': 6.0,
        'cycling_16mph': 12.0,
        'swimming_leisurely': 6.0,
        'swimming_vigorous': 11.0,
        'weight_training': 6.0,
        'hiit': 12.0,
        'yoga': 2.5,
        'pilates': 3.0,
        'basketball': 8.0,
        'tennis': 8.0,
        'soccer': 10.0
    }

    @staticmethod
    def calculate_calories_burned(activity, weight_kg, duration_minutes):
        """Calculate calories burned for an activity"""
        if activity not in METCalculator.MET_VALUES:
            raise ValueError(f'Unknown activity: {activity}')
        
        met = METCalculator.MET_VALUES[activity]
        duration_hours = duration_minutes / 60
        return met * weight_kg * duration_hours

    @staticmethod
    def calculate_duration(activity, weight_kg, target_calories):
        """Calculate duration needed to burn target calories"""
        if activity not in METCalculator.MET_VALUES:
            raise ValueError(f'Unknown activity: {activity}')
        
        met = METCalculator.MET_VALUES[activity]
        duration_hours = target_calories / (met * weight_kg)
        return duration_hours * 60  # Convert to minutes

# Usage
calories_burned = METCalculator.calculate_calories_burned('running_6mph', 80, 30)
print(f'Calories burned: {calories_burned:.0f}')

duration_needed = METCalculator.calculate_duration('walking_4mph', 80, 300)
print(f'Duration needed: {duration_needed:.0f} minutes')
```

---

## Macro Calculations

### Macro Ratios

Common macro ratios for different goals:

| Goal | Protein | Carbs | Fat |
|---|---|---|---|
| Weight Loss | 35% | 45% | 20% |
| Muscle Gain | 30% | 45% | 25% |
| Balanced | 25% | 50% | 25% |
| Keto | 25% | 5% | 70% |
| Endurance | 15% | 65% | 20% |

### Implementation

#### JavaScript

```javascript
class MacroCalculator {
  static macroRatios = {
    'weight_loss': { protein: 0.35, carbs: 0.45, fat: 0.20 },
    'muscle_gain': { protein: 0.30, carbs: 0.45, fat: 0.25 },
    'balanced': { protein: 0.25, carbs: 0.50, fat: 0.25 },
    'keto': { protein: 0.25, carbs: 0.05, fat: 0.70 },
    'endurance': { protein: 0.15, carbs: 0.65, fat: 0.20 }
  };

  static calculateMacros(tdee, goal) {
    const ratios = this.macroRatios[goal] || this.macroRatios['balanced'];

    return {
      protein_grams: Math.round((tdee * ratios.protein) / 4),
      carbs_grams: Math.round((tdee * ratios.carbs) / 4),
      fat_grams: Math.round((tdee * ratios.fat) / 9),
      protein_calories: Math.round(tdee * ratios.protein),
      carbs_calories: Math.round(tdee * ratios.carbs),
      fat_calories: Math.round(tdee * ratios.fat)
    };
  }

  static calculateFromGrams(protein_g, carbs_g, fat_g) {
    return {
      protein_calories: protein_g * 4,
      carbs_calories: carbs_g * 4,
      fat_calories: fat_g * 9,
      total_calories: (protein_g * 4) + (carbs_g * 4) + (fat_g * 9)
    };
  }
}

// Usage
const macros = MacroCalculator.calculateMacros(2000, 'muscle_gain');
console.log('Daily Macros:', macros);
// Output: { protein_grams: 150, carbs_grams: 225, fat_grams: 56, ... }
```

#### Python

```python
class MacroCalculator:
    MACRO_RATIOS = {
        'weight_loss': {'protein': 0.35, 'carbs': 0.45, 'fat': 0.20},
        'muscle_gain': {'protein': 0.30, 'carbs': 0.45, 'fat': 0.25},
        'balanced': {'protein': 0.25, 'carbs': 0.50, 'fat': 0.25},
        'keto': {'protein': 0.25, 'carbs': 0.05, 'fat': 0.70},
        'endurance': {'protein': 0.15, 'carbs': 0.65, 'fat': 0.20}
    }

    @staticmethod
    def calculate_macros(tdee, goal):
        """Calculate daily macro targets"""
        ratios = MacroCalculator.MACRO_RATIOS.get(goal, MacroCalculator.MACRO_RATIOS['balanced'])

        return {
            'protein_grams': int((tdee * ratios['protein']) / 4),
            'carbs_grams': int((tdee * ratios['carbs']) / 4),
            'fat_grams': int((tdee * ratios['fat']) / 9),
            'protein_calories': int(tdee * ratios['protein']),
            'carbs_calories': int(tdee * ratios['carbs']),
            'fat_calories': int(tdee * ratios['fat'])
        }

    @staticmethod
    def calculate_from_grams(protein_g, carbs_g, fat_g):
        """Calculate calories from macro grams"""
        return {
            'protein_calories': protein_g * 4,
            'carbs_calories': carbs_g * 4,
            'fat_calories': fat_g * 9,
            'total_calories': (protein_g * 4) + (carbs_g * 4) + (fat_g * 9)
        }

# Usage
macros = MacroCalculator.calculate_macros(2000, 'muscle_gain')
print('Daily Macros:', macros)
```

---

## Calorie Deficit/Surplus

### Weight Change Calculation

```
1 pound of body fat ≈ 3,500 calories
1 kilogram of body fat ≈ 7,700 calories
```

### Implementation

#### JavaScript

```javascript
class WeightChangeCalculator {
  static CALORIES_PER_POUND_FAT = 3500;
  static CALORIES_PER_KG_FAT = 7700;

  static calculateDailyDeficit(tdee, target_daily_calories) {
    return tdee - target_daily_calories;
  }

  static calculateWeeklyDeficit(daily_deficit) {
    return daily_deficit * 7;
  }

  static estimateWeightLoss(daily_deficit, weeks) {
    const total_deficit = daily_deficit * 7 * weeks;
    return {
      pounds: (total_deficit / this.CALORIES_PER_POUND_FAT).toFixed(1),
      kilograms: (total_deficit / this.CALORIES_PER_KG_FAT).toFixed(1)
    };
  }

  static calculateTargetCalories(tdee, goal_type) {
    const deficits = {
      'aggressive_loss': 0.25,    // 25% deficit
      'moderate_loss': 0.15,      // 15% deficit
      'mild_loss': 0.10,          // 10% deficit
      'maintenance': 0.00,        // 0% deficit
      'mild_gain': -0.10,         // 10% surplus
      'moderate_gain': -0.15      // 15% surplus
    };

    const deficit = deficits[goal_type] || 0;
    return Math.round(tdee * (1 - deficit));
  }
}

// Usage
const tdee = 2500;
const targetCalories = WeightChangeCalculator.calculateTargetCalories(tdee, 'moderate_loss');
const dailyDeficit = WeightChangeCalculator.calculateDailyDeficit(tdee, targetCalories);
const weightLoss = WeightChangeCalculator.estimateWeightLoss(dailyDeficit, 12);

console.log(`Target Calories: ${targetCalories}`);
console.log(`Daily Deficit: ${dailyDeficit}`);
console.log(`Estimated 12-week weight loss: ${weightLoss.pounds} lbs (${weightLoss.kilograms} kg)`);
```

#### Python

```python
class WeightChangeCalculator:
    CALORIES_PER_POUND_FAT = 3500
    CALORIES_PER_KG_FAT = 7700

    @staticmethod
    def calculate_daily_deficit(tdee, target_daily_calories):
        """Calculate daily calorie deficit"""
        return tdee - target_daily_calories

    @staticmethod
    def calculate_weekly_deficit(daily_deficit):
        """Calculate weekly calorie deficit"""
        return daily_deficit * 7

    @staticmethod
    def estimate_weight_loss(daily_deficit, weeks):
        """Estimate weight loss over time"""
        total_deficit = daily_deficit * 7 * weeks
        return {
            'pounds': round(total_deficit / WeightChangeCalculator.CALORIES_PER_POUND_FAT, 1),
            'kilograms': round(total_deficit / WeightChangeCalculator.CALORIES_PER_KG_FAT, 1)
        }

    @staticmethod
    def calculate_target_calories(tdee, goal_type):
        """Calculate target daily calories based on goal"""
        deficits = {
            'aggressive_loss': 0.25,
            'moderate_loss': 0.15,
            'mild_loss': 0.10,
            'maintenance': 0.00,
            'mild_gain': -0.10,
            'moderate_gain': -0.15
        }
        
        deficit = deficits.get(goal_type, 0)
        return int(tdee * (1 - deficit))

# Usage
tdee = 2500
target_calories = WeightChangeCalculator.calculate_target_calories(tdee, 'moderate_loss')
daily_deficit = WeightChangeCalculator.calculate_daily_deficit(tdee, target_calories)
weight_loss = WeightChangeCalculator.estimate_weight_loss(daily_deficit, 12)

print(f'Target Calories: {target_calories}')
print(f'Daily Deficit: {daily_deficit}')
print(f'Estimated 12-week weight loss: {weight_loss["pounds"]} lbs ({weight_loss["kilograms"]} kg)')
```

---

## Complete Example: Full Calculation

### JavaScript

```javascript
class FitnessCalculator {
  constructor(gender, weight_kg, height_cm, age_years, activity_level) {
    this.gender = gender;
    this.weight_kg = weight_kg;
    this.height_cm = height_cm;
    this.age_years = age_years;
    this.activity_level = activity_level;
  }

  getFullCalculation(goal) {
    const bmr = BMRCalculator.calculateHarrisBenedict(
      this.gender, this.weight_kg, this.height_cm, this.age_years
    );

    const tdee = TDEECalculator.calculateTDEE(bmr, this.activity_level);
    const targetCalories = WeightChangeCalculator.calculateTargetCalories(tdee, goal);
    const macros = MacroCalculator.calculateMacros(targetCalories, goal);
    const dailyDeficit = WeightChangeCalculator.calculateDailyDeficit(tdee, targetCalories);

    return {
      bmr: Math.round(bmr),
      tdee: Math.round(tdee),
      target_calories: targetCalories,
      daily_deficit: Math.round(dailyDeficit),
      macros: macros,
      weekly_deficit: Math.round(dailyDeficit * 7),
      estimated_weight_loss_12_weeks: WeightChangeCalculator.estimateWeightLoss(dailyDeficit, 12)
    };
  }
}

// Usage
const calc = new FitnessCalculator('male', 80, 180, 30, 'moderately_active');
const plan = calc.getFullCalculation('muscle_gain');
console.log(JSON.stringify(plan, null, 2));
```

### Python

```python
class FitnessCalculator:
    def __init__(self, gender, weight_kg, height_cm, age_years, activity_level):
        self.gender = gender
        self.weight_kg = weight_kg
        self.height_cm = height_cm
        self.age_years = age_years
        self.activity_level = activity_level

    def get_full_calculation(self, goal):
        bmr = BMRCalculator.calculate_harris_benedict(
            self.gender, self.weight_kg, self.height_cm, self.age_years
        )
        
        tdee = TDEECalculator.calculate_tdee(bmr, self.activity_level)
        target_calories = WeightChangeCalculator.calculate_target_calories(tdee, goal)
        macros = MacroCalculator.calculate_macros(target_calories, goal)
        daily_deficit = WeightChangeCalculator.calculate_daily_deficit(tdee, target_calories)

        return {
            'bmr': int(bmr),
            'tdee': int(tdee),
            'target_calories': target_calories,
            'daily_deficit': int(daily_deficit),
            'macros': macros,
            'weekly_deficit': int(daily_deficit * 7),
            'estimated_weight_loss_12_weeks': WeightChangeCalculator.estimate_weight_loss(daily_deficit, 12)
        }

# Usage
calc = FitnessCalculator('male', 80, 180, 30, 'moderately_active')
plan = calc.get_full_calculation('muscle_gain')
import json
print(json.dumps(plan, indent=2))
```

---

**Last Updated:** January 2026  
**Maintained By:** Nutrition Library Team
