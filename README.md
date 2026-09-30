# Smart Pantry Manager

A Java Android application that helps a user reduce food waste by tracking the ingredients
they have at home (their "pantry") and suggesting recipes they can cook using **strictly**
those ingredients - no recipe is suggested unless every one of its required ingredients is
already in the pantry, in sufficient quantity.

## Database choice: SQLite (SQLiteOpenHelper)

SQLite was chosen over Firebase and PostgreSQL because:
- The app's data (a personal pantry list and a fixed recipe collection) is inherently
  single-user and local; there is no requirement for real-time sync across devices.
- SQLite works fully offline, with no backend to host, secure or pay for - appropriate for
  a scoped academic assignment with no server infrastructure.
- `SQLiteOpenHelper` gives direct control over the schema (three related tables: pantry
  items, recipes, recipe ingredients) and lets the strict-matching logic run as plain,
  auditable Java rather than through an ORM or a REST layer.
- Data genuinely persists between app launches (verified in the video demonstration) because
  it is written to the device's own database file, not held in memory.

## Features

- **Pantry List** - add, edit, and delete ingredients (name, quantity, unit, optional expiry
  date), shown in a RecyclerView bound to the SQLite database.
- **Suggested Recipes** - runs the strict-matching rule against the current pantry and lists
  only recipes the user can make right now, plus a separate "Almost There" section for
  recipes missing exactly one ingredient (bonus feature).
- **Recipe Detail** - full ingredient list and method for a selected recipe.
- **Settings** - toggle expiring-soon alerts and a preferred unit system, stored with
  SharedPreferences.
- 18 recipes are pre-loaded into the database the first time the app runs.

### The strict-matching rule

Implemented in `logic/RecipeMatcher.java` and `logic/IngredientMatcher.java`. A recipe
qualifies only if every required ingredient is found in the pantry under a normalised name
(handling case, punctuation and common plural forms - e.g. "Tomatoes" matches "tomato") **and**
in at least the required quantity, once both quantities are converted to a common base unit
(grams for weight, millilitres for volume). If a unit is missing or not recognised, the check
falls back to presence-only for that ingredient rather than silently rejecting the recipe.

## Project structure

```
app/src/main/java/com/jacob/smartpantry/
  model/      PantryItem, Recipe, RecipeIngredient, MatchResult
  db/         DatabaseHelper (SQLite CRUD), RecipeSeedData (18 pre-loaded recipes)
  logic/      IngredientMatcher, RecipeMatcher (the strict-matching rule)
  adapter/    PantryAdapter, RecipeAdapter (RecyclerView adapters)
  activity/   PantryListActivity (launcher), AddEditIngredientActivity,
              SuggestedRecipesActivity, RecipeDetailActivity, SettingsActivity,
              NavigationHelper (shared bottom-navigation wiring)
app/src/main/res/     layouts, strings, colours, theme, menu, vector icons
```

## Setup and run instructions (Android Studio, Windows)

1. Install [Android Studio](https://developer.android.com/studio) (includes the Android SDK).
2. `File > Open` and select the `SmartPantryManager` project folder (this repository).
3. Let Gradle sync (Android Studio downloads the Gradle wrapper jar automatically on first
   sync - an internet connection is needed for this one-time step).
4. Create or start an emulator (`Tools > Device Manager`), or connect a physical Android
   device with USB debugging enabled.
5. Click **Run** (green triangle). The app installs and launches on the Pantry List screen.
6. The database and its 18 recipes are created automatically on first launch.

No API keys, no location permissions and no internet access are required by the app itself
(and none are used - the assignment brief explicitly excludes maps/location features).

## Minimum SDK

`minSdk 21` (Android 5.0+), `targetSdk 34`, built with Gradle 8.4 / AGP 8.1.4, Java 8
language level. Built entirely in Java, no Kotlin.
